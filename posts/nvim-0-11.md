---
title: Neovim 0.11 Released! 🎉 What should you care upgrading from an old version?
date: 2025-04-02
permalink: /nvim-0-11
---

# What's new in Neovim 0.11

## Treesitter

## gx fixed

## Built-in completion

## Diagnostic virtual lines

includes some autocmd

## Default keymaps

Mappings inspired by Tim Pope's vim-unimpaired:

- `[q`, `]q`, `[Q`, `]Q`, `[CTRL-Q`, `]CTRL-Q` navigate through the quickfix list
- `[l`, `]l`, `[L`, `]L`, `[CTRL-L`, `]CTRL-L` navigate through the location-list
- `[t`, `]t`, `[T`, `]T`, `[CTRL-T`, `]CTRL-T` navigate through the tag-matchlist
- `[a`, `]a`, `[A`, `]A` navigate through the argument-list
- `[b`, `]b`, `[B`, `]B` navigate through the buffer-list
- `[<Space>`, `]<Space>` add an empty line above and below the cursor
- `[[` and `]]` in Normal mode jump between shell prompts for shells which emit OSC 133 sequences ("shell integration" or "semantic prompts").
