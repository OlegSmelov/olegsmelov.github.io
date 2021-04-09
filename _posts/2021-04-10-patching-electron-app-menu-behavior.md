---
layout: post
title:  "Patching Electron app menu behavior"
date:   2021-04-09 20:00:00 UTC
---

For a few weeks now, I've been trying out [Kinto][0] – an application for Linux (and Windows) that remaps keyboard keys so you can use macOS shortcuts on your Linux computer. While not perfect, I've been surprised by how well it worked.

On macOS, the shortcut to delete a word is Alt-Backspace. When you press it on a Linux computer with Kinto installed, it correctly remaps that to Ctrl-Backspace. This works well in almost all applications, except for Electron apps. Electron apps activate the main menu when you release Alt.

Apparently, this is a feature that comes from the way most applications on Windows work: if you press Alt without pressing anything else, it will focus the first menu item. This is not a common behavior on Linux however – neither GTK nor Qt apps do this.

Even though this can be worked around in Kinto with varying levels of success, the real fix needs to happen in the Electron framework. With this in mind, I've set out to dig into this issue.

# Looking into Electron's source code

After going through [the pull request that introduced this][1], I realized there doesn't seem to be a way to disable this behavior. The culprit is [this piece of code][2]. Since a proper fix won't come anytime soon, let's binary patch compiled executables ✨.

# Analyzing the binary

First, we need to find where the binary is. Let's run Signal and list running processes:

```sh
$ ps aux | grep signal
[...]
user      211853  2.5  0.8 6837048 267392 ?      SLl  01:08   0:02 /opt/Signal/signal-desktop --no-sandbox
user      211865  0.0  0.1 208664 45708 ?        S    01:08   0:00 /opt/Signal/signal-desktop --type=zygote --no-zygote-sandbox --no-sandbox
user      211866  0.0  0.1 208664 45932 ?        S    01:08   0:00 /opt/Signal/signal-desktop --type=zygote --no-sandbox
[...]
```

There it is! Let's open `/opt/Signal/signal-desktop` in IDA Pro. There's a [free version][3] that will do just fine for our needs. Signal binary is huge (~130 MB), so the analysis takes a looong time.

After the analysis is finished, our main goal is to find where that piece of code is in the binary. Sadly, the executable doesn't have any debugging symbols, so we can't just search for the function by its name. Without compiling Electron ourselves (which I guess takes like half a day), we need to find something unique about this piece of code. String literals work best, however, there seem to be no string literals around.

I then found the following function:

```cpp
const int kMenuBarHeight = 25;
// ...
int RootView::GetMenuBarHeight() const {
  return kMenuBarHeight;
}
```

While not perfect, we can search for this value in IDA Pro by pressing Alt-I. Enter 25, press Enter. 32265 results. Oof. Let's try to narrow them down.

If we reference [the standard calling conventions][4] we should expect the function to store the constant into the `rax`/`eax`/`ax`/`al` register. So let's press Ctrl-F and enter

```assembly
mov     eax, 19h
```

57 results. That's better. Let's go through all of them and find a really short function. To my surprise, the function I needed was the first in the list. Let's rename it to `GetMenuBarHeight`:

```assembly
.text:0000000001A5FAA0 GetMenuBarHeight proc near
.text:0000000001A5FAA0     push    rbp
.text:0000000001A5FAA1     mov     rbp, rsp
.text:0000000001A5FAA4     mov     eax, 19h
.text:0000000001A5FAA9     pop     rbp
.text:0000000001A5FAAA     retn
.text:0000000001A5FAAA GetMenuBarHeight endp
```

This function by itself is not very important, but if we go through adjacent functions, we'll realize the compiled binary follows the source code pretty closely. I figured out the next functions were: `SetAutoHideMenuBar`, `IsMenuBarAutoHide`, `IsMenuBarVisible` and then `HandleKeyEvent`, the one we need to patch!

# Patch idea

If we pretend that the Alt key press never happened, the menu will not be activated. We need to make a patch as if [this line][5] was

```cpp
    menu_bar_alt_pressed_ = false;
```

# Analyzing HandleKeyEvent function

If we look at the source code again, the function has an early return which ensures that `menu_bar_` is set:

