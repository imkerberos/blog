+++
title = "把 Markdown 格式的文件转换成 org-mode 格式"
author = ["Eviler"]
lastmod = 2019-12-15T18:30:07+08:00
tags = ["Emacs", "org-mode", "markdown"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 2002
authorbox = true
comments = true
toc = true
mathjax = true
+++

正在把以前的 jekyll 博客迁移到新的 ox-hugo 上，虽然没有几篇，但是如果手工把
markdown 转换成 org-mode 还是有不小的工作量的， 还好有 `pandoc`, 转换完成后稍微修改一下就可以了。

```text
brew install pandoc
pandoc -f markdown -t org xxxx.md -o xxxx.org
```
