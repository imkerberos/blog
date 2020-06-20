+++
title = "卡特兰数"
author = ["Evilee"]
date = 2020-06-20
lastmod = 2020-06-20T13:44:32+08:00
draft = false
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

卡特兰数
<!--more-->

-   通项公式一: \\( C\_n = \frac{1}{n + 1}C^n\_{2n} = C^n\_{2n} - C^{n - 1}\_{2n} \\)
-   通项公式二: \\( C\_n = \frac{1}{n + 1}\sum\_{i=0}^{n}(C^i\_n)^2 \\)
-   递推公式一: \\( C\_{n + 1} = \frac{2(2n + 1)}{n + 2}C\_n \\) 且 \\( C\_0 = 1 \\)
-   递推公式二: \\( C\_{n + 1} = \sum\_{i = 0}^{n} C\_i C\_{n - i} \\) 且 \\( C\_0 = 1 \\)
-   递推公式三: \\( h(n) = ((4 \cdot n - 2) / (n + 1)) \cdot h(n - 1)\\)
-   递推公式四: \\( h(n) = h(0) \cdot h(n -1) + h(1) \cdot h(n - 2) + \ldots + h(n - 1) \cdot h(0) \\)
