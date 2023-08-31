---
title: VIM中输入法自动切换的问题
date: 2021-02-24T23:02:20+08:00
tags:
- vim
draft: false
---

问题描述：

> 在命令行中使用vim，或VSCode中安装了vim插件并启用vim模式后，随时可能输入中文和英文。当在vim的插入模式下输入中文后，立即使用ESC切换为命令模式后，因为还是中文输入法状态，所以vim是接收不到命令的，需要首先将输入法切换为英文。

期望结果：

>插入模式下可以随意输入中英文，然后使用ESC进入命令模式后，自动切换为英文输入法，命令模式直接可用，再进入插入模式时，自动切换为上一次的输入法。

### 确定英文输入法

首先确定自己使用的英文输入法（中文输入法随意），macOS下建议使用默认的`U.S.`，如下图所示：

![vim_mac-input-method.png](/images/tech-by-tag/vim_mac-input-method.png)

### VSCode Vim中如何解决

首先需要安装[im-select](https://github.com/daipeihust/im-select#installation)，参考最新的安装文档，如在macOS系统下，通过以下命令安装：

```bash
curl -Ls https://raw.githubusercontent.com/daipeihust/im-select/master/install_mac.sh | sh
```

获取当前输入法的ID，如果英文输入法为`U.S.`，则ID应该为`com.apple.keylayout.US`：

```bash
➜  ~ im-select
com.apple.keylayout.US
```

然后在VSCode的`settings.json`（通过 `Open Settings (JSON)` 命令可以直接打开）中添加如下配置：

```json
"vim.autoSwitchInputMethod.enable": true,
"vim.autoSwitchInputMethod.defaultIM": "com.apple.keylayout.US",
"vim.autoSwitchInputMethod.obtainIMCmd": "/usr/local/bin/im-select",
"vim.autoSwitchInputMethod.switchIMCmd": "/usr/local/bin/im-select {im}"
```

测试下，应该可以了。

### iTerm2 Vim中如何解决

通过[SmartIM](https://github.com/ybian/smartim)解决，先参考官方文档安装，如通过Vundle安装：

- 在`~/.vimrc`中添加配置：`Plugin 'ybian/smartim'`
- 打开vim，执行命令：`:PluginInstall`

相关的`.vimrc`参考：

```yaml
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim

call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" Keep Plugin commands between vundle#begin/end.

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" smartim to swith input method automatically
Plugin 'ybian/smartim'

" All of your Plugins must be added before the following line
call vundle#end()            " required

filetype plugin indent on    " required

" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
```

[SmartIM](https://github.com/ybian/smartim)假设默认的英文输入法为`U.S.`（即`com.apple.keylayout.US`)，如果是其它英文输入法，则在`.vimrc`中需要添加如下配置：

```yaml
let g:smartim_default = '<your_default_keyboard_id>'
```

---

> 参考：
>
> 1. [如何解决VSCode Vim中文输入法切换问题？](https://www.zhihu.com/question/303850876)
> 2. [SmartIM](https://github.com/ybian/smartim)
