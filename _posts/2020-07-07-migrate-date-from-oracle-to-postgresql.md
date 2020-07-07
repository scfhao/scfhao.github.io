---
layout: post
title: "从Oracle迁移数据到PostgreSQL"
date: 2020-07-07 09:55:21 +0800
categories: 
---

公司的数据库由原来的 Oracle 转向 PostgreSQL 也有半年多了，大部分系统里的数据也已由 Oracle 转为 PostgreSQL，有 ora2pg 这个神器也使这项工作轻松了不少。我是在自己电脑里的 VirtualBox 里的 Linux 虚拟机里搭建的 ora2pg 环境，在虚拟机里用 ora2pg 还是有很多限制：

* 一是给虚拟机分配的内存太小，经常会有内存不够而无法继续的情况，这种情况比较好解决，加大虚拟机的内存就没事了。
* 另一个限制就是硬盘，如果源 Oracle 数据库中数据较多，ora2pg 输出导出文件时，硬盘容量占满后就无法输出了，当然这个问题也可以通过调整虚拟机磁盘大小来改善。
* 在 ora2pg 的配置文件里，只能设置导出对象的类型，比如 INSERT 就把所有的表数据导出来了，而不能只导出指定某个表的数据。

所以我有时候就需要只导某个表的数据到 PostgreSQL，思路很简单：

1. 从 Oracle 导出某个表的数据。
2. 将导出的数据导入PostgreSQL。

因为从 Oracle 导出的都是 INSERT 语句，语法比较简单，大部分 INSERT 语句都可以直接在 PostgreSQL 中运行，个别的数据导入会有问题，比如 Date，从 Oracle 中导出的日期格式如：“22-3月 -12”，而 PostgreSQL 可识别的格式是：“22-May-12”，如果数据量小的话可以自己手动替换一下，但是数据量稍大一点手动替换就会让你怀疑人生。本着能动脑就不动手的原则，加上久仰 Shell 命令里的 sed 命令的大名（sed命令可以用于替换文件内容）。写了个简单的脚本来做这件事。

Ora2pgDate.sh

```Bash
#!/bin/bash

if [[ ! -f "$1" ]]; then
	echo "Usage: ora2pgDate.sh /path/to/ora.sql"
	exit
fi

sed -i "" 's/DD-MON-RR/DD-Mon-YY/g' "$1"
sed -i "" 's/-1月 -/-Jan-/g' "$1"
sed -i "" 's/-1月 -/-Jan-/g' "$1"
sed -i "" 's/-2月 -/-Feb-/g' "$1"
sed -i "" 's/-3月 -/-Mar-/g' "$1"
sed -i "" 's/-4月 -/-Apr-/g' "$1"
sed -i "" 's/-5月 -/-May-/g' "$1"
sed -i "" 's/-6月 -/-Jun-/g' "$1"
sed -i "" 's/-7月 -/-Jul-/g' "$1"
sed -i "" 's/-8月 -/-Aug-/g' "$1"
sed -i "" 's/-9月 -/-Sep-/g' "$1"
sed -i "" 's/-10月-/-Oct-/g' "$1"
sed -i "" 's/-11月-/-Nov-/g' "$1"
sed -i "" 's/-12月-/-Dec-/g' "$1"
```

这样针对单表数据迁移就比较方便了。

1. 从 Oracle 导出某个表的数据。

我这边用的是“Oracle SQL Developer”客户端，直接选中一个表，右键导出，取消选中“导出DDL”，取消选中“显示方案”，点击下一步，这样导出的就是最简单的 INSERT 语句了。

2. 替换导出文件中的日期

```Bash
Ora2pgDate.sh 导出.sql
```

3. 导入 PostgreSQL

```Bash
psql -d dbname -h 127.0.0.1 -U uname -f 导出.sql
```
