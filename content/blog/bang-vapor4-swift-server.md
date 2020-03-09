+++
title = "Swift Server 方案真是一步十坑"
author = ["Evilee"]
date = 2020-03-09
lastmod = 2020-03-09T11:31:44+08:00
draft = false
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

最近做的小方案，服务器端用的是 Vapor4. 一路上一步十坑走过来，终于遇到了没法解决的问题，或者说需要付出很大成本才能解决的问题。

<!--more-->

事情是这样的，昨天配置好了 PG 的中文检索以后，准备使用，发现了几个问题：

1.  Vapor4 的 ORM 系统 Fluent 不支持 tsvector, tsquery 数据类型。
2.  Fluent4 居然不支持添加索引，非得要我另外写 SQL 脚本在 table 上添加额外的索引。

也不知道这种 ORM 有什么 P 用，写一个查询本来用 SQL 10s 可以搞定的，非得经过 n (
n > 5) 次的封装设计以后才行，而且吧，由于不支持一些非常非常基本的功能导致无法直接用 Model 来创建数据库 Schema. 这样我还用这种 ORM 干嘛呢？ 更要命的是，Fluent
居然不支持原始 SQL 语句操作。我操，你一些功能不支持也就罢了，大不了我写 SQL 搞定，连这个路子都给封死，开发人员想啥呢？是天天用 Swift 的最新某些特性改写已有的代码写迷糊了吧？

实在不行我换 DotNet Core, 换来换去还是 MS 靠谱。
