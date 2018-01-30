---
title: mysql支持emoji
date: 2017-09-29 16:52:06
tags: mysql
---



MYSQL 5.5 之前，UTF8 编码只支持1-3个字节，只支持BMP这部分的unicode编码区，BMP是从哪到哪，到http://en.wikipedia.org/wiki/Mapping_of_Unicode_characters这里看，基本就是0000～FFFF这一区。 从MYSQL5.5开始，可支持4个字节UTF编码utf8mb4，一个字符最多能有4字节，所以能支持更多的字符集。

```
utf8mb4 is a superset of utf8
```

utf8mb4兼容utf8，且比utf8能表示更多的字符。 

emoji存入mysql需要字段字符集为utf8mb4。
## 解决办法 ##


<!-- more -->
修改/etc/my.cof文件

```
[mysqld]
#修改编码格式
character-set-server=utf8mb4                            
```
重启mysql

```
/etc/init.d/mysqld stop
/etc/init.d/mysqld start
```
修改字段编码格式
```
ALTER TABLE `table_name` MODIFY COLUMN `columb_name` VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL
```
然后就可以存储使用emoji表情了。