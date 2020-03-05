+++
title = "Swift 5.2 中新增的几个 SwiftPM 提案"
author = ["Evilee"]
date = 2020-03-05
lastmod = 2020-03-05T12:49:53+08:00
draft = false
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

<!--more-->

Swift 5.2 中新增的几个 SwiftPM 提案：

1.  <https://github.com/apple/swift-evolution/blob/master/proposals/0226-package-manager-target-based-dep-resolution.md>
    目标依赖方案
2.  <https://github.com/apple/swift-evolution/blob/master/proposals/0271-package-manager-resources.md>
    资源管理方案
3.  <https://github.com/apple/swift-evolution/blob/master/proposals/0272-swiftpm-binary-dependencies.md>
    闭源二进制目标方案
4.  <https://github.com/apple/swift-evolution/blob/master/proposals/0273-swiftpm-conditional-target-dependencies.md>
    目标条件依赖方案

再加上 Xcode11 中已经实现的 Swift Package 依赖功能，iOS 开发已经可以抛弃
CocoaPods 和 Carthage, 使用纯 SwiftPM 方案进行工程管理了。
