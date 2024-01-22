---
layout: default
title: Micro
nav_order: 3
parent: Guides
permalink: /guides/micro
---

# Using the Micro Editor

[Micro](https://micro-editor.github.io/) is a terminal-based text editor that is easy to use and highly customizable. In this tutorial, we will cover the basic functionalities of Micro, including opening and saving files, exiting the editor, copying, cutting, and pasting text, searching for text, finding additional occurrences, and indenting a region. I will also show some basic customizations.

This guide is a quick introduction to get you started. However, it is worthwhile to read through the [Mico Installation Page](https://github.com/zyedidia/micro#installation). In particular, if you are on a Mac, to use the `ALT` (`OPTION`) keybinding you will need to configure you Terminal. See the [Micro MacOS Terminal Page](https://github.com/zyedidia/micro#macos-terminal).


## Installation

Micro is installed in the RISC-V VMs we are using this semester. However, it is easy to install if you want to use it from your shell on your own computer. You can download
it from the official website: [https://micro-editor.github.io/](https://micro-editor.github.io/)


## Opening Files

To open a file in Micro, simply run the following command in your terminal:

```text
micro <filename>
```

Replace `<filename>` with the path to the file you want to open. If the file does
not exist, Micro will create a new file with that name.


## Saving Files

To save a file in Micro, press `Ctrl + S`. If the file has not been saved before,
Micro will prompt you to enter a filename. Otherwise, it will save the changes
to the existing file.


## Exiting Micro

To exit Micro, press `Ctrl + Q`. If there are unsaved changes in the file, Micro
will prompt you to save them before exiting.


## Copying, Cutting, and Pasting Text

To select a region of text that you want to cut or copy, press `SHIFT` and the arrow keys. This will highlight a text region. Micro uses the default key bindings for copying, cutting, and pasting text:


- To copy text, select the desired text using the arrow keys, then press `Ctrl
+ C`.

- To cut text, select the desired text using the arrow keys, then press `Ctrl
+ X`.

- To paste text, place the cursor at the desired location and press `Ctrl + V`.


## Searching for Text

To search for text in Micro, press `Ctrl + F`. This will open the search bar at
the bottom of the editor. Enter the text you want to search for and press `Enter`.
Micro will highlight the first occurrence of the text.


## Finding Additional Occurrences

To find additional occurrences of the searched text, press `Ctrl + N`. Micro will
move the cursor to the next occurrence. You can continue pressing `Ctrl + N` to
cycle through all the occurrences.

## Replacing Text

You can do simple text replacement by using the micro `replace` command. You need to enter command mode `CTRL + e` (or `ALT/OPTION + e`, see below). This will prompt you to enter the string you want to replace followed by a space and the new string. If the string your want to replace or the new string has spaces in them, you can use double quotes `"` around the string. Press `RETURN` then micro will ask you to confirm each replacement.


## Indenting a Region

To indent a region of text in Micro, select the desired text using `SHIFT` and the arrow keys. Then, press `TAB` to indent the selected region. To unindent the region, press
`Shift + TAB`.

## Create a New Tab to Edit Multiple Files

Micro allows you to have multiple files open at the same time and you can easily switch between these open files. Type `CTRL + T` to create and switch to a new tab. Once you are in the tab, you can type `CTRL + O` to open a file. You can switch between tabs using `ALT + ,` (Previous Tab) and `ALT + .` (Next Tab). Note on a Mac you use the `Option` as the `ALT` key. I like to customize the keybinding so that `CTRL + LeftArrow` means Previous Tab and `CTRL + RightArrow` means Next Tab. See below for my customizations. If you no longer want a specific tab open, in that tab, type `CTRL + Q` to quit the tab. This will not quit micro unless you are in the last open tab.

## Additional Keybindings

Micro has several other useful keybindings and they can all be customized. To see all the default keybindings go to the [Default Keybinding Configuration Page](https://github.com/zyedidia/micro/blob/master/runtime/help/keybindings.md).

## Using Spaces Instead of Tabs

One thing you should definitely customize is how the `TAB` key works. It is easiest to get consistent indentation in your source code if you use spaces instead of tabs when indenting. In micro, you can add the following to `~/.config/micro/settings.json`:

```text
{
    "tabstospaces": true
}
```

## Changing Colors

You can change the color appearance of micro by changing the default `colorscheme`. You can do this by creating or editing `~/.config/micro/settings.json```:

```text
{
    "colorscheme": "bubblegum",
}
```

To learn more go to the [Micro Colors Page](https://github.com/zyedidia/micro/blob/master/runtime/help/colors.md).

## My Customizations

I try to keep customizations minimal so that I can still be productive in a default micro environment. Here is my `~/.config/micro/settings`:

```text
{
    "autoclose": false,
    "clipboard": "internal",
    "colorscheme": "xcodelike",
    "ft:c": {
        "tabsize": 4
    },
    "softwrap": true,
    "tabstospaces": true
}
```

Explanations:
- **autoclose set to false** This prevents micro from automatically adding a closing paren ')', brace '}', or bracket ']' when you type an opening paren, brace, or bracket.
- **clipboard set to internal** This is a recommended setting to get the clipboard to work the most consistently in different environments.
- **colorscheme set to xcodelike** This is a custom colorscheme I use. If you like this colorscheme, you can download it here: [xcodelike.micro](/files/xcodelike.micro). Put this file into `~/.config/micro/colorschemes`.
- **ft:c.tabsize set to 4** This tells micro to use for 4 spaces for indentation.
- **softwarp set to true** This tells micro to allow long lines to wrap to the next line so you can see the entire line on the screen even when it doesn't it into the width of the terminal window.
- **tabstospaces set to true** Use spaces instead of tab characters for indentation.

I also modify some of the default keybindings, `~/.config/micro/bindings.json`:

```text
{
    "Alt-/": "lua:comment.comment",
    "Alt-a": "SelectAll",
    "Alt-e": "CommandMode",
    "Ctrl-a": "StartOfLine",
    "Ctrl-e": "EndOfLine",
    "Ctrl-r": "HSplit",
    "CtrlDown": "CursorPageDown",
    "CtrlLeft": "PreviousTab",
    "CtrlRight": "NextTab",
    "CtrlShiftDown": "CursorEnd",
    "CtrlShiftUp": "CursorStart",
    "CtrlUnderscore": "lua:comment.comment",
    "CtrlUp": "CursorPageUp"
}
```

Note, that these keybindings remap `CTRL + E` to EndOfLine, which is an Emacs keybinding that also work in bash and zsh as well as in any edit context in MacOS. However, teh default behavior of `CTRL + E` is to enter Micro's command mode. For this, I mapped `ALT + E` (`OPTION` on a Mac) to enter command mode.

I also map `CTRL` plus the arrow keys to scroll a page down or page up at a time, which make it quicker to move through a file. `CTRL + SHIFT + UpArrow` goes to the beginning of the file and `CTRL + SHIFT + DownArrow` goes to the end of the file. These are also useful when you are working with a large file.
