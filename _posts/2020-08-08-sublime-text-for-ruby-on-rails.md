---
layout: post
title:  "Sublime Text for Ruby on Rails"
date:   2020-08-08 00:00:00 UTC
---

![Sublime Text 3 on macOS with Sidekiq source file open]({{ "/assets/sublime-text-sidekiq.png" | absolute_url }})

I've been using Sublime Text for over 5 years now primarily for Ruby on Rails development. I want to go over the configuration and plugins I've used that have served me well during all those years.

This isn't a beginner's guide though. If you're unsure how to install plugins or how to edit configuration, look up a guide on the internet and then go back to this post.

## Plugins

* [AdvancedNewFile](https://github.com/skuroda/Sublime-AdvancedNewFile)

    Allows creating new files relative to the current file easily. Instead of choosing the name and location of the file in a native save dialog, you can instead press Alt+Cmd+N, type in `:foo/bar.rb` and it'll create a folder named `foo` and a file `bar.rb` relative to current file. Without the `:` it'll create the file relative to the project root.
* [ApplySyntax](https://github.com/facelessuser/ApplySyntax)

  In Ruby, you'll have files that need different syntax highlighting schemes even though they're all have the same `.rb` extension. For example, a file might be a regular Ruby source file, the other might be an RSpec file. This plugin has a list of rules to apply the right syntax.
* [Git](https://github.com/kemayo/sublime-text-git)

  Git integration. The only function I use it for is to show `git diff`. Since I don't need it very often, I don't have a shortcut bound for it, I use the command palette.
* [GitGutter](https://packagecontrol.io/packages/GitGutter)

  Shows which lines were added or removed next to line numbers. It's very convenient because it allows you to concentrate on the changed areas.
* [All Autocomplete](https://github.com/alienhard/SublimeAllAutocomplete)

  Suggests words from other open files.
* [ChangeQuotes](https://github.com/colinta/SublimeChangeQuotes)

  Toggles single/double quotes. Depending on your code style, you might need to switch between single and double quotes quite often. This plugin doesn't ship with default keybindings, I've bound the primary function to Cmd+'
* [MarkdownPreview](https://github.com/facelessuser/MarkdownPreview)

  Renders a markdown file into HTML and shows it in the browser. Something I didn't realize straight away is that you don't need to run this command over and over after a change, the plugin will rerender the file for you, it's enough to refresh the page in the browser.
* [SublimeLinter](https://github.com/SublimeLinter/SublimeLinter), [SublimeLinter-rubocop](https://github.com/SublimeLinter/SublimeLinter-rubocop), [SublimeLinter-shellcheck](https://github.com/SublimeLinter/SublimeLinter-shellcheck)

  A linter framework with linter plugins for RuboCop and Shellcheck. They provide feedback on your code while you're typing.
* [TestRSpec](https://github.com/astrauka/TestRSpec)

  Allows you to switch between code and spec, create the spec if it doesn't exist, and run specs in Sublime.
* [Theme - SoDaReloaded](https://github.com/michaelworm/SoDaReloaded-Theme) (Dark)

  The theme I've used for a while, try it out!

## Settings

```json
{
  "font_face": "Source Code Pro",
  "font_size": 10
}
```

I've been using [Source Code Pro](https://github.com/adobe-fonts/source-code-pro) as my primary coding font.

```json
{
  "tab_size": 2,
  "translate_tabs_to_spaces": true,
  "trim_trailing_white_space_on_save": true,
  "ensure_newline_at_eof_on_save": true
}
```

Use 2 spaces for indentation, remove trailing newline on save, add a newline at the end of file if it's missing.

```json
{
  "word_separators": "./\\()\"'-:,.;<>~@#$%^&*|+=[]{}`~"
}
```

It's the default list with `!` and `?` removed, since they can be a part of the method name in Ruby.


```json
{
  "file_exclude_patterns": [
    ".DS_Store"
  ],
  "folder_exclude_patterns": [
    "log",
    ".git",
    "node_modules",
    "tmp",
    "__pycache__",
    ".vscode",
    ".idea"
  ],
  "index_exclude_patterns": [
    "*/log/*",
    "*/node_modules/*",
    "*/tmp/*"
  ]
}
```

Rules to exclude irrelevant files and folders.

```json
{
  "rulers": [99]
}
```

Adds a vertical line at 99 columns to let you know if you're under line length limit. Adjust to match your project's conventions.
