---
title: Neovim Custom Keymap Essentials
date: 2025-04-05
permalink: /nvim-keymap-essentials
---

# Neovim Custom Keymap Essentials

## Keymaps in Neovim

In Neovim, keymaps are used to map keys to actions. They are a major feature in the vim world. They fit in very well with the philosophy of vim, which is to make the most common actions as simple as possible.

In vim, there are different modal modes, such as normal mode, insert mode, and visual mode. Each mode has its own set of keymaps.

For example, in normal mode, you can use the `h` key to move the cursor left, while in visual mode, you can use `h` to select a character to the left.

## QoL Keymaps Collections

### `*zz`

`gg` and `G` are two keymaps that are used to move the cursor to the top or bottom of the file, respectively.

The problem with `G` are that the cursor will be placed at the top or bottom row of the file. Since your eyes have to travel a lot to find the cursor, it can be very disturbing.

The `zz` keymap moves the cursor to the center of the screen, which in-turn moves the code at the end of the file to the center of the screen.

This is much better from an ergonomics perspective.

``` lua
vim.keymap.set({ "n" }, "G", "Gzz", { noremap = true, desc = "Go to bottom" })
```

Besides `G`, there are also `n`, `N`, `*`, `#`, `g*`, and `g#` keymaps that share the same problem. They move the line to just barely visible, which means the cursor is most likely at the bottom of the screen.

``` lua
vim.keymap.set("n", "n", "nzz", { noremap = true })
vim.keymap.set("n", "N", "Nzz", { noremap = true })
vim.keymap.set("n", "*", "*zz", { noremap = true })
vim.keymap.set("n", "#", "#zz", { noremap = true })
vim.keymap.set("n", "g*", "g*zz", { noremap = true })
vim.keymap.set("n", "g#", "g#zz", { noremap = true })
```

### Duplicate a line and comment out the first line

This is a common task when editing code. You want to make some changes to a line, but you want to keep the original line around for reference.

``` lua
vim.keymap.set("n", "yc", "yy<cmd>normal gcc<CR>p", { noremap = true, desc = "Duplicate a line and comment" })
```

In visual mode, you may use the following snippet:

``` lua
local function duplicate_and_comment()
  -- Escape the visual mode
  local esc = vim.api.nvim_replace_termcodes("<esc>", true, false, true)
  vim.api.nvim_feedkeys(esc, "x", false)

  -- Get the selected text
  local start_line, end_line = vim.fn.line("'<"), vim.fn.line("'>")

  -- Duplicate the selected lines
  vim.cmd(start_line .. "," .. end_line .. "yank")
  vim.cmd(end_line + 1 .. "put")

  -- reselect previous visual selection
  vim.api.nvim_feedkeys("gv", "n", false)

  -- comment the visual selection
  vim.api.nvim_feedkeys("gc", "v", false)
end

vim.keymap.set("v", "yc", duplicate_and_comment, { noremap = true, desc = "Duplicate selection and comment" })
```

### Select to the end of line

To yank to the end of the line, we have `Y`.

To delete to the end of the line, we have `D`.

To cut to the end of the line, we have `C`.

Why shouldn't we have something to select to the end of the line?

``` lua
vim.keymap.set({ "n" }, "<leader>v", "vg_", { noremap = true, desc = "Select to last non-blank character" })
```

### Move cursor to the start / end of the line

`_` moves the cursor to the start of the line, and `$` moves the cursor to the end of the line.

But they are quite far from the home row, so I have these keymaps to move the cursor to the start / end of the line.

``` lua
-- Move to start/end of line
vim.keymap.set({ "n", "v" }, "gh", "_", { noremap = true })
vim.keymap.set({ "n", "v" }, "gl", "$", { noremap = true })
```

### Quit insert mode

It is quite common to remap another key to `<esc>` in insert mode, such that your fingers don't have to move far away from the home row to quit insert mode.

Common candidates includes:

- `<C-c>`
- `jj`
- `jk`
- `kj`
- `kk`

I personally chose `jk` because you can press it with two fingers, making it faster than `jj`.

