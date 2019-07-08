---
title: ssh免密登录之目录权限配置
date: 2019-07-09 12:23:30
toc: false
comments: true
categories:
- Linux基本功
tags:
- ssh
---

背景：(ubuntu)之前ssh免密登录一直运行正常，今天修改了home目录权限之后免密登录突然失效了。折腾一番才知道，原来ssh免密登录对用户home目录和.ssh目录权限配置有要求。

1. /home/当前用户 需要700权限
2. .ssh目录及其文件，需要700权限

```bash
sudo chmod 700 /home/work  #work用户
chmod 700 -R ~/.ssh
```

参考资料
[SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
[ssh免密登录，各种权限设置都无效的解决办法！](https://blog.csdn.net/u011552404/article/details/78955395)



