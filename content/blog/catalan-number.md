+++
title = "卡特兰数"
author = ["Evilee"]
date = 2020-06-20
lastmod = 2020-06-20T13:34:04+08:00
draft = false
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

卡特兰数
<!--more-->

\\[ C\_n = \frac{1}{n + 1}C^n\_{2n} = C^n\_{2n} - C^{n - 1}\_{2n} \\]
\\[ C\_n = \frac{1}{n + 1}\sum\_{i=0}^{n}(C^i\_n)^2 \\]
\\[ C\_{n + 1} = \frac{2(2n + 1)}{n + 2}C\_n 且 C\_0 = 1 \\]
\\[ C\_{n + 1} = \sum\_{i = 0}^{n} C\_i C\_{n - i} 且 C\_0 = 1 \\]
