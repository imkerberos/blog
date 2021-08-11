+++
title = "从 zplug 迁移到 zinit"
author = ["Evilee"]
date = 2021-08-11
lastmod = 2021-08-11T19:41:46+08:00
draft = false
creator = "Emacs 27.2 (Org mode 9.5 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

使用 zplug 管理插件已经有些年头了，加载时间大约在 6s-8s 之间。最近换了一个 13 寸的 macbook pro,
可能是性能的原因吧，zsh 的加载时间基本固定在了 8s. 实在忍受不了了，用 zinit 替换了 zplug ，加载
时间瞬间从 8s 降到了 0.6s, 真的是飞速了， 现在想想，真不知道当初是怎么过来的。
