---
title: Neovim 0.11 Released! 🎉 What should you care upgrading from an old version?
date: 2025-04-02
permalink: /nvim-0-11
---

# What's new in Neovim 0.11

Neovim 0.11 is packed with new features and bug fixes. For the complete list of changes, see:
<https://neovim.io/doc/user/news-0.11.html>

Take a look at the milestone to admire all the contributions made by the community:
<https://github.com/neovim/neovim/milestone/41>

*Here are some of the highlights:*

## Treesitter

Many treesitter performance improvements have been merged and released in 0.11.

### Async parsing

<https://github.com/neovim/neovim/pull/31631>

### Non-blocking injection query

<https://github.com/neovim/neovim/pull/32000>

## gx fixed

[`gx`](https://neovim.io/doc/user/various.html#gx) in neovim opens a filepath or URL under the cursor.

Previously, if you try to `gx` an URL on markdown files, you should find out that it doesn't work if your cursor is on the link text.

```
[...](https://...)
  ^
```

> Doesn't work 🫠

With this fixed, you can now `gx` on anywhere in the link text!

Checkout the PR for more details:\
<https://github.com/neovim/neovim/pull/28630>

### Bonus tips

Check this plugin out for enhanced `gx` functionality, including:

- hover over a plugin name to open the plugin github page in the browser
- the word/selection is automatically searched on the web, if there is no url found under the cursor

<https://github.com/chrishrb/gx.nvim>

## Built-in completion

## Diagnostic virtual lines

includes some autocmd

<https://www.reddit.com/r/neovim/comments/1jpbc7s/disable_virtual_text_if_there_is_diagnostic_in/>

<https://www.reddit.com/r/neovim/comments/1jo9oe9/i_set_up_my_config_to_use_virtual_lines_for/>

### Double diagnostic text

Disable `nvim-lspconfig` virtual text

``` lua
return {
  "neovim/nvim-lspconfig",
  opts = {
    diagnostics = {
      virtual_text = false,
    },
  },
}
```

### Toggle diagnostic virtual lines and virtual text

``` lua
vim.keymap.set('n', '<leader>tdd', function()
    vim.diagnostic.config {
        virtual_lines = not vim.diagnostic.config().virtual_lines,
        virtual_text = not vim.diagnostic.config().virtual_text,
     }
end, { desc = 'Toggle diagnostic virtual lines and virtual text' })
```

> [Source](https://www.reddit.com/r/neovim/comments/1jo9oe9/comment/mkti11p/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

## Default keymaps

Mappings inspired by Tim Pope's vim-unimpaired:

- `[q`, `]q`, `[Q`, `]Q`, `[CTRL-Q`, `]CTRL-Q` navigate through the quickfix list
- `[l`, `]l`, `[L`, `]L`, `[CTRL-L`, `]CTRL-L` navigate through the location-list
- `[t`, `]t`, `[T`, `]T`, `[CTRL-T`, `]CTRL-T` navigate through the tag-matchlist
- `[a`, `]a`, `[A`, `]A` navigate through the argument-list
- `[b`, `]b`, `[B`, `]B` navigate through the buffer-list
- `[<Space>`, `]<Space>` add an empty line above and below the cursor
- `[[` and `]]` in Normal mode jump between shell prompts for shells which emit OSC 133 sequences ("shell integration" or "semantic prompts").
