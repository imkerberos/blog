+++
title = "使用 Tagro 开发小程序"
author = ["Evilee"]
date = 2020-02-04
lastmod = 2020-02-15T23:20:37+08:00
tags = ["typescript", "taro", "小程序", "RxJS"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
+++

## <span class="section-num">1</span> 建立手脚架 {#建立手脚架}

1.  安装 NodeJS
2.  安装 Taro
3.  安装镜像配置
4.  初始化项目
5.  安装 TaroUI
6.  安装 RxJS, RxJS-compact


## <span class="section-num">2</span> 问题 {#问题}

1.  除了在 .tsx 文件中 import 对应的模块以外，还需要在 scss 文件中 import 对应的样式表，否则显式不正常。
2.  不要使用 Redux 模式，觉得不爽，尤其是和 RxJS 配合的时候。
3.  需要考虑 DeepLink


## <span class="section-num">3</span> 知识点 {#知识点}

1.  如何创建一个 Component.
2.  如何进行导航和 Url 跳转。
3.  如何进行网络访问。
4.  如何针对 json 进行序列化？


## <span class="section-num">4</span> 项目布局 {#项目布局}

<div class="PLAIN">
  <div></div>

src
    components
    containers
    pages
    store
        hero
            types.ts
            actions
            epics
            reducers
            constants
        index.ts

</div>