``` lua
-- remap jk
vim.keymap.set({ "i" }, "jk", "<Esc>", { noremap = true, desc = "jk to escape" })
vim.keymap.set({ "i" }, "JK", "<Esc>", { noremap = true, desc = "JK to escape" })
```

> **The only drawback is that now I am constantly inserting excessive `jk` at the end of the sentence all over the place outside of NeoVim.jk**

Alternatively, you may remap the following keys to `<esc>` on the system level or keyboard firmware level.

- `Capslock`

## Keymaps expanding based on current feature

### Diff side-by-side

I have been using the builtin diff feature for quite a while now, and it has been very helpful especially when comparing JSON files.

The `:windo diffthis` command will diff the current file with the file in the other window.

The `:diffoff!` command will turn off the diff feature.

So I wrote this little lua function to toggle the diff side-by-side.

``` lua
local function toggle_diff()
  if vim.wo.diff then
    vim.cmd("diffoff!")
  else
    vim.cmd("windo diffthis")
  end
end
vim.keymap.set({ "n" }, "<leader>dd", toggle_diff, { noremap = true, desc = "Diff side by side" })
```

### Search and replace the word under the cursor

From the Vim wiki: <https://bit.ly/4eLAARp>

``` lua
vim.keymap.set(
  "n",
  "<Leader>r",
  [[:%s/\<<C-r><C-w>\>//g<Left><Left>]],
  { desc = "Search and replace word under cursor" }
)
```

### Movements in insert mode

Now I know what you are thinking, you should not be moving the cursor in insert mode. There are different modes for a reason.

But there are just some scenarios where you need to move the cursor just a little bit in insert mode.

Sure, you can use the `<Ctrl-o>` key to just allow the following key to be normal mode, but it is *suboptimal* with the number of keys you need to press.

So here are some keymaps that I use to move the cursor in insert mode.

``` lua
-- One character at a time
vim.keymap.set("i", "<C-n>", "<Down>", { noremap = true })
vim.keymap.set("i", "<C-p>", "<Up>", { noremap = true })
vim.keymap.set("i", "<C-b>", "<Left>", { noremap = true })
vim.keymap.set("i", "<C-f>", "<Right>", { noremap = true })
vim.keymap.set("i", "<C-e>", "<C-o>$", { noremap = true })
```

These keymaps are actually quite common keymaps in other places. Most MacOS apps, even the terminal I use, [Kitty](https://github.com/kovidgoyal/kitty), comes with these keymaps out of the box.

I also have these keymaps to move the cursor one word at a time.

``` lua
-- Move one word at a time
vim.keymap.set("i", "<M-f>", "<C-o>w", { noremap = true })
vim.keymap.set("i", "<M-b>", "<C-o>b", { noremap = true })
```

### Put searched matches in the Quickfix list

``` lua
-- / or * to search, then g/ to put them into the quickfix list
vim.keymap.set("n", "g/", ":vimgrep /<C-R>//j %<CR>|:cw<CR>", { noremap = true, silent = true })
```

### Yank / delete / change up to the next quote

It has always annoyed me that we have `yw` to yank to the next word, `y]%` to yank to the next bracket, but we have to do `yt"` to yank to the next **double** quote. Especially that we have the `q` text objects from `mini.ai`.

So, `yiw` works, `yiq` works, `yw` works, but there is no `yq` to yank to the next quote.

So I embarked on a journey to create a keymap that will yank / delete / change up to the next quote.

Not a lot of searching and typing later, I came up with this:

``` lua
-- Operations up to next quote
vim.keymap.set({ "n" }, "dq", "v/[\"'`]<CR><Left>d<cmd>nohlsearch<CR>", { noremap = true })
vim.keymap.set({ "n" }, "yq", "v/[\"'`]<CR><Left>y<cmd>nohlsearch<CR>", { noremap = true })
vim.keymap.set({ "n" }, "cq", "v/[\"'`]<CR><Left>di<cmd>nohlsearch<CR>", { noremap = true })
```

Sure it isn't the most prettiest piece of code in the world, but it gets the job done.
