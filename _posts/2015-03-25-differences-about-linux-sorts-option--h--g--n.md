---
layout: post
title: "differences about linux sort's option  h  g  n"
description: ""
category: Linux 
tags: [Linux, shell]
summary: sort中三个用于numeric-sort的选项的区别
---
{% include JB/setup %}

**Overview**

{{ page.summary }}

# sort 是什么

通常我们在shell中面临对简单数据进行排序的工作时会选择调用sort这个命令。sort由[coreutil](http://www.gnu.org/software/coreutils/)这个项目提供。

常用的选项包括：
* `-t<分隔字符>`: 指定排序某一列时所用的分隔字符
* `-k`:指定选择哪一列进行排序
* `-u`: 输出中去除重复行
* `-r`: sort结果默认是字典升序，`-r`改成降序
* `-o`: 输出到文件
* `-f`: 忽略大小写
* `-M`: 月份排序，JAN、FEB之类
* `-b`: 忽略行前前置空白
* `-n`: --numeric-sort, compare according to string numerical value
* `-h`: --human-numeric-sort, compare human readable numbers (e.g., 2K 1G)
* `-g`: --general-numeric-sort, compare according to general numerical value

# 对h, n, g的实验结果
![sort result](/img/2015-03-25-differences-about-linux-sorts-option/sort-result.png)

可以看出-n可以识别出前置0，但无法判断e, K, M等符号。-g可以识别e符号，但无法识别k，M等。-h可以识别k, M等，但是会判断2044K 小于1M，即-h会先按K，M等排序，再对数字进行一次稳定的排序。
