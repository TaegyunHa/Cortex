---
related:
  - "[[Programming]]"
created: 2026-02-22
tags:
---
# nvim configuration

## Install nvim

Windows
```bash
scoop bucket add main
scoop install neovim
```
MacOS
```bash
brew install neovim
```

## External Dependencies

- Basic utils: `git`, `make`, `unzip`, C Compiler (`gcc`)
- [ripgrep](https://github.com/BurntSushi/ripgrep#installation),   [fd-find](https://github.com/sharkdp/fd#installation)
- Clipboard tool (xclip/xsel/win32yank or other depending on the platform)
- A [Nerd Font](https://www.nerdfonts.com/): optional, provides various icons
-  if you have it set `vim.g.have_nerd_font` in `init.lua` to true
- Language Setup:
    - If you want to write Typescript, you need `npm`
    - If you want to write Golang, you will need `go`
    -  etc.

Windows
```bash
scoop install ripgrep
scoop install fd
scoop bucket add nerd-fonts
scoop install Hack-NF
```

MacOS
```bash
brew install ripgrep
brew install fd
brew install font-hack-nerd-font
```

## Configuration path

> Initial file is `init.lua`

| OS                   | PATH                                      |
| :------------------- | :---------------------------------------- |
| Linux, MacOS         | `$XDG_CONFIG_HOME/nvim`, `~/.config/nvim` |
| Windows (cmd)        | `%localappdata%\nvim\`                    |
| Windows (powershell) | `$env:LOCALAPPDATA\nvim\`                 |
## Learn LUA

> https://learnxinyminutes.com/docs/lua/

## LazyVim

Check the current status of the plugins
```
:Lazy
```

# LSP

Check if language server is installed
```
:Mason
```

Check if LSP is properly attached
```
:LspInfo
```