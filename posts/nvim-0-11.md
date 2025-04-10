---
title: Neovim 0.11 Released! 🎉 Why upgrade from an older version?
date: 2025-03-30
permalink: /nvim-0-11
---

# What's New in Neovim 0.11

Neovim 0.11 is packed with exciting new features, performance boosts, and bug fixes.  
For the full changelog, check out:  
<https://neovim.io/doc/user/news-0.11.html>

Or admire the community's hard work in the milestone:  
<https://github.com/neovim/neovim/milestone/41>

---

## Highlights

### Treesitter: Faster, Smoother, Async

Neovim 0.11 brings major Treesitter improvements, dramatically enhancing the experience when working with large files:

- [**Async parsing**](https://github.com/neovim/neovim/pull/31631): Parsing now happens asynchronously, so your UI won't freeze on big files.
- [**Non-blocking injection queries**](https://github.com/neovim/neovim/pull/32000): Language injections (like embedded code blocks) are now parsed without blocking.
- [**Async folding**](https://github.com/neovim/neovim/pull/31827): Code folding is now async too, making it smooth even on huge files.

---

### `gx` Now Works on Markdown Link Text

The default [`gx`](https://neovim.io/doc/user/various.html#gx) keymap opens the URL or file path under your cursor.

**Before:** In markdown, `gx` only worked if your cursor was on the URL part, not the link text:

```
[...](https://...)
  ^
```

> ⚠️ *Spoiler: it didn't work on the link text* 🫠

**Now:** You can `gx` anywhere inside the link text, and it will open the URL!  
See [PR #28630](https://github.com/neovim/neovim/pull/28630) for details.

**Bonus:**  
Try this plugin for supercharged `gx`:

- Open plugin GitHub pages by `gx` on the plugin name
- Search the web if no URL is found under the cursor

<div class="gh-card gh-large" data-repo="chrishrb/gx.nvim"></div>

---

### Built-in Auto-Completion

Neovim 0.11 introduces **native LSP completion** — no plugin required!

Try this snippet to enable it:

```lua
vim.api.nvim_create_autocmd('LspAttach', {
  callback = function(event)
    local client = vim.lsp.get_client_by_id(event.data.client_id)
    if client:supports_method('textDocument/completion') then
      vim.lsp.completion.enable(true, client.id, event.buf, { autotrigger = true })
    end
  end,
})
```

With [Neovim approaching 1.0](https://github.com/neovim/neovim/issues/20451), this is a big step forward.

I'm excited to see more new vimmers onboard with these features that make Neovim more powerful and enjoyable!

#### Should you ditch `nvim-cmp` or `blink.cmp`?

<div class="github-card" data-github="hrsh7th/nvim-cmp" data-width="400"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>
<div class="gh-card gh-large" data-repo="Saghen/blink.cmp"></div>

<div class="github-card" data-github="" data-width="400"></div>
<script src="//cdn.jsdelivr.net/github-cards/latest/widget.js"></script>

With blink.cmp 1.0 just released, many have switched from nvim-cmp.

However, the built-in completion **still lacks**:

- Snippet support
- Deep customization options

If you rely heavily on those, **stick with your plugin for now**.

Personally, I use blink.cmp with lots of tweaks, so I’m sticking with it on 0.11 — hoping someday the built-in completion will be good enough to replace it.

If your current setup works well, **no need to rush**.  
Remember:

![If it works, it works](https://btj93.github.io/nvim-0-11/if_it_works_it_works.png)  
> [Source](https://www.reddit.com/r/ProgrammerHumor/comments/w6ysl9/if_it_works_it_works/)

---

### Diagnostic Virtual Lines

By default, **virtual text diagnostics are now disabled**.  
To enable them:

```lua
vim.diagnostic.config({ virtual_text = true })
```

But wait! There's more:

Neovim 0.11 introduces **diagnostic virtual lines**, inspired by [lsp_lines.nvim](https://sr.ht/~whynothugo/lsp_lines.nvim/) (and maybe Helix editor).  
They:

- Pinpoint error positions more clearly
- Show multiple errors on the same line
- Are less cluttered than inline virtual text

#### Avoid double diagnostics

If you see **duplicate** diagnostics, disable `nvim-lspconfig`'s virtual text:

```lua
return {
  "neovim/nvim-lspconfig",
  opts = {
    diagnostics = {
      virtual_text = false,
    },
  },
}
```

#### Show errors as virtual lines, warnings as virtual text

```lua
vim.diagnostic.config({
  virtual_text = {
    severity = { max = vim.diagnostic.severity.WARN },
  },
  virtual_lines = {
    severity = { min = vim.diagnostic.severity.ERROR },
  },
})
```

> [Source](https://www.reddit.com/r/neovim/comments/1jo9oe9/i_set_up_my_config_to_use_virtual_lines_for)

#### Toggle virtual lines and virtual text with a keymap

```lua
vim.keymap.set('n', '<leader>tdd', function()
  vim.diagnostic.config {
    virtual_lines = not vim.diagnostic.config().virtual_lines,
    virtual_text = not vim.diagnostic.config().virtual_text,
  }
end, { desc = 'Toggle diagnostic virtual lines and virtual text' })
```

> [Source](https://www.reddit.com/r/neovim/comments/1jo9oe9/comment/mkti11p/)

#### Show virtual lines **only** on the cursor line

Show inline text normally, but virtual lines **only** on the current line:

```lua
vim.diagnostic.config({
  virtual_text = true,
  virtual_lines = { current_line = true },
  underline = true,
  update_in_insert = false,
})
```

Pair it with this autocmd to toggle dynamically:

```lua
local og_virt_text
local og_virt_line

vim.api.nvim_create_autocmd({ 'CursorMoved', 'DiagnosticChanged' }, {
  group = vim.api.nvim_create_augroup('diagnostic_only_virtlines', {}),
  callback = function()
    if og_virt_line == nil then
      og_virt_line = vim.diagnostic.config().virtual_lines
    end

    -- Ignore if virtual_lines.current_line is disabled
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
  end,
})
```

> [Source](https://www.reddit.com/r/neovim/comments/1jpbc7s/disable_virtual_text_if_there_is_diagnostic_in/)

---

### New Default Keymaps

Inspired by Tim Pope’s vim-unimpaired, Neovim 0.11 adds many handy mappings:

- `[q`, `]q`, `[Q`, `]Q`, `[Ctrl-Q`, `]Ctrl-Q`: quickfix list navigation
- `[l`, `]l`, `[L`, `]L`, `[Ctrl-L`, `]Ctrl-L`: location list navigation
- `[t`, `]t`, `[T`, `]T`, `[Ctrl-T`, `]Ctrl-T`: tag match list navigation
- `[a`, `]a`, `[A`, `]A`: argument list navigation
- `[b`, `]b`, `[B`, `]B`: buffer list navigation
- `[<Space>`, `]<Space>`: add empty line above/below
- `[[` and `]]`: jump between shell prompts (OSC 133 semantic prompts)

---

# Should You Upgrade?

**Absolutely!**  
Even just for the Treesitter improvements, Neovim 0.11 is a fantastic upgrade.  
Enjoy a faster, smoother, and more powerful Neovim!
