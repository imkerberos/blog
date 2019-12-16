+++
title = "如何优雅地删除 Git submodule"
author = ["Eviler"]
date = 2019-12-16
lastmod = 2019-12-16T18:10:33+08:00
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

1.  在 `.gitmodules` 文件中删除关于 xxxx 的 section.
2.  保存 `.gitmodules` 并使用 `git add .gitmoudles` 保存修改。
3.  在 `.git/config` 文件中删除关于 xxxx 模块的配置章节。
4.  运行 `git rm --cached path_to_xxxx_submodule` (没有后面的 "/").
5.  运行 `rm -rf .git/modules/path_to_xxxx_submodule` (没有后面的 "/").
6.  提交修改 `git ci -m "remove xxxx submmodule "` .
7.  删除不用的目录 `rm -rf path_to_xxxx_submodule` .