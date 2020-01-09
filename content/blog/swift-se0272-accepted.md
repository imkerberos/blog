+++
title = "SE-0272: Package Manager Binary Dependencies"
author = ["Evilee"]
date = 2020-01-09
lastmod = 2020-01-09T09:47:15+08:00
tags = ["Swift"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

SE-0272: Package Manager Binary Dependencies 提案被接受。
<!--more-->
经过漫长的讨论，Swift Package Manager 的二进制文件依赖的提案终于通过了，以后
SwiftPM 终于可以直接管理其他第三方的，无源代码的各种 SDK 了。 继 Cocoapods,
Carthage 之后，SwiftPM 终于成为了一个可用的 iOS 工程管理方案。
