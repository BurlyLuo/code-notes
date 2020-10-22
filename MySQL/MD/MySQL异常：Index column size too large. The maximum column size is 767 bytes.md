# MySQL异常：Index column size too large. The maximum column size is 767 bytes

## 问题描述

Mysql创建索引时报错：Index column size too large. The maximum column size is 767 bytes.

## 问题定位

建表时使用 utf8mb4 字符集，这是一个4 字节字符集。

当索引最大限制是 767 bytes时，那么一个 varchar 字段：767/4=191.75。

当 varchar 字段设置为默认的 255 长度，就超出了限制。

## 问题解决

1.将建立索引的 varchar 字段长度调低，使所有字段字节之和不超过 767 bytes。

2.建表时将 MySQL 的 charset 改为 utf8 ，collation 改为 utf8_general_ci。

3.将 MySQL 升级到 5.7 版本。

MySQL5.7 有默认参数 innodb_default_row_format=dynamic 和 innodb_large_prefix=on。这样的参数下是允许建立长字节索引的。

4.建表时添加 `ROW_FORMAT=DYNAMIC`
