+++
title = "MacOS 系统上 Posgresql 的中文全文搜索配置和使用"
author = ["Evilee"]
date = 2020-03-08
lastmod = 2020-03-10T15:40:27+08:00
draft = false
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
authorbox = true
comments = true
toc = false
mathjax = true
+++

最近在做一个小方案，体验一下 PG 的全文搜索。由于我的工作环境是 MacOS, 所以记录一下，等搞定了才发现这个跟 Linux 没多大区别。

<!--more-->

安装 postgresql

```text
brew install postgresql
```


## pg\_jieba 方案 {#pg-jieba-方案}

安装

```text
brew install cmake
mkdir ~/tmp && cd ~/tmp && git clone https://github.com/jaiminpan/pg_jieba && cd pg_jieba
git submodule update --init --recursive
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=/usr/local/opt/postgres ..
make install
```

测试

```text
$ psql -d vapordb
psql (12.2)
Type "help" for help.

@vapordb=# CREATE EXTENSION pg_jieba;
CREATE EXTENSION
@vapordb=# SELECT * FROM to_tsvector('jiebacfg', '小明硕士毕业于中国科学院计算所，后在日本京都大学深造');
                                   to_tsvector
----------------------------------------------------------------------------------
 '中国科学院':5 '小明':1 '日本京都大学':10 '毕业':3 '深造':11 '硕士':2 '计算所':6
(1 row)

@vapordb=# \quit
```

在测试时，可以感觉到 jieba 的第一次分词有明显的延迟和卡顿，可以通过 Postgresq 预加载 jieba 的动态库和配置文件改善(/usr/local/var/postgres/postgresql.conf)。

```text
#------------------------------------------------------------------------------
# CUSTOMIZED OPTIONS
#------------------------------------------------------------------------------

# Add settings for extensions here
# pg_jieba
shared_preload_libraries = 'pg_jieba.so'  # (change requires restart)
# default_text_search_config='pg_catalog.simple'; default value
default_text_search_config='jiebacfg'; uncomment to make 'jiebacfg' as default
```


## zhparser 方案 {#zhparser-方案}

安装 scws

```text
brew install scws
scws -v
```

下载词典文件

```text
mkdir -p /usr/local/etc/scws
curl "http://www.xunsearch.com/scws/down/scws-dict-chs-utf8.tar.bz2" | tar xvjf -
mv dict.utf8.xdb /usr/local/etc/scws/
```

测试效果

```text
scws -c utf8 -d /usr/local/etc/scws/dict.utf8.xdb -r /usr/local/opt/scws/etc/rules.utf8.ini -M 9 "PostgreSQL 自带有一个简易的全文检索引擎"
PostgreSQL 自带 自 带 有 一个 一 个 简易 简 易 的 全文检索 全文 检索 全 文 检 索 引擎 引 擎
+--[scws(scws-cli/1.2.3)]----------+
| TextLen:   52                  |
| Prepare:   0.0007    (sec)     |
| Segment:   0.0002    (sec)     |
+--------------------------------+
```

安装 zhparser

```text
mkdir ~/tmp && cd ~/tmp
git clone https://github.com/amutu/zhparser.git && cd zhparser
make install
```

测试 zhparser

```text
$ psql -d vapordb
psql (12.2)
Type "help" for help.

@vapordb=# CREATE EXTENSION zhparser;
CREATE EXTENSION
@vapordb=# CREATE TEXT SEARCH CONFIGURATION zhcnsearch (PARSER = zhparser);
CREATE TEXT SEARCH CONFIGURATION
@vapordb=# ALTER TEXT SEARCH CONFIGURATION zhcnsearch ADD MAPPING FOR n,v,a,i,e,l,j WITH simple;
ALTER TEXT SEARCH CONFIGURATION
@vapordb=# SELECT to_tsvector('zhcnsearch', '人生苦短，我用 Python');
               to_tsvector
------------------------------------------
 'python':5 '人生':1 '用':4 '短':3 '苦':2
(1 row)

@vapordb=# \quit
```

大功告成。


## 对比 {#对比}

两种方案效果上差不多.

```text
$ psql -d vapordb
psql (12.2)
Type "help" for help.

@vapordb=# SELECT * FROM to_tsvector('jiebacfg', '小明硕士毕业于中国科学院计算所，后在日本京都大学深造');
                                   to_tsvector
----------------------------------------------------------------------------------
 '中国科学院':5 '小明':1 '日本京都大学':10 '毕业':3 '深造':11 '硕士':2 '计算所':6
(1 row)

@vapordb=# SELECT * FROM to_tsvector('zhcnsearch', '小明硕士毕业于中国科学院计算所，后在日本京都大学深造');
                                to_tsvector
---------------------------------------------------------------------------
 '中国科学院计算所':4 '小明':1 '日本京都大学':5 '毕业':3 '深造':6 '硕士':2
(1 row)

@vapordb=# \quit
```


## 如何使用 {#如何使用}

对于全文检索，有两种使用方式，大家可以权衡自己的内容进行选择。

1.  在搜索的时候进行分词，然后搜索对应的字段。
2.  提前把表中需要检索的字段进行分词，保存到一个新的字段中，再在这个字段上建立索引进行搜。

两种方案就是时间和空间的取舍：第一种方式创建索引简单，存储空间少，但是比较慢。第二种方案由于预先进行了分词并存储，浪费了空间，但是时间上肯定用得少。创建索引也有两种方案：gin 索引和 rum 索引。


### 第一种 {#第一种}

创建索引:

```sql
CREATE INDEX idx_xxxx ON xxxx_table USING gin(to_tsvector('jiebacfg',
COALESCE(xx_field, '') || COALESCE(xxx_field, '')));
```

查询：

```sql
EXPLAIN ANALYSE SELECT * FROM xxxx_table
        WHERE to_tsvector('jiebacfg', COALESCE(xx_field, '') || COALESCE(xxx_field, '')) @@
        to_tsquery('jiebacfg', '关键字或者句子');
```


### 第二种 {#第二种}

创建 tsv 字段和索引

```sql
ALTER TABLE xxxx_table ADD COLUMN tsv tsvector;
UPDATE xxxx_table SET tsv_field = to_tsvector('jiebacfg', COALESCE(xx_field, '') || COALESCE(xxx_field, ''));
CREATE INDEX idx_xxxx ON xxxx_table USING gin(tsv_field);
```

查询：

```sql
EXPLAIN ANALYSE SELECT * FROM xxxx_table WHERE tsv_field @@ to_tsquery('jiebacfg', '关键词或者句子');
```

当然因为是预先分词保存，所以需要在 update 的时候藉由 `触发器` 来更新 tsv 字段，。

```sql
CREATE TRIGGER tsvector_update BEFORE INSERT OR UPDATE OF (xx_field, xxx_field)
       ON xxxx_table FOR EACH ROW  EXECUTE PROCEDURE tsvector_update_trigger('tsv_field', 'jiebacfg', 'xx_field', 'xxx_field');
```


### rum 索引 {#rum-索引}

使用 rum 索引类似, 但是 rum 引擎默认是没有安装的，需要自己编译，暂时先不用了。

```sql
CREATE INDEX idx_xxxx ON xxxx_table USING rum(tsv_field rum_tsvector_ops);
```

另外 rum 还支持相似度的查询:

```sql
SELECT * FROM to_tsvector('jiebacfg', '小明硕士毕业于中国科学院计算所，后在日本京都大学深造');
SELECT * FROM rum_ts_distance(to_tsvector('jiebacfg', '小明硕士毕业于中国科学院计算所，后在日本京都大学深造') , to_tsquery('计算所'));
```