```cpp
void RootView::HandleKeyEvent(const content::NativeWebKeyboardEvent& event) {
  if (!menu_bar_)
    return;
  // ...
}
```

Before we go further, I need to mention that methods in C++ have an implicit first parameter which is a pointer to the object (`this`). To the method above actually has two (`RootView *this, NativeWebKeyboardEvent &event`). This is what the method looks like in the disassembly:

```assembly
HandleKeyEvent proc near
push    rbp                 ; init stack frame, save register values, etc.
mov     rbp, rsp
push    r14
push    rbx
mov     r14, rdi            ; rdi stores the first argument (`this` keyword)
                            ; it is then copied to r14

mov     rdi, [rdi+288h]     ; rdi = this->menu_bar_;
test    rdi, rdi            ; if (!rdi)
jz      loc_1A5FBE7         ;     return;
```

This gives us some important information:

* `r14` stores the pointer to the `RootView` object.
* `288h` is the offset in the object for `menu_bar_` variable.

Now, if we now look at [the header file][6], we can find the offset for `menu_bar_alt_pressed_`:

```cpp
class RootView : public views::View {
 public:
  // ...
  // Menu bar.
  std::unique_ptr<MenuBar> menu_bar_;   // r14+288h
  bool menu_bar_autohide_ = false;      // r14+288h+8   (8 is sizeof(std::unique_ptr<MenuBar>)
  bool menu_bar_visible_ = false;       // r14+288h+8+1 (1 is sizeof(bool))
  bool menu_bar_alt_pressed_ = false;   // r14+288h+8+1+1 = r14+292h
```

Now we need to look for `[r14+292h]` in the disassembly. There are 4 matches:

```assembly
.text:0000000001A5FB39    mov   byte ptr [r14+292h], 1   ; set to true
.text:0000000001A5FB50    cmp   byte ptr [r14+292h], 0   ; compare to 0
.text:0000000001A5FB5E    mov   byte ptr [r14+292h], 0   ; set to false
.text:0000000001A5FBDF    mov   byte ptr [r14+292h], 0   ; set to false
```

The first one is exactly what we were looking for. If you now go to the `mov byte ptr [r14+292h], 1` line, IDA Pro will tell you the offset where the instruction in the input file is. In my case, it was `01A5EB39`.

# Digging into the instruction

If you open the hex view in IDA Pro while the instruction is selected, it will highlight instruction's machine code:

```
41 C6 86 92 02 00 00 01    mov   byte ptr [r14+292h], 1
```

Using [x86-64 opcode reference][7] we can figure out what each byte means:

```
41               register extension prefix (REX.B), allows access to r14
C6               instruction: mov r/m8, imm8
86               ModR/M byte which corresponds to [R14/R14D]+disp32
92 02 00 00      disp32 value: 0x292 (encoded as little endian)
01               imm8 value (0x01)
```

The last byte is the one we need to patch.

# Patching

We're very close now! Let's open this file in GHex:

```sh
$ sudo ghex /opt/Signal/signal-desktop
```

Press Ctrl-J and type the offset. At offset 0x01A5EB40, there is the value 1 that corresponds to `true`. Change it to `00` (`false`) and save.

Run Signal, press and release Alt, and there it is, the menu no longer gets activated! 🎉🎉🎉

# Next steps

The biggest issue is that this patch needs to be reapplied after each update. The other obvious issue is that each Electron app needs to be patched.

Hang on until the next blog post where I'll show you how to write a generic patcher that works with any Electron version and any Electron app – until developers make changes to that part of the code, of course.

[0]: https://github.com/rbreaves/kinto
[1]: https://github.com/electron/electron/pull/15302
[2]: https://github.com/electron/electron/blob/95e26e2fd4bb096cbcc7e7803da7dedebfa1e4cf/shell/browser/ui/views/root_view.cc#L137-L159
[3]: https://www.hex-rays.com/products/ida/support/download_freeware/
[4]: https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI
[5]: https://github.com/electron/electron/blob/95e26e2fd4bb096cbcc7e7803da7dedebfa1e4cf/shell/browser/ui/views/root_view.cc#L141
[6]: https://github.com/electron/electron/blob/95e26e2fd4bb096cbcc7e7803da7dedebfa1e4cf/shell/browser/ui/views/root_view.h
[7]: http://ref.x86asm.net/coder64.html
