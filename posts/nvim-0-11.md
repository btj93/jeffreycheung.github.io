---
title: Neovim 0.11 Released! 🎉 Why should you upgrade from an old version?
date: 2025-03-30
permalink: /nvim-0-11
---

# What's new in Neovim 0.11

Neovim 0.11 is packed with exciting new features and bug fixes. For the complete list of changes, see:
<https://neovim.io/doc/user/news-0.11.html>

Take a look at the milestone to admire all the contributions made by the community:
<https://github.com/neovim/neovim/milestone/41>

**Here are some of the highlights:**

## Treesitter

Many treesitter performance improvements have been merged and released in 0.11. Providing a huge boost to user experience when working with large files.

- [**Async parsing**](https://github.com/neovim/neovim/pull/31631): Treesitter now supports asynchronous parsing, which no longer blocks the UI when parsing large files.
- [**Non-blocking injection query**](https://github.com/neovim/neovim/pull/32000): Treesitter now supports non-blocking injection queries, which can be used to improve performance when working / editing with large files.

- [Asnyc folding](https://github.com/neovim/neovim/pull/31827): Treesitter now supports asynchronous folding, which no longer blocks the UI when folding a large chunk of code.

## gx fixed

The default [`gx`](https://neovim.io/doc/user/various.html#gx) keymap in neovim opens a filepath or URL under the cursor.

Previously, if you try to `gx` an URL on markdown files, you should find out that it doesn't work if your cursor is on the link text.

```
[...](https://...)
  ^
```

> *Spoiler alert: it doesn't work* 🫠

With this fixed, you can now `gx` on anywhere in the link text!

Checkout the PR for more details:\
<https://github.com/neovim/neovim/pull/28630>

### Bonus tips

Check this plugin out for enhanced `gx` functionality, including:

- `gx` with your cursor on top of a plugin name to open the plugin github page in the browser
- the word/selection is automatically searched on the web, if there is no url found under the cursor

<div class="github-card" data-user="chrishrb" data-repo="gx.nvim"></div>

## Builtin auto-completion

Another long-awaited feature from Neovim 0.11 is the builtin auto-completion.

You can now try the following autocmd snippet to see the auto-completion in action:

``` lua
vim.api.nvim_create_autocmd('LspAttach', {
  callback = function(event)
    local client = vim.lsp.get_client_by_id(event.data.client_id)
    if client:supports_method('textDocument/completion') then
      vim.lsp.completion.enable(true, client.id, event.buf, { autotrigger = true })
    end
  end,
})
```

With [Neovim approaching 1.0](https://github.com/neovim/neovim/issues/20451), this is definitely a step in the right direction.

I'm excited to see more new vimmers onboarding with all these new features that makes Neovim a more powerful and enjoyable editor!

### I'm already using nvim-cmp or blink.cmp, should I switch?

<div class="github-card" data-user="hrsh7th" data-repo="nvim-cmp"></div>
<div class="github-card" data-user="Saghen" data-repo="blink.cmp"></div>

With blink.cmp 1.0 recently released, quite a few people have already switched from nvim-cmp to it.

The new built-in auto-completion is still missing some customization options and supports for snippets, so if you are currently using nvim-cmp or blink.cmp with heavy configurations, maybe you should wait it out.

I'm personally using blink.cmp, maybe one day the builtin auto completion will be able to replace blink.cmp.

I know how time-consuming it is to tinker with the config *(I have spent hours and hours on it...)*. So if your life is happy with whatever completion plugin you are current using, feel free to not worry about it.

![If it works, it works](https://btj93.github.io/nvim-0-11/if_it_works_it_works.png)
> [Source](https://www.reddit.com/r/ProgrammerHumor/comments/w6ysl9/if_it_works_it_works/)

## Diagnostic virtual lines

If you are using the builtin diagnostic virtual text, note that it is now disabled by default. It must now be opt-ed in manually with

``` lua
vim.diagnostic.config({ virtual_text = true })
```

If you are looking at a more fancy diagnostic, you can try out the new diagnostic vistual lines feature.

This is inspired by a plugin made by whynothugo called [lsp_lines.nvim](https://sr.ht/~whynothugo/lsp_lines.nvim/). Which in turns *might* be inspired by the helix editor.

It offers more features such as pin-pointing the position of error, and showing multiple errors on the same line.

<https://www.reddit.com/r/neovim/comments/1jo9oe9/i_set_up_my_config_to_use_virtual_lines_for/>

### Double diagnostic text

If you are experiencing double diagnostic message showing up, try disabling `nvim-lspconfig` virtual text.

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

### Virtual lines for errors and virtual text for warnings

``` lua
vim.diagnostic.config({
  virtual_text = {
    severity = {
      max = vim.diagnostic.severity.WARN,
    },
  },
  virtual_lines = {
    severity = {
      min = vim.diagnostic.severity.ERROR,
    },
  },
})
```

> [Source](https://www.reddit.com/r/neovim/comments/1jo9oe9/i_set_up_my_config_to_use_virtual_lines_for)

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

### Show virtual lines only on the cursor line

The following snippets will show virtual lines only on the cursor line. Otherwise, it will show the diagnostics as vistual text.

``` lua
vim.diagnostic.config({
  virtual_text = true,
  virtual_lines = { current_line = true },
  underline = true,
  update_in_insert = false
})
```

Pair it with the autocmd:

``` lua
local og_virt_text
local og_virt_line
vim.api.nvim_create_autocmd({ 'CursorMoved', 'DiagnosticChanged' }, {
  group = vim.api.nvim_create_augroup('diagnostic_only_virtlines', {}),
  callback = function()
    if og_virt_line == nil then
      og_virt_line = vim.diagnostic.config().virtual_lines
    end

    -- ignore if virtual_lines.current_line is disabled
    if not (og_virt_line and og_virt_line.current_line) then
      if og_virt_text then
        vim.diagnostic.config({ virtual_text = og_virt_text })
        og_virt_text = nil
      end
      return
    end

    if og_virt_text == nil then
      og_virt_text = vim.diagnostic.config().virtual_text
    end

    local lnum = vim.api.nvim_win_get_cursor(0)[1] - 1

    if vim.tbl_isempty(vim.diagnostic.get(0, { lnum = lnum })) then
      vim.diagnostic.config({ virtual_text = og_virt_text })
    else
      vim.diagnostic.config({ virtual_text = false })
    end
  end
})
```

> [Source](https://www.reddit.com/r/neovim/comments/1jpbc7s/disable_virtual_text_if_there_is_diagnostic_in/)

## Default keymaps

Mappings inspired by Tim Pope's vim-unimpaired:

- `[q`, `]q`, `[Q`, `]Q`, `[CTRL-Q`, `]CTRL-Q` navigate through the quickfix list
- `[l`, `]l`, `[L`, `]L`, `[CTRL-L`, `]CTRL-L` navigate through the location-list
- `[t`, `]t`, `[T`, `]T`, `[CTRL-T`, `]CTRL-T` navigate through the tag-matchlist
- `[a`, `]a`, `[A`, `]A` navigate through the argument-list
- `[b`, `]b`, `[B`, `]B` navigate through the buffer-list
- `[<Space>`, `]<Space>` add an empty line above and below the cursor
- `[[` and `]]` in Normal mode jump between shell prompts for shells which emit OSC 133 sequences ("shell integration" or "semantic prompts").

# Should you upgrade?

**I would recommend upgrading to 0.11 for the treesitter improvements alone.**
