+++
title = "在 org-mode 中内嵌源代码"
author = ["Evilee"]
date = 2019-12-19
lastmod = 2020-01-10T18:48:39+08:00
tags = ["Emacs", "org-mode"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

我们知道在 org-mode 中，单独的源代码块环境是使用 `#+BEGIN_SRC` 和 `#+END_SRC`.
但是在很多情况下，我们可能在一句话中内嵌一句代码，这时候用 `#+BEGIN_SRC` 就无法做到了。
<!--more-->

org-mode 的内嵌代码块格式是： `src_LANG[headers]{your code}`, 例如：
`src_sh[:exports code]{echo -e "test"}` 的效果是这样的：`echo -e "test"`.
`src_xml[:exports code]{<tag>text</tag>}` 的效果是这样的：`<tag>text</tag>`.

虽然在博客里面看不出效果，如果导出成 HTML 并且支持语法高亮的话，就会看出来内嵌代码语句的语法高亮效果了。
