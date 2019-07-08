---
title: vim常用操作
date: 2019-07-08 11:23:30
toc: true
comments: true
categories: 
- Linux基本功
tags: 
- vim
---

### 显示/隐藏非可见字符（tab，行尾等）
```bash
:set list    #显示
:set nolist  #隐藏
:set list!   #切换显示/隐藏
```
### 大小写敏感配置
```bash
set ignorecase  #忽略大小写
set smartcase   #大小写敏感
```
### 查找
在normal模式下按下/即可进入查找模式：
```bash
/foo\c  #查找：忽略大小写
/foo\C  #查找：大小写敏感
```
按下n查找下一个，按下N查找上一个。
参考：(在 Vim 中优雅地查找和替换)[https://harttle.land/2016/08/08/vim-search-in-file.html]

