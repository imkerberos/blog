+++
title = "解决 Archlinux 的域名解析超时出错的问题"
author = ["Eviler"]
lastmod = 2019-12-13T19:30:29+08:00
tags = ["ArchLinux", "resolved", "systemd"]
categories = ["Linux"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 2004
+++

如果使用 systemd-resovled 启动域名解析服务，在一段时间后进行网络访问时经常出现
`Host name not found`, 解决的方法是在 `/etc/systemd/resolved.conf` 文件中添加:

> DNSSEC=no

这个问题困扰了我好久。:(
