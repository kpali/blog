---
layout: post
title: Vim配置文件备忘
category: 工具
tags: Vim
---

```
"我的配置
"""""""""其他"""""""""""
set nocompatible "不使用vi兼容模式
set nobackup "设置不自动备份文件
set history=1000 "设置记录历史的行数
set autoread "文件在Vim之外修改过，自动重新读入
""""""""""显示""""""""""
syntax on "设置语法高亮显示
colorscheme desert "设置配色方案
set number "显示行数
set ruler "在状态栏显示当前的光标位置
set showmode "在状态栏显示当前的模式
set showcmd "在状态栏显示当前的命令
set cursorline "高亮当前行
"set cursorcolumn "高亮当前列
set showmatch "设置插入括号时短暂的跳转到匹配的对应括号
set matchtime=2 "短暂跳转到匹配括号的时间
"set nowrap "当一行文字很长时取消换行
set laststatus=2 "总是显示状态行
set hls "检索时高亮显示匹配项
""""""""""缩进""""""""""
set tabstop=4 "设置制表符宽度为4
set softtabstop=4 "设置软制表符宽度为4
set shiftwidth=4 "设置缩进的空格数为4
set autoindent "设置自动缩进
set smartindent "设置智能缩进
set cindent "设置C/C++语言的自动缩进方式
```

