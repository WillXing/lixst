---
title: Tern＋YouCompleteMe实现vim中JS自动补全
date: 2017-05-28  14:00:00
tags:
  - javascript
  - vim
---
使用Vim的过程中会发现其自带的代码补全功能非常的不好用。

尝试了Tern和YouCompleteMe做不全，这里就介绍一下如何使用Tern和YouCompleteMe在Vim中实现Javascript自动补全。

## 安装Vundle

1. 使用如下命令下载 Vundle到.vim/bundle/Vundle.vim目录下
```bash
mkdir ~/.vim/bundle
mkdir ~/.vim/bundle/Vundle.vim
cd ~/.vim/bundle/Vundle.vim
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
2. 在.vimrc中配置Vundle
```bash
vim ~/.vimrc
```
在其中加入如下内容
```bash
set nocompatible
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'gmarik/Vundle.vim'
call vundle#end()
filetype plugin indent on
```

## 安装 Auto Complete 和 Tern

1. 使用Vundle安装YouCompleteMe和Tern
  1. 打开 .vimrc
``` bash
vim ~/.vimrc
```
  2. 在 `Plugin 'gmarik/Vundle.vim'` 后面追加如下内容
```bash
Plugin 'Valloric/YouCompleteMe'
Plugin 'marijnh/tern_for_vim'
```
  3. 打开vim，并运行:PluginInstall
2. 编译YouComplete
  1. 编译前需要先安装cmake
```bash
cd ~/.vim/bundle/YouCompleteMe/
./install.sh
```

## 配置 Tern

1. 在Tern目录下执行npm install
```bash
cd ~/.vim/bundle/tern_for_vim
npm install
```
2. 在你的项目根目录创建 .tern_project 文件，并配置
```bash
touch .tern_project
```
在其中加入如下内容
```json
{
  "libs": [
    "browser",
    "underscore",
    "jquery"
  ],
  "plugins": {}
}
```
在plugin中可以加入node或者angular等关键字，加载该库的补全功能。如下：
```json
"plugins": {
  "node": {}
}
```
接下来便可以进行使用了，打开项目中的文件，感受自动补全的快感～