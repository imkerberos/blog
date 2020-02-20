+++
title = "初识 SwiftNIO"
author = ["Evilee"]
date = 2020-02-20
lastmod = 2020-02-20T18:32:02+08:00
tags = ["Swift", "SwiftNIO"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

`SwiftNIO` 是 SSWG (Swift Server Working Group) 最重要的基础组件之一, `Kitura`,
`Perfect`, `Vapor` 都基于 `SwiftNIO`.

<!--more-->

当然，如果只是基于以上三个框架进行开发，或者主要关注于 HTTP 层的开发，是没有必要去了解 `SwiftNIO` 的。但如进行比较复杂的非阻塞模型开发的话，=SwiftNIO= 有着不小的帮助。
