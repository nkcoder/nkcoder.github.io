---
title: Intellij Idea在VIM模式下，Shift+方向键无法选中
date: 2020-02-19T23:20:51+08:00
tags:
- vim
draft: false
---

正常情况下，Intellij Idea 中使用`Shift+方向键`是可以选中内容的，但是如果启用[VIM 插件](https://plugins.jetbrains.com/plugin/164-ideavim)，无论在哪种模式下，`Shift+方向键`无法选中内容，而是在单词之间移动。

查看 Intellij Idea 的 `Perference -> KeyMap`，通过快捷键搜索，发现该快捷键组合只赋给了`Selection`，并没有冲突。

这是 VIM 插件的一个 bug，ticket 在[这里](https://youtrack.jetbrains.com/issue/VIM-437)，正式版本还未修复，[EAP 版本](https://github.com/JetBrains/ideavim#get-an-early-access)已经修复了。

在正式版本下，也可以通过添加配置文件`~/.ideavimrc`启用 Shift+方向键的选中功能，内容如下：

```text
nnoremap <S-Left> :action EditorLeftWithSelection<CR>
nnoremap <S-Right> :action EditorRightWithSelection<CR>
nnoremap <S-Up> :action EditorUpWithSelection<CR>
nnoremap <S-Down> :action EditorDownWithSelection<CR>

inoremap <S-Left> <C-O>:action EditorLeftWithSelection<CR>
inoremap <S-Right> <C-O>:action EditorRightWithSelection<CR>
inoremap <S-Up> <C-O>:action EditorUpWithSelection<CR>
inoremap <S-Down> <C-O>:action EditorDownWithSelection<CR>
```

**参考:**

- [Using PhpStorm IdeaVim, I can't use shift+arrow keys to select words](https://stackoverflow.com/questions/21711551/using-phpstorm-ideavim-i-cant-use-shiftarrow-keys-to-select-words)
- [Possible to enable shift+arrows selection mode?](https://youtrack.jetbrains.com/issue/VIM-437)
