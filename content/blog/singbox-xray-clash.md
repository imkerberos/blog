+++
title = "对比 sing-box, xray, clash"
author = ["Evilee"]
date = 2012-04-02
lastmod = 2024-11-29T14:41:16+08:00
tags = ["ssh", "gfw"]
categories = ["计算机"]
draft = true
creator = "Emacs 29.4 (Org mode 9.7.11 + ox-hugo)"
+++

上次写穿墙已经是 10 年前了，最近这 10 年, 墙和翻墙工具都有了很大的变化，这里简单对比一下
sing-box, xray, clash 这三个工具。
&lt;!--more--&gt;


## <span class="section-num">1</span> TL;DR {#tl-dr}

我这里使用的方案是: sing-box(客户端) + xray(服务器), 后期可能迁移到 sing-box &lt;-&gt; sing-box 的方案。


## <span class="section-num">2</span> 各个工具的优缺点: {#各个工具的优缺点}

| 工具           | 优点                     | 缺点                           |
|--------------|------------------------|------------------------------|
| sing-box       | 协议支持全, 独有的 DNS 分流特别适合客户端 | 新项目, 不知道后续, 在服务器端没有太大优势 |
| xray (v2ray)   | 老牌,稳定, 协议领头羊    | 客户端没有 tun 模式, 规则处理弱 |
| clash (mihomo) | 规则处理完善             | 无法对 DNS 分流, 内置 DNS 分流策略高深(混乱?) |
