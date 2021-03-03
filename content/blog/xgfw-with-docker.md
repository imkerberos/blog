+++
title = "Blog 和 Xgfw 转移到了 docker 里面"
author = ["Evilee"]
date = 2021-03-03
lastmod = 2021-03-03T10:10:26+08:00
categories = ["计算机"]
draft = false
creator = "Emacs 27.1 (Org mode 9.5 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

把 Blog 和 Xgfw 的配置做成了 Docker, 以后不怕迁移了. 这个 docker 使用了 caddy v1 (主要是 caddy v2 不支持 github hook 了),
和 hugo 来托管我的 blog. 另外内置了 v2ray 来 x GFW.
