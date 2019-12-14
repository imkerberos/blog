+++
title = "在 Debian 6.0 (Squeeze) 上安装 MongoDB gridfs-fuse"
author = ["Eviler"]
date = 2012-04-01
lastmod = 2019-12-15T01:47:32+08:00
tags = ["debian", "squeeze", "nginx", "mongodb"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
+++

nginx-gridfs 不是很稳定, 在前一段时间折腾了几次都没有成功.
最近经过查询资料, 发现可 以使用 fuse 模块来把 gridfs 做成文件系统,
而且性能要比 nginx-gridfs 高得多.
<!--more-->

从 [CoffeePowered](http://www.coffeepowered.net/2010/02/17/serving-files-out-of-gridfs/) 上发现了一篇 GridFS 的三种方式的性能测试文章,
由于我的 nginx-gridfs 一直没有配置成功, 无法得出数据, 这里只好引用一下:


## Filesystem read through Apache {#filesystem-read-through-apache}

```text
[chris@polaris conf]# ab -n 50000 -c 10 http://advice/images/embed/alliance-60.png

Server Software:        Apache/2.2.13
Server Hostname:        advice
Server Port:            80

Document Path:          /images/embed/normal_alliance-60.png
Document Length:        31596 bytes

Concurrency Level:      10
Time taken for tests:   1.904 seconds
Complete requests:      5000
Failed requests:        0
Write errors:           0
Total transferred:      159463760 bytes
HTML transferred:       158043192 bytes
Requests per second:    2625.37 [#/sec] (mean)
Time per request:       3.809 [ms] (mean)
Time per request:       0.381 [ms] (mean, across all concurrent requests)
Transfer rate:          81767.87 [Kbytes/sec] received

Connection Times (ms)
            min  mean[+/-sd] median   max
Connect:        0    1   0.4      1       4
Processing:     1    3   0.5      3       6
Waiting:        0    1   0.4      1       4
Total:          2    4   0.4      4       8

Percentage of the requests served within a certain time (ms)
50%      4
66%      4
75%      4
80%      4
90%      4
95%      4
98%      5
99%      5
100%      8 (longest request)
```


## Filesystem read through nginx {#filesystem-read-through-nginx}

```text
[chris@polaris conf]# ab -n 50000 -c 10 http://advice:81/images/embed/normal_alliance-60.png

Server Software:        nginx/0.8.33
Server Hostname:        advice
Server Port:            81

Document Path:          /images/embed/normal_alliance-60.png
Document Length:        31596 bytes

Concurrency Level:      10
Time taken for tests:   7.623 seconds
Complete requests:      50000
Failed requests:        0
Write errors:           0
Total transferred:      1590513618 bytes
HTML transferred:       1579863192 bytes
Requests per second:    6559.31 [#/sec] (mean)
Time per request:       1.525 [ms] (mean)
Time per request:       0.152 [ms] (mean, across all concurrent requests)
Transfer rate:          203763.10 [Kbytes/sec] received

Connection Times (ms)
            min  mean[+/-sd] median   max
Connect:        0    0   0.2      0       9
Processing:     1    1   0.4      1      11
Waiting:        0    0   0.1      0       9
Total:          1    1   0.5      1      12

Percentage of the requests served within a certain time (ms)
50%      1
66%      1
75%      1
80%      2
90%      2
95%      2
98%      3
99%      3
100%     12 (longest request)
```

nginx _screams_. At 6500 requests/sec, it's blisteringly fast.


## GridFS read through nginx-gridfs {#gridfs-read-through-nginx-gridfs}

```text
[chris@polaris conf]# ab -n 5000 -c 10 http://advice:81/images/gfs/uploads/user/avatar/4b7b2c0e98db7475fc000003/normal_alliance-60.png

Server Software:        nginx/0.8.33
Server Hostname:        advice
Server Port:            81

Document Path:          /images/gfs/uploads/user/avatar/4b7b2c0e98db7475fc000003/normal_alliance-60.png
Document Length:        31596 bytes

Concurrency Level:      10
Time taken for tests:   4.613 seconds
Complete requests:      5000
Failed requests:        0
Write errors:           0
Total transferred:      158580000 bytes
HTML transferred:       157980000 bytes
Requests per second:    1083.88 [#/sec] (mean)
Time per request:       9.226 [ms] (mean)
Time per request:       0.923 [ms] (mean, across all concurrent requests)
Transfer rate:          33570.65 [Kbytes/sec] received

Connection Times (ms)
            min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       1
Processing:     1    9   4.7      9     103
Waiting:        1    9   4.7      9     102
Total:          2    9   4.7      9     103

Percentage of the requests served within a certain time (ms)
50%      9
66%      9
75%      9
80%      9
90%      9
95%      9
98%      9
99%     11
100%    103 (longest request)
```


## Rails Metal handler {#rails-metal-handler}

```text
[chris@polaris nginx-gridfs]$ ab -n 250 -c 4  http://advice/images/gfs/uploads/user/avatar/4b7b2c0e98db7475fc000003/normal_alliance-60.png

Server Software:        Apache/2.2.13
Server Hostname:        advice
Server Port:            80

Document Path:          /images/gfs/uploads/user/avatar/4b7b2c0e98db7475fc000003/normal_alliance-60.png
Document Length:        31596 bytes

Concurrency Level:      4
Time taken for tests:   4.646 seconds
Complete requests:      250
Failed requests:        0
Write errors:           0
Total transferred:      7960000 bytes
HTML transferred:       7899000 bytes
Requests per second:    53.81 [#/sec] (mean)
Time per request:       74.338 [ms] (mean)
Time per request:       18.585 [ms] (mean, across all concurrent requests)
Transfer rate:          1673.10 [Kbytes/sec] received

Connection Times (ms)
            min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       1
Processing:    15   74  75.6     34     287
Waiting:        0   72  75.8     30     276
Total:         15   74  75.6     34     288

Percentage of the requests served within a certain time (ms)
50%     34
66%     39
75%    139
80%    192
90%    201
95%    210
98%    239
99%    245
100%    288 (longest request)
```


## 总结比较 {#总结比较}

| Solution                            | Requests/second | Apache FS | Nginx FS | Nginx GridFS | Apache Ruby |
|-------------------------------------|-----------------|-----------|----------|--------------|-------------|
| Filesystem via Apache               | 2625.37         | -         | 40.03    | 242.2392     | 4,878.96    |
| Filesystem via Nginx                | 6559.31         | 249.84    | -        | 605.17       | 12,189.76   |
| GridFS via nginx module             | 1083.88         | 41.28     | 16.52    | -            | 2014.27     |
| Railsls metal handler via Passenger | 53.81           | 2.05      | 0.82     | 4.96         | -53         |


## 参考资料 {#参考资料}

[CoffeePowered]
<http://www.coffeepowered.net/2010/02/17/serving-files-out-of-gridfs/>
"CoffeePowered 上的性能测试"
