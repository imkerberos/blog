+++
title = "初识 SwiftNIO"
author = ["Evilee"]
date = 2020-02-20
lastmod = 2021-03-03T09:52:58+08:00
tags = ["Swift", "SwiftNIO"]
categories = ["计算机"]
draft = true
creator = "Emacs 27.1 (Org mode 9.5 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

`SwiftNIO` 是 SSWG (Swift Server Working Group) 最重要的基础组件之一, `Kitura`,
`Perfect`, `Vapor` 都基于 `SwiftNIO`.

<!--more-->

当然，如果只是基于以上三个框架进行开发，或者主要关注于 HTTP 层的开发，是没有必要去了解 `SwiftNIO` 的。但如进行比较复杂的非阻塞模型开发的话，=SwiftNIO= 有着不小的帮助。

或许你接触过 RxSwift 或者 ES6, 那么对 future, promise 等不会陌生. 但是小心，这里有很多陷阱。ES6 中的 promise 和 RxSwift 中的 observable 很相似，但是 SwiftNIO 中的 promise 确是和 RxSwift 中的 observer 相似。
