+++
title = "在 Debian 6.0 (Squeeze) 上安装 nginx_gridfs"
author = ["Evilee"]
date = 2012-04-01
lastmod = 2019-12-15T01:49:43+08:00
tags = ["debian", "squeeze", "nginx", "mongodb"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
+++

网站或者论坛对于用户上传的图片管理比较麻烦, 尤其是在 Web 具有横向扩展能力的情况下.
但存把图片存放在 web 服务器的某个目录先已经不能满足各台 Web 主机同步要求. 这种情况下使用共享式文件系统或者分布式文件系统是一个比较好的选择.
<!--more-->


## 背景 {#背景}

由于前台有数台 Web Server, 用户上传的图片的同步是一个很大的问题,
调查了一下是否存在 分布式文件系统上, 但是 hadoop 的 HDFS 的 block 大小是
64M, 一个用户的图片撑死也就 100k 左右而已, 所以, 肯定不能使用 HDFS 了,
而且 HDFS 的性能也未必满足实时图片的需要. 正好前台的数据库是 MongoDB,
所以把图片保存在 MongoDB 的 GridFS 上是一个不错的选择, 虽然 GridFS 的
chunk 大小是 256K, 浪费一点救浪费一点吧.


## 小折腾 {#小折腾}

写了一个脚本和 Web 从 GridFS 里面读取图片返回给前台,
同时写了一个性能测试程序, 发现 效果不是很好. 100
个并发请求大约需要的时间在 0.5s - 10s 之间. 比用 Apache 进行静态文件存储访问还低, 不是很满意.

前台 nginx 做静态文件需要编写 _try\_files_ 脚本来做缓存, squeeze 中的
nginx 支持的模 块还没有 perl 和 lua, 需要自己编译. 而且对于 perl
不是很熟悉. 想起来就发怵.

添加了 nginx 官方源, nginx 官方源的版本要比 squeeze 仓库中的要新一些,
有可能有 perl 的支持吧:

```text
#cat /etc/apt/sources.list.d/kereros.list
deb http://nginx.org/packages/debian/ squeeze nginx
deb-src http://nginx.org/packages/debian/ squeeze nginx
```

查看 nginx 模块支持

```text
#/usr/sbin/nginx -V
nginx version: nginx/1.0.14
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx/ --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-mail --with-mail_ssl_module --with-file-aio --with-ipv6
```

看来也是没有 Perl 的支持啊, 郁闷. 只好自己动手丰衣足食了. 经过
[Google](http://www.google.com.hk) 之后, 发现 nginx 可以配合
nginx-gridfs 进行 GridFS 的文件系统的访问. 所以就打算用这个了, 无论如何人家是 _C_ 实现的, 总会比我用 [Python](http://www.python.org)
直接访问要快一些吧. 说干就干, 先准 备 deb 包的打包工具, 下载 nginx
的官网 1.0.14 稳定版本的源代码:

```text
#apt-get install build-dep nginx
#mkdir nginx-build
#cd nginx-build
#apt-get source nginx
```

准备 nginx-gridfs 的源代码, 由于 nginx-gridfs 源代码在
[GitHub](http://www.github.com) 上保存着, 所以用 git 很容易获取下来.
按照 nginx-gridfs 的官方仓库的说明进行安装:

```text
#cd nginx-1.0.14
#git clone  https://github.com/mdirolf/nginx-gridfs.git
#cd nginx-gridfs
#git checkout v0.8
#git submodule init
#git submodule update
#cd ..
```

修改缺省配置: `debian/rules`, 在 `dh_auto_build` 部分的 `./configure`
后添加 `--add-module=nginx-gridfs`. 最后文件如下:

```text
#!/usr/bin/make -f

#export DH_VERBOSE=1

%:
    dh $@
override_dh_auto_configure: configure_debug

override_dh_strip:
    dh_strip -Xdebug

override_dh_auto_build:
    dh_auto_build
    mv objs/nginx objs/nginx.debug
    ./configure \
        --prefix=/etc/nginx/ \

.......... # 中间省略一些

        --with-ipv6 \
        --add-module=nginx-gridfs
    dh_auto_build
configure_debug:
    ./configure \
        --prefix=/etc/nginx/ \
        --sbin-path=/usr/sbin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \

.......... # 中间省略一些

        --with-file-aio \
        --with-ipv6 \
        --add-module=nginx-gridfs \
        --with-debug

.......... # 其他部分
```

使用 --add-module=nginx-gridfs 来添加模块, `nginx-gridfs` 是
nginx-gridfs 的源代码路径. nginx 的模块 是直接编译到 nginx
可执行文件中的, 并非是使用动态库实现的, 当初我还找了半天 nginx-gridfs
模块到底编译 到哪个 lib 中去了, :) 真土.

下面就正式开始制作 deb 安装包了:

```text
#fakeroot dpkg-buildpackage
```

经过一系列的编译之后终于编译成功, 还好中间没有出现什么错误.

```text
kerberos@debian:~/nginx/1.0.14/nginx-1.0.14$ ls ../
nginx-1.0.14                          nginx_1.0.14-2~squeeze.debian.tar.gz  nginx-debug_1.0.14-2~squeeze_amd64.deb
nginx_1.0.14-2~squeeze_amd64.changes  nginx_1.0.14-2~squeeze.dsc
nginx_1.0.14-2~squeeze_amd64.deb      nginx_1.0.14.orig.tar.gz
```

可以看到多了 2 个 deb 文件, 就是 debian 的安装包了. nginx
开头的是正式安装包, nginx-debug 开头的是带 调试信息的包.
先删除原来官方的版本, 在安装自己编译出来的正式版本,
生产环境当然不能使用调试模式的.

```text
kerberos@debian: sudo apt-get remove --purge nginx
kerberos@debian: sudo dpkg -i ../nginx_1.0.14-2~squeeze_amd64.deb
```

下面就进行测试了, 修改 nginx 的配置文件 `vim /etc/nginx/conf.d/default`

```text
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # 以下配置 gridfs, 使用 192.168.1.222 上的 mongodb

    location /image/ {
        gridfs test field=filename type=string;
        mongo 192.168.1.222:27017;
    }

    ........ #此处省略若干配置
```

重新启动 nginx `sudo /etc/init.d/nginx restart`, 可以看到 nginx
正常启动了. 访问一下试试:

```text
kerberos@debian: wet http://localhost/image/kerberos.png
```

`kerberos.png` 是已经传到 gridfs 里面的图片, 这一访问, 居然把 nginx
搞死了一个进程. 看来 nginx-gridfs 还不是很稳定. 又在 github 上逛了半天,
找到了几个版本:

-   <https://github.com/viotti/nginx-gridfs/> 号称修复了 replica set
    连接的版本


## 结果 {#结果}

尝试过各种组合, `nginx 0.7.67` + `nginx-gridfs` 和 `nginx 1.0.14` +
`nginx-gridfs` 以及 `mongo-c-driver` 始终没有顺利运行起来, 总结结果如下:

-   gridfs 配置不生效, 访问 /image/kerberos.png 没有任何 mongodb
    的链接信息, 而是访问到了 nginx 的缺省 root 目录下.
-   数据库有连接响应, 但是浏览器端一直 reading, 后台的 log 显示 connection
    droped. 之后的访问不再出现数据库链接信息, 而是直接访问到 nginx 缺省的
    root 目录下
-   后台出现 unknown exception 错误.


## 结论 {#结论}

看起来 nginx-gridfs 不是很稳定, 或许和 nginx, mongo-c-driver
的版本有强烈的依赖关系, 总之不是很好配置. 一直没有成功, 迫于时间压力,
已经没有时间再折腾了. 即使折腾成功, 从过程来看, 很难让人相信这个 nginx
模块的成熟性和稳定性. 自己平时 玩玩还可以, 如果放在生产系统上,
肯定让人寝食难安的.
