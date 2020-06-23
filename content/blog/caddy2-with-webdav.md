+++
title = "Caddy2 Webdav 配置"
author = ["Evilee"]
date = 2020-06-23
lastmod = 2020-06-23T16:47:43+08:00
tags = ["Caddy", "Webdav"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

Caddy2 和 Caddy1 相比，配置文件差距巨大。这里记录下。
<!--more-->
MacOS 上默认安装的 Caddy2 是没有 Webdav 的，需要自己下载源代码编译。

```sh
brew install caddy
```

> ```sh
> #克隆代码
> git clone https://github.com/caddyserver/xcaddy
> #进入文件夹
> cd xcaddy
> #安装 xcaddy 工具
> go get -u github.com/caddyserver/xcaddy/cmd/xcaddy
> #编译 caddy，指定版本为 v2.0.0
> xcaddy build v2.0.0  --with  github.com/mholt/caddy-webdav
> #查看编译了什么模块，应该有 http.handlers.webdav
> ./caddy list-modules
> #把新版 caddy，复制到系统路径
> cp ./caddy /usr/local/bin/
> #如果是 windows 的话
> ./caddy.exe list-modules
> #copy
> cp caddy /usr/local/Celler/caddy/2.0.0/bin/caddy
> ```

配置文件

````text
{
        order webdav last
}
http://localhost:8080 {
        encode gzip
        root * ./
        log {
                output file ./access.log
        }
        webdav {
                root ./
        }
}
````

运行

````sh
caddy run -config Caddyfile
````
