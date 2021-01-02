---
layout: post
title: MySQL从入门到删库跑路
tags: [MySQL, 数据库]
index_img: https://w.wallhaven.cc/full/nk/wallhaven-nkx9d7.jpg
banner_img: https://w.wallhaven.cc/full/nk/wallhaven-nkx9d7.jpg
categories: [数据库]
date: 2018-01-19 19:25:23
---

Mysql是最流行的关系型数据库管理系统，在WEB应用方面MySQL是最好的RDBMS(Relational Database Management System：关系数据库管理系统)应用软件之一。
<!-- more -->

# 一、什么是数据库？

数据库（Database）是按照数据结构来组织、存储和管理数据的仓库，每个数据库都有一个或多个不同的API用于创建，访问，管理，搜索和复制所保存的数据。我们也可以将数据存储在文件中，但是在文件中读写数据速度相对较慢。所以，现在我们使用关系型数据库管理系统（RDBMS）来存储和管理的大数据量。所谓的关系型数据库，是建立在关系模型基础上的数据库，借助于集合代数等数学概念和方法来处理数据库中的数据。
RDBMS即关系数据库管理系统(Relational Database Management System)的特点：

* 数据以表格的形式出现
* 每行为各种记录名称
* 每列为记录名称所对应的数据域
* 许多的行和列组成一张表单
* 若干的表单组成database

# 二、RDBMS 术语

* 数据库: 数据库是一些关联表的集合。.
* 数据表: 表是数据的矩阵。在一个数据库中的表看起来像一个简单的电子表格。
* 列: 一列(数据元素) 包含了相同的数据, 例如邮政编码的数据。
* 行：一行（=元组，或记录）是一组相关的数据，例如一条用户订阅的数据。
* 冗余：存储两倍数据，冗余降低了性能，但提高了数据的安全性。
* 主键：主键是唯一的。一个数据表中只能包含一个主键。你可以使用主键来查询数据。
* 外键：外键用于关联两个表。
* 复合键：复合键（组合键）将多个列作为一个索引键，一般用于复合索引。
* 索引：使用索引可快速访问数据库表中的特定信息。索引是对数据库表中一列或多列的值进行排序的一种结构。类似于书籍的目录。
* 参照完整性: 参照的完整性要求关系中不允许引用不存在的实体。与实体完整性是关系模型必须满足的完整性约束条件，目的是保证数据的一致性。


# 三、使用MySQL Client执行简单的SQL命令

连接到MySQL服务器上，在MAC上，可以先到**系统偏好设置**里将MySQL服务打开：
![](/img/mysql/start_MySQL.png)
![](/img/mysql/start_MySQL2.png)

在终端输入命令：

```sql
ecarry:~ ecarry$ mysql -u root -p
```
>其中 -u为用户，后面紧跟root，意为以root账户登录， -p为密码

```sql
ecarry:~ ecarry$ mysql -u root -p
Enter password: 
mysql> 
```

# 四、更改密码
老版本的MySQL安装成功后，默认的root用户和密码为空，可以使用命令来创建root用户和密码：
>我使用MySQL的版本是5.6.39，可以用`mysql --version`命令来查看当前版本

```sql
ecarry:~ ecarry$ mysql --version
mysql  Ver 14.14 Distrib 5.6.39, for macos10.13 (x86_64) using  EditLine wrapper
```

使用`mysqladmin -u root password "new_password"`来更改密码:
```sql
ecarry:~ ecarry$ mysqladmin -u root password new_password
```

# 五、基本操作

## 5.1 MySQL用户设置

添加MySQL用户，在mysql数据库中的user表添加新用户即可。
下面实例将添加用户名为guest，密码为guest123，权限可进行**SELECT**,**INSERT**和**UPDATE**操作权限：

```sql
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> INSERT INTO user
    -> (host,user,password,
    -> select_priv,insert_priv,update_priv)
    -> VALUES('localhost','guest',
    -> PASSWORD('guest123'),'Y','Y','Y');
Query OK, 1 row affected, 3 warnings (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT host,user,password FROM user WHERE user = 'guest';
+-----------+-------+-------------------------------------------+
| host      | user  | password                                  |
+-----------+-------+-------------------------------------------+
| localhost | guest | *F1573429579994EEA4459170FDAC55DF96C4BBE6 |
+-----------+-------+-------------------------------------------+
1 row in set (0.00 sec)
```

在添加用户时，请注意使用MySQL提供的PASSWORD()函数来对密码进行加密，可以在后面看到密码为：*F1573429579994EEA4459170FDAC55DF96C4BBE6

**注意：**在MySQL5.7中user表的password换成了authentication_string。
**注意：**`FLUSH PRIVILEGES`语句，这个命令执行后会重新载入授权表。不使用这个命令，将无法使用新创建的用户来连接MySQL服务器，除非重启MySQL服务器。
在创建用户时，可以给用户指定权限，在对应权限列中，在插入语句中设置为'Y'即可。

**用户权限表：**
* SELECT_PRIV：选择数据
* INSERT_PRIV：插入数据
* UPDATE_PRIV：修改现有数据
* DELETE_PRIV：删除现有数据
* CREATE_PRIV：创建新的数据库和表
* DROP_PRIV：删除现有数据库和表
* RELOAD_PRIV：执行刷新和重新加载MySQL所用各种内部缓存的特定命令，包括日志，权限，主机，查询和表
* SHUTDOWN_PRIV：关闭MySQL服务器
* PROCESS_PRIV：可以通过SHOW PROCESSLIST命令查看其它用户的进程
* FILE_PRIV：执行SELECT INTO OUTFILE 和LOADDATA INFILE命令
* GRANT_PRIV：将已经授予给该用户自己的权限再授予其它用户。
* REFERENCES_PRIV
* INDEX_PRIV：创建和删除表索引
* ALTER_PRIV：重命名和修改表结构


### 5.1.1 管理MySQL的命令

 **SHOW DATABASES**
列出MySQL数据库管理系统的数据库列表：

```sql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| test               |
+--------------------+
2 rows in set (0.00 sec)
```

 **USE 数据库名**
选择要操作的MySQL数据库，使用这命令后所有mysql命令都只针对该数据库。
```sql
mysql> USE test;
Database changed
```

 **SHOW TABLES**
显示指定数据库的所有表，使用该命令前需要使用USE命令来选择要操作的数据库。
```sql
mysql> USE mysql;
Database changed
mysql> SHOW TABLES;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| event                     |
| time_zone_transition_type |
| user                      |
+---------------------------+
28 rows in set (0.00 sec)
```
 **SHOW COLUMNS FROM 数据表**
显示数据表的属性，属性类型，主键信息，是否为NULL，默认值等 其他信息。
```sql
mysql> SHOW COLUMNS FROM user;
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Field                  | Type                              | Null | Key | Default               | Extra |
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Host                   | char(60)                          | NO   | PRI |                       |       |
| User                   | char(16)                          | NO   | PRI |                       |       |
| Password               | char(41)                          | NO   |     |                       |       |
| Select_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| ssl_type               | enum('','ANY','X509','SPECIFIED') | NO   |     |                       |       |
| x509_subject           | blob                              | NO   |     | NULL                  |       |
| max_user_connections   | int(11) unsigned                  | NO   |     | 0                     |       |
| plugin                 | char(64)                          | YES  |     | mysql_native_password |       |
| authentication_string  | text                              | YES  |     | NULL                  |       |
| password_expired       | enum('N','Y')                     | NO   |     | N                     |       |
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
43 rows in set (0.00 sec)
```
 **SHOW INDES FROM 数据表**
显示数据表的详细索引信息，包括PRIMARY KEY（主键）。
```sql
mysql> SHOW INDEX FROM user;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| user  |          0 | PRIMARY  |            1 | Host        | A         |        NULL |     NULL | NULL   |      | BTREE      |         |               |
| user  |          0 | PRIMARY  |            2 | User        | A         |           7 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.01 sec)
```
## 5.2 MySQL创建数据库

**使用mysqladmin创建数据库：**
```sql
ecarry:~ ecarry$ mysqladmin -u root -p create test
Enter password:
```

## 5.3 MySQL删除数据库

 **使用mysqladmin删除数据库：**
```sql
ecarry:~ ecarry$ mysqladmin -u root -p drop test
Enter password:

Dropping the database is potentially a very bad thing to do.
Any data stored in the database will be destroyed.

Do you really want to drop the 'test' database [y/N] y
Database "test" dropped
```

## 5.4 MySQL数据类型

MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串（字符）类型。

 **数值类型**

MySQL支持所有标准SQL数值数据类型。
这些类型包括严格数值数据类型(INTEGER、SMALLINT、DECIMAL和NUMERIC)，以及近似数值数据类型(FLOAT、REAL和DOUBLE PRECISION)。
关键字INT是INTEGER的同义词，关键字DEC是DECIMAL的同义词。
BIT数据类型保存位字段值，并且支持MyISAM、MEMORY、InnoDB和BDB表。
作为SQL标准的扩展，MySQL也支持整数类型TINYINT、MEDIUMINT和BIGINT。下面的表显示了需要的每个整数类型的存储和范围。

 | 类型        | 大小      |  范围（有符号）  |  范围（无符号） |  用途    |
 | :-------- :  |:-----:   |:----:        |  :----:    |:----: |
 | TINYINT    | 1字节     | (-128，127)   | (0，255)        | 小整数值 |
 | SMALLINT   |2字节     |(-32 768，32 767)|(0，65 535)     |大整数值  |
 | MEDIUMINT  | 3字节    |(-8 388 608，8 388 607)|(0，16 777 215)| 大整数值 |
 |INT或INTEGER| 4字节    |(-2 147 483 648，<br>2 147 483 647)|(0，4 294 967 295)|大整数值 |
 | BIGINT     |8字节    |(-9 233 372 036 854 775 808，<br>9 223 372 036 854 775 807)|(0，18 446 744 073 709 551 615)|极大整数值  |
 | FLOAT     | 4字节    |(-3.402 823 466 E+38，<br>-1.175 494 351 E-38)，0，(1.175 494 351 E-38，<br>3.402 823 466 351 E+38)|0，(1.175 494 351 E-38，<br>3.402 823 466 E+38)|单精度/浮点数值|
 |DOUBLE     |8字节    |(-1.797 693 134 862 315 7 E+308，<br>-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，<br>1.797 693 134 862 315 7 E+308)| 0，(2.225 073 858 507 201 4 E-308，<br>1.797 693 134 862 315 7 E+308)                      |双精度/浮点数值|
 | DECIMAL   |对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2|依赖于M和D的值|依赖于M和D的值|小数值|


 **日期和时间类型**

表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR。
每个时间类型有一个有效值范围和一个"零"值，当指定不合法的MySQL不能表示的值时使用"零"值。
TIMESTAMP类型有专有的自动更新特性。

 | 类型        | 大小(字节） |  范围                 |  格式       |  用途     |
 | :-------- :  |:-----:     |:----:                 |  :----:    |:----:    |
 |DATE        |   3       |1000-01-01<br>9999-12-31  |YYYY-MM-DD  |日期值     |
 |TIME         |  3       |-838:59:59'/'838:59:59|HH:MM:SS    |时间值或持续时间|
 |YEAR         |   1       |1901/2155              |YYYY        |年份值     |
 |DATETIME     |    8      |1000-01-01 00:00:00<br>9999-12-31 23:59:59|YYYY-MM-DD<br>HH:MM:SS|混合日期和<br>时间值|
 |TIMESTAMP|     4         |1970-01-01 00:00:00/2038|YYYYMMDD<br>HHMMSS|混合日期和时间值<br>时间戳|

 **字符串类型**

字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。

 | 类型        | 大小(字节） |  用途      |
 | :-------- :  |:-----:      |:----:     |
 |CHAR|0-255|定长字符串|
 |VARCHAR	|0-65535 |	变长字符串|
 |TINYBLOB|	0-255	|不超过 255 个字符的二进制字符串|
 |TINYTEXT|	0-255|	短文本字符串|
 |BLOB|	0-65 535	|二进制形式的长文本数据|
 |TEXT|	0-65 535	|长文本数据|
 |MEDIUMBLOB	|0-16 777 215	|二进制形式的中等长度文本数据|
 |MEDIUMTEXT	|0-16 777 215|中等长度文本数据|
 |LONGBLOB	|0-4 294 967 295|	二进制形式的极大文本数据|
 |LONGTEXT	|0-4 294 967 295	|极大文本数据|

* CHAR和VARCHAR类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。
* BINARY和VARBINARY类类似于CHAR和VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。
* BLOB是一个二进制大对象，可以容纳可变数量的数据。有4种BLOB类型：TINYBLOB、BLOB、MEDIUMBLOB和LONGBLOB。它们只是可容纳值的最大长度不同。
* 有4种TEXT类型：TINYTEXT、TEXT、MEDIUMTEXT和LONGTEXT。这些对应4种BLOB类型，有相同的最大长度和存储需求。


## 5.5 MySQL创建数据表

创建MySQL数据表需要以下信息：
* 表名
* 表字段名
* 定义每个表字段

**语法**

```sql
CREATE TABLE table_name (column_name column_type);
```
下面通过实例在test数据库中创建数据表test_tbl：
```sql
mysql> USE test;
Database changed

mysql> SHOW TABLES;
Empty set (0.00 sec)

mysql> CREATE TABLE test_tbl(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> name CHAR(20) NOT NULL,
    -> age int(9) NOT NULL,
    -> sex CHAR(2) NOT NULL,
    -> submission_date DATE,
    -> PRIMARY KEY (id));
Query OK, 0 rows affected (0.03 sec)
```

创建成功后，可以通过命令来查看表结构：

```sql
mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| test_tbl       |
+----------------+
1 row in set (0.00 sec)

mysql> desc test_tbl;
+-----------------+----------+------+-----+---------+----------------+
| Field           | Type     | Null | Key | Default | Extra          |
+-----------------+----------+------+-----+---------+----------------+
| id              | int(11)  | NO   | PRI | NULL    | auto_increment |
| name            | char(20) | NO   |     | NULL    |                |
| age             | int(9)   | NO   |     | NULL    |                |
| sex             | char(2)  | NO   |     | NULL    |                |
| submission_date | date     | YES  |     | NULL    |                |
+-----------------+----------+------+-----+---------+----------------+
5 rows in set (0.00 sec)
```

## 5.6 MySQL删除数据表

**语法**
`DROP TABLE 数据表名`

```sql
mysql> USE test;
Database changed

mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| test_tbl       |
+----------------+
1 row in set (0.00 sec)

mysql> DROP TABLE test_tbl;
Query OK, 0 rows affected (0.8 sec)

mysql> SHOW TABLES;
Empty set (0.01 sec)
```

## 5.7 MySQL插入数据

MySQL 表中使用 **INSERT INTO** SQL语句来插入数据。

**实例**
向test_tbl插入数据：
```sql
mysql> INSERT INTO test_tbl
    -> (name,age,sex,submission_date)
    -> VALUES
    -> ("pangzi W","25","M",NOW());
Query OK, 1 row affected, 1 warning (0.00 sec)

mysql> SELECT * FROM test_tbl;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  1 | pangzi W |  25 | M   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
1 row in set (0.00 sec)
```
* 没有提供id的数据，因为该字段我们在创建表的时候已经设置它为 AUTO_INCREMENT(自动增加) 属性，该字段会自动递增而不需要我们去设置。
* NOW() 是一个 MySQL 函数，该函数返回日期和时间。

## 5.8 MySQL查询数据

MySQL 数据库使用SQL SELECT语句来查询数据。
**语法**

```sql
SELECT column_name,column_name
FROM table_name
[WHERE Clause]
[LIMIT N][ OFFSET M]
```

* 查询语句中你可以使用一个或者多个表，表之间使用逗号(,)分割，并使用WHERE语句来设定查询条件。
* SELECT 命令可以读取一条或者多条记录。
* 你可以使用星号（*）来代替其他字段，SELECT语句会返回表的所有字段数据
* 你可以使用 WHERE 语句来包含任何条件。
* 你可以使用 LIMIT 属性来设定返回的记录数。
* 你可以通过OFFSET指定SELECT语句开始查询的数据偏移量。默认情况下偏移量为0。

**实例**
返回数据表所有数据：

```sql
mysql> SELECT * FROM bili;
+----------+----------+---------+--------+----------+--------+--------+
| aid      | view     | danmaku | reply  | favorite | coin   | share  |
+----------+----------+---------+--------+----------+--------+--------+
|        0 |        0 |       0 |      0 |        0 |      0 |      0 |
|        2 |       -1 |    1866 |   4888 |     1604 |    480 |   1292 |
|        7 |  1158359 |   22770 |  23613 |     9991 |   2189 |    538 |
|        9 |  1237494 |    5025 |   4858 |    10015 |    994 |    833 |
|       11 |   168711 |    2087 |   2499 |     1336 |    165 |     33 |
|       12 |   495352 |    1392 |   2501 |     3743 |    357 |    310 |
|       16 |   108035 |    1492 |   1144 |      874 |     81 |     36 |
|       20 |   157311 |    3833 |   1788 |     3347 |    361 |    104 |
|       23 |       -1 |   40371 |   9711 |    29289 |   2570 |   4562 |
|       24 |   196740 |    5105 |    855 |     2897 |    296 |    124 |
|       25 |    62049 |     931 |    448 |      726 |     56 |     32 |
|       26 |    86121 |    2166 |    574 |     1800 |    252 |    124 |
|       28 |    54482 |    1474 |    627 |      396 |     49 |     17 |
|       31 |    60681 |    1375 |    457 |      306 |     14 |     24 |
+----------+----------+---------+--------+----------+--------+--------+
14 row in set (0.00 sec)
```

## 5.9 MySQL WHERE子句

有条件的选取数据，将WHERE子句添加到SELECT语句中。
**语法**

```sql
SELECT field1, field2,...fieldN FROM table_name1, table_name2...
[WHERE condition1 [AND [OR]] condition2.....
```
* 查询语句中你可以使用一个或者多个表，表之间使用逗号, 分割，并使用WHERE语句来设定查询条件。
* 你可以在 WHERE 子句中指定任何条件。
* 你可以使用 AND 或者 OR 指定一个或多个条件。
* WHERE 子句也可以运用于 SQL 的 DELETE 或者 UPDATE 命令。
* WHERE 子句类似于程序语言中的 if 条件，根据 MySQL 表中的字段值来读取指定的数据。

**实例**
搜索数据表中的属性为女性数据：

```sql
mysql> SELECT * FROM test_tbl;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  1 | pangzi W |  25 | M   | 2018-01-22      |
|  2 | san A    |  25 | M   | 2018-01-22      |
|  3 | sao A    |  24 | M   | 2018-01-22      |
|  4 | juju GOU |  26 | M   | 2018-01-22      |
|  5 | li XIAO  |  23 | W   | 2018-01-22      |
|  6 | nv JI    |  18 | W   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
6 rows in set (0.00 sec)

mysql> select * FROM test_tbl WHERE sex = 'W';
+----+---------+-----+-----+-----------------+
| id | name    | age | sex | submission_date |
+----+---------+-----+-----+-----------------+
|  5 | li XIAO |  23 | W   | 2018-01-22      |
|  6 | nv JI   |  18 | W   | 2018-01-22      |
+----+---------+-----+-----+-----------------+
2 rows in set (0.01 sec)
```

## 5.10 MySQL UPDATE查询

如果我们需要修改或更新 MySQL 中的数据，我们可以使用 SQL UPDATE 命令来操作。.
**语法**

以下是 UPDATE 命令修改 MySQL 数据表数据的通用 SQL 语法：

```sql
UPDATE table_name SET field1=new-value1, field2=new-value2
[WHERE Clause]
```

* 你可以同时更新一个或多个字段。
* 你可以在 WHERE 子句中指定任何条件。
* 你可以在一个单独表中同时更新数据。

**实例**

更新test_tbl数据表id为1的name数据：

```sql
mysql> SELECT * FROM test_tbl;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  1 | pangzi W |  25 | M   | 2018-01-22      |
|  2 | san A    |  25 | M   | 2018-01-22      |
|  3 | sao A    |  24 | M   | 2018-01-22      |
|  4 | juju GOU |  26 | M   | 2018-01-22      |
|  5 | li XIAO  |  23 | W   | 2018-01-22      |
|  6 | nv JI    |  18 | W   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
6 rows in set (0.00 sec)

mysql> UPDATE test_tbl SET name='nv JI' WHERE id=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT * FROM test_tbl;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  1 | nv JI    |  25 | M   | 2018-01-22      |
|  2 | san A    |  25 | M   | 2018-01-22      |
|  3 | sao A    |  24 | M   | 2018-01-22      |
|  4 | juju GOU |  26 | M   | 2018-01-22      |
|  5 | li XIAO  |  23 | W   | 2018-01-22      |
|  6 | nv JI    |  18 | W   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
6 rows in set (0.00 sec)
```

## 5.11 MySQL DELETE语句

**语法**

```sql
DELETE FROM table_name [WHERE Clause]
```
* 如果没有指定 WHERE 子句，MySQL 表中的所有记录将被删除。
* 你可以在 WHERE 子句中指定任何条件
* 您可以在单个表中一次性删除记录。

**实例**

删除test_tbl数据表id=5的数据：

```sql
mysql> SELECT * FROM test_tbl;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  1 | pangzi W |  25 | M   | 2018-01-22      |
|  2 | san A    |  25 | M   | 2018-01-22      |
|  3 | sao A    |  24 | M   | 2018-01-22      |
|  4 | juju GOU |  26 | M   | 2018-01-22      |
|  5 | li XIAO  |  23 | W   | 2018-01-22      |
|  6 | nv JI    |  18 | W   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
6 rows in set (0.00 sec)

mysql> DELETE FROM test_tbl WHERE id=5;
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM test_tbl;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  1 | pangzi W |  25 | M   | 2018-01-22      |
|  2 | san A    |  25 | M   | 2018-01-22      |
|  3 | sao A    |  24 | M   | 2018-01-22      |
|  4 | juju GOU |  26 | M   | 2018-01-22      |
|  6 | nv JI    |  18 | W   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
5 rows in set (0.00 sec)
```
## 5.12 MySQL LIKE子句

WHERE 子句中可以使用等号 `=` 来设定获取数据的条件，如 "url = 'www.google.com'"。
但是有时候我们需要获取 url 字段含有 "com" 字符的所有记录，这时我们就需要在 WHERE 子句中使用 SQL LIKE 子句。

**语法**

```sql
SELECT field1, field2,...fieldN 
FROM table_name
WHERE field1 LIKE condition1 [AND [OR]] filed2 = 'somevalue'
```
* 你可以在 WHERE 子句中指定任何条件。
* 你可以在 WHERE 子句中使用LIKE子句。
* 你可以使用LIKE子句代替等号 =。
* LIKE 通常与 % 一同使用，类似于一个元字符的搜索。
* 你可以使用 AND 或者 OR 指定一个或多个条件。
* 你可以在 DELETE 或 UPDATE 命令中使用 WHERE...LIKE 子句来指定条件。

**实例**
查找url数据表中所有含.com的地址：
```sql
mysql> SELECT * FROM url;
+----+----------------+
| id | url_list       |
+----+----------------+
|  1 | www.baidu.com  |
|  2 | www.google.com |
|  3 | ecarry.cc      |
|  4 | weibo.com      |
|  5 | www.google.cn  |
+----+----------------+
5 rows in set (0.00 sec)

mysql> SELECT * FROM url WHERE url_list LIKE '%com';
+----+----------------+
| id | url_list       |
+----+----------------+
|  1 | www.baidu.com  |
|  2 | www.google.com |
|  4 | weibo.com      |
+----+----------------+
3 rows in set (0.00 sec)
```

## 5.13 MySQL UNION操作符

**描述**

MySQL UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。

**语法**

```sql
SELECT expression1, expression2, ... expression_n
FROM tables
[WHERE conditions]
UNION [ALL | DISTINCT]
SELECT expression1, expression2, ... expression_n
FROM tables
[WHERE conditions];
```

* **expression1, expression2, ... expression_n**: 要检索的列。
* **tables**: 要检索的数据表。
* **WHERE conditions**: 可选， 检索条件。
* **DISTINCT**: 可选，删除结果集中重复的数据。默认情况下 UNION 操作符已经删除了重复数据，所以 DISTINCT 修饰符对结果没啥影响。
* **ALL**: 可选，返回所有结果集，包含重复数据。

**实例**

下面是选自 "Websites" 表的数据：

```sql
mysql> SELECT * FROM Websites;
+----+--------------+---------------------------+-------+---------+
| id | name         | url                       | alexa | country |
+----+--------------+---------------------------+-------+---------+
| 1  | Google       | https://www.google.cm/    | 1     | USA     |
| 2  | 淘宝          | https://www.taobao.com/   | 13    | CN      |
| 3  | 菜鸟教程      | http://www.runoob.com/    | 4689  | CN      |
| 4  | 微博          | http://weibo.com/         | 20    | CN      |
| 5  | Facebook     | https://www.facebook.com/ | 3     | USA     |
| 7  | stackoverflow | http://stackoverflow.com/ |   0 | IND      |
+----+---------------+---------------------------+-------+---------+
```

下面是 "apps" APP 的数据：

```sql
mysql> SELECT * FROM apps;
+----+------------+-------------------------+---------+
| id | app_name   | url                     | country |
+----+------------+-------------------------+---------+
|  1 | QQ APP     | http://im.qq.com/       | CN      |
|  2 | 微博 APP   | http://weibo.com/        | CN      |
|  3 | 淘宝 APP   | https://www.taobao.com/  | CN      |
+----+------------+-------------------------+---------+
3 rows in set (0.00 sec)
```

### 5.13.1 SQL UNION 实例

下面的 SQL 语句从 "Websites" 和 "apps" 表中选取所有不同的country（只有不同的值）：

```sql
mysql> SELECT country FROM Websites
    -> UNION
    -> SELECT country FROM apps
    -> ORDER BY country;
+---------+
| country |
+---------+
| CN      |
| IND     |
| USA     |
+---------+  
3 rows in set (0.00 sec) 
```

**注释：**UNION 不能用于列出两个表中所有的country。如果一些网站和APP来自同一个国家，每个国家只会列出一次。UNION 只会选取不同的值。请使用 UNION ALL 来选取重复的值！

### 5.13.2 SQL UNION ALL 实例

下面的 SQL 语句使用 UNION ALL 从 "Websites" 和 "apps" 表中选取所有的country（也有重复的值）：

```sql
mysql> SELECT country FROM Websites
    -> UNION ALL
    -> SELECT country FROM apps
    -> ORDER BY country;
+---------+
| country |
+---------+
| CN      |
| CN      |
| CN      |
| CN      |
| CN      |
| IND     |
| USA     |
| USA     |
| USA     |
+---------+  
3 rows in set (0.00 sec) 
```

### 5.13.3 带有 WHERE 的 SQL UNION ALL

下面的 SQL 语句使用 UNION ALL 从 "Websites" 和 "apps" 表中选取所有的中国(CN)的数据（也有重复的值）：

```sql
mysql> SELECT country,name FROM Websites
    -> WHERE country='CN'
    -> UNION ALL
    -> SELECT country,app_name FROM apps
    -> WHERE country='CN'
    -> ORDER BY country;
+---------+--------------+
| country |    name      |
+---------+--------------+
| CN      |   淘宝        |
| CN      |   QQ APP     |
| CN      |   菜鸟        |
| CN      |   微博 APP    |
| CN      |   微博        |
| CN      |   淘宝 APP    |
+---------+--------------+  
6 rows in set (0.00 sec) 
```

## 5.14 MySQL排序

我们知道从 MySQL 表中使用 SQL SELECT 语句来读取数据。
如果我们需要对读取的数据进行排序，我们就可以使用 MySQL 的 ORDER BY 子句来设定你想按哪个字段哪种方式来进行排序，再返回搜索结果。

**语法**

以下是 SQL SELECT 语句使用 ORDER BY 子句将查询数据排序后再返回数据：

```sql
SELECT field1, field2,...fieldN table_name1, table_name2...
ORDER BY field1, [field2...] [ASC [DESC]]
```
* 你可以使用任何字段来作为排序的条件，从而返回排序后的查询结果。
* 你可以设定多个字段来排序。
* 你可以使用 ASC 或 DESC 关键字来设置查询结果是按升序或降序排列。 默认情况下，它是按升序排列。
* 你可以添加 WHERE...LIKE 子句来设置条件。

**实例**

演示**test_tbl**数据表

```sql
mysql> SELECT * FROM test_tbl;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  1 | pangzi W |  25 | M   | 2018-01-22      |
|  2 | san A    |  25 | M   | 2018-01-22      |
|  3 | sao A    |  24 | M   | 2018-01-22      |
|  4 | juju GOU |  26 | M   | 2018-01-22      |
|  6 | nv JI    |  18 | W   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
5 rows in set (0.00 sec)
```

将**test_tbl**通过**age**升序排列：

```sql
mysql> SELECT * FROM test_tbl ORDER BY age ASC;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  6 | nv JI    |  18 | W   | 2018-01-22      |
|  3 | sao A    |  24 | M   | 2018-01-22      |
|  1 | pangzi W |  25 | M   | 2018-01-22      |
|  2 | san A    |  25 | M   | 2018-01-22      |
|  4 | juju GOU |  26 | M   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
5 rows in set (0.01 sec)
```

将**test_tbl**通过**age**降序排列：

```sql
mysql> SELECT * FROM test_tbl ORDER BY age DESC;
+----+----------+-----+-----+-----------------+
| id | name     | age | sex | submission_date |
+----+----------+-----+-----+-----------------+
|  4 | juju GOU |  26 | M   | 2018-01-22      |
|  1 | pangzi W |  25 | M   | 2018-01-22      |
|  2 | san A    |  25 | M   | 2018-01-22      |
|  3 | sao A    |  24 | M   | 2018-01-22      |
|  6 | nv JI    |  18 | W   | 2018-01-22      |
+----+----------+-----+-----+-----------------+
5 rows in set (0.00 sec)
```

## 5.15 MySQL GROUP BY语句（分组）

GROUP BY 语句根据一个或多个列对结果集进行分组。在分组的列上我们可以使用 **COUNT, SUM, AVG**等函数。

**语法**

```sql
SELECT column_name, function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
```

**实例**

对employee_tbl数据表进行分组：

```sql
mysql> SELECT * FROM employee_tbl;
+----+--------+---------------------+--------+
| id | name   | date                | singin |
+----+--------+---------------------+--------+
|  1 | 小明 | 2016-04-22 15:25:33 |      1 |
|  2 | 小王 | 2016-04-20 15:25:47 |      3 |
|  3 | 小丽 | 2016-04-19 15:26:02 |      2 |
|  4 | 小王 | 2016-04-07 15:26:14 |      4 |
|  5 | 小明 | 2016-04-11 15:26:40 |      4 |
|  6 | 小明 | 2016-04-04 15:26:54 |      2 |
+----+--------+---------------------+--------+
6 rows in set (0.00 sec)
```

使用 GROUP BY 语句 将数据表按名字进行分组，并统计每个人有多少条记录：

```sql
mysql> SELECT name, COUNT(*) FROM   employee_tbl GROUP BY name;
+--------+----------+
| name   | COUNT(*) |
+--------+----------+
| 小丽 |        1 |
| 小明 |        3 |
| 小王 |        2 |
+--------+----------+
3 rows in set (0.01 sec)

```
### 5.15.1 使用 WITH ROLLUP

WITH ROLLUP 可以实现在分组统计数据基础上再进行相同的统计（SUM,AVG,COUNT…）。

例如我们将以上的数据表按名字进行分组，再统计每个人登录的次数：
```sql
mysql> SELECT name, SUM(singin) as singin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
+--------+--------------+
| name   | singin_count |
+--------+--------------+
| 小丽 |            2 |
| 小明 |            7 |
| 小王 |            7 |
| NULL   |           16 |
+--------+--------------+
4 rows in set (0.00 sec)
```

>其中记录 NULL 表示所有人的登录次数。

我们可以使用 coalesce 来设置一个可以取代 NUll 的名称，coalesce 语法：`select coalesce(a,b,c);`
参数说明：如果a==null,则选择b；如果b==null,则选择c；如果a!=null,则选择a；如果a b c 都为null ，则返回为null（没意义）。
以下实例中如果名字为空我们使用总数代替：

```sql
mysql> SELECT coalesce(name, '总数'), SUM(singin) as singin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
+--------------------------+--------------+
| coalesce(name, '总数') | singin_count |
+--------------------------+--------------+
| 小丽                   |            2 |
| 小明                   |            7 |
| 小王                   |            7 |
| 总数                   |           16 |
+--------------------------+--------------+
4 rows in set (0.01 sec)
```

## 5.16 MySQL连接的使用

在MySQL中，可以使用JOIN在两个或多个表中查询数据。
可以在 SELECT, UPDATE 和 DELETE 语句中使用 Mysql 的 JOIN 来联合多表查询。
JOIN 按照功能大致分为如下三类：
* **INNER JOIN（内连接,或等值连接）**：获取两个表中字段匹配关系的记录。
* **LEFT JOIN（左连接）**：获取左表所有记录，即使右表没有对应匹配的记录。
* **RIGHT JOIN（右连接）**： 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。


**实例：**使用菜鸟教程数据库

```sql
mysql> use RUNOOB;
Database changed
mysql> SELECT * FROM tcount_tbl;
+---------------+--------------+
| runoob_author | runoob_count |
+---------------+--------------+
| 菜鸟教程  | 10           |
| RUNOOB.COM    | 20           |
| Google        | 22           |
+---------------+--------------+
3 rows in set (0.01 sec)
 
mysql> SELECT * from runoob_tbl;
+-----------+---------------+---------------+-----------------+
| runoob_id | runoob_title  | runoob_author | submission_date |
+-----------+---------------+---------------+-----------------+
| 1         | 学习 PHP    | 菜鸟教程  | 2017-04-12              |
| 2         | 学习 MySQL  | 菜鸟教程  | 2017-04-12              |
| 3         | 学习 Java   | RUNOOB.COM    | 2015-05-01         |
| 4         | 学习 sql | RUNOOB.COM    | 2016-03-06         |
| 5         | 学习 C      | FK            | 2017-04-05         |
+-----------+---------------+---------------+-----------------+
5 rows in set (0.01 sec)
```

### 5.16.1 MySQL INNER JOIN

使用MySQL的INNER JOIN(也可以省略 INNER 使用 JOIN，效果一样)来连接以上两张表来读取runoob_tbl表中所有runoob_author字段在tcount_tbl表对应的runoob_count字段值：

```sql
mysql> SELECT a.runoob_id, a.runoob_author, b.runoob_count FROM runoob_tbl a INNER JOIN tcount_tbl b ON a.runoob_author = b.runoob_author;
+-------------+-----------------+----------------+
| a.runoob_id | a.runoob_author | b.runoob_count |
+-------------+-----------------+----------------+
| 1           | 菜鸟教程    | 10             |
| 2           | 菜鸟教程    | 10             |
| 3           | RUNOOB.COM      | 20             |
| 4           | RUNOOB.COM      | 20             |
+-------------+-----------------+----------------+
4 rows in set (0.00 sec)
```

等价于使用** WHERE子句**：

```sql
mysql> SELECT a.runoob_id, a.runoob_author, b.runoob_count FROM runoob_tbl a, tcount_tbl b WHERE a.runoob_author = b.runoob_author;
+-------------+-----------------+----------------+
| a.runoob_id | a.runoob_author | b.runoob_count |
+-------------+-----------------+----------------+
| 1           | 菜鸟教程    | 10             |
| 2           | 菜鸟教程    | 10             |
| 3           | RUNOOB.COM      | 20             |
| 4           | RUNOOB.COM      | 20             |
+-------------+-----------------+----------------+
4 rows in set (0.01 sec)
```

![](/img/mysql/img_innerjoin.gif)

### 5.16.2 MySQL LEFT JOIN

MySQL left join 与 join 有所不同。 MySQL LEFT JOIN 会读取左边数据表的全部数据，即便右边表无对应数据。
```sql
mysql> SELECT a.runoob_id, a.runoob_author, b.runoob_count FROM runoob_tbl a LEFT JOIN tcount_tbl b ON a.runoob_author = b.runoob_author;
+-------------+-----------------+----------------+
| a.runoob_id | a.runoob_author | b.runoob_count |
+-------------+-----------------+----------------+
| 1           | 菜鸟教程    | 10             |
| 2           | 菜鸟教程    | 10             |
| 3           | RUNOOB.COM      | 20             |
| 4           | RUNOOB.COM      | 20             |
| 5           | FK              | NULL           |
+-------------+-----------------+----------------+
5 rows in set (0.01 sec)
```

![](/img/mysql/img_leftjoin.gif)

### 5.16.3 MySQL RIGHT JOIN

MySQL RIGHT JOIN 会读取右边数据表的全部数据，即便左边边表无对应数据。

```sql
mysql> SELECT a.runoob_id, a.runoob_author, b.runoob_count FROM runoob_tbl a RIGHT JOIN tcount_tbl b ON a.runoob_author = b.runoob_author;
+-------------+-----------------+----------------+
| a.runoob_id | a.runoob_author | b.runoob_count |
+-------------+-----------------+----------------+
| 1           | 菜鸟教程    | 10             |
| 2           | 菜鸟教程    | 10             |
| 3           | RUNOOB.COM      | 20             |
| 4           | RUNOOB.COM      | 20             |
| NULL        | NULL            | 22             |
+-------------+-----------------+----------------+
5 rows in set (0.01 sec)
```
![](/img/mysql/img_rightjoin.gif)

## 5.17 MySQL NULL值处理

MySQL 使用 SQL SELECT 命令及 WHERE 子句来读取数据表中的数据,但是当提供的查询条件字段为 NULL 时，该命令可能就无法正常工作。
为了处理这种情况，MySQL提供了三大运算符:

* **IS NULL**: 当列的值是 NULL,此运算符返回 true。
* **IS NOT NULL**: 当列的值不为 NULL, 运算符返回 true。
* **<=>**: 比较操作符（不同于=运算符），当比较的的两个值为 NULL 时返回 true。

关于 NULL 的条件比较运算是比较特殊的。你不能使用 = NULL 或 != NULL 在列中查找 NULL 值 。
在 MySQL 中，NULL 值与任何其它值的比较（即使是 NULL）永远返回 false，即 NULL = NULL 返回false 。
MySQL 中处理 NULL 使用 IS NULL 和 IS NOT NULL 运算符。

**实例**

在数据库test中创建数据表test_NULL含有两列name，age，在age中插入NULL值。

```sql
mysql> CREATE TABLE test_NULL( name CHAR(10) NOT NULL, age INT);
Query OK, 0 rows affected (0.02 sec)

mysql> desc test_NULL;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| name  | char(10) | NO   |     | NULL    |       |
| age   | int(11)  | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
2 rows in set (0.00 sec)

mysql> INSERT INTO test_NULL( name,age) VALUES ('LILI',20);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO test_NULL( name,age) VALUES ('LALA',NULL);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO test_NULL( name,age) VALUES ('FEIFEI',NULL);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO test_NULL( name,age) VALUES ('JIJI',NULL);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO test_NULL( name,age) VALUES ('GEGE',25);
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM test_NULL;
+--------+------+
| name   | age  |
+--------+------+
| LILI   |   20 |
| LALA   | NULL |
| FEIFEI | NULL |
| JIJI   | NULL |
| GEGE   |   25 |
+--------+------+
5 rows in set (0.00 sec)
```

使用SELECT,WHERE查询NULL值用**=**和**！=**运算符是不起作用的：

```sql
mysql> SELECT * FROM test_NULL WHERE age=NULL;
Empty set (0.00 sec)

mysql> SELECT * FROM test_NULL WHERE age!=NULL;
Empty set (0.00 sec)
```

要查询表中age列为是否为NULL值的，必须使用**IS NULL**和**IS NOT NULL** ：

```sql
mysql> SELECT * FROM test_NULL WHERE age is NULL;
+--------+------+
| name   | age  |
+--------+------+
| LALA   | NULL |
| FEIFEI | NULL |
| JIJI   | NULL |
+--------+------+
3 rows in set (0.00 sec)

mysql> SELECT * FROM test_NULL WHERE age is NOT NULL;
+------+------+
| name | age  |
+------+------+
| LILI |   20 |
| GEGE |   25 |
+------+------+
2 rows in set (0.00 sec)
```

## 5.18 MySQL正则表达式

下表中的正则模式可应用于 REGEXP 操作符中。

| 模式        | 描述                                                                                         |
|:--------:  |:-----:                                                                                       |
|    ^       |匹配输入字符串的开始位置。如果设置了 RegExp 对象的 Multiline 属性，<br> ^ 也匹配 '\n' 或 '\r' 之后的位置。|
|    $       |匹配输入字符串的结束位置。如果设置了RegExp 对象的 Multiline 属性，<br> $也匹配 '\n' 或 '\r' 之前的位置。|
|    .       |匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符，<br>请使用象 '[.\n]' 的模式。            |
|   [...]    |字符集合。匹配所包含的任意一个字符。例如， '[abc]' 可以匹配 "plain" 中的 'a'。                         |
|   [^...]   |负值字符集合。匹配未包含的任意字符。例如， '[^abc]' 可以匹配 "plain" 中的'p'。                         |
|  p1/p2/p3	| 匹配 p1 或 p2 或 p3。例如，'z/food' 能匹配 "z" 或 "food"。<br>'(z/f)ood' 则匹配 "zood" 或 "food"。|
|    *       |匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。                            |
|+           |匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，<br>但不能匹配 "z"。+ 等价于 {1,}。     |
|{n}         |n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，<br>但是能匹配 "food" 中的两个 o。|
|{n,m}       |m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。                                        |

**实例**

在数据表name中查找名字时：
查找name字段中以‘st’为开头的所有数据：
```sql
mysql> SELECT name FROM person_tbl WHERE name REGEXP '^st';
```
查找name字段中以‘ok’结尾的所有数据：
```sql
mysql> SELECT name FROM person_tbl WHERE name REGEXP 'ok$';
```
查找name字段中包含‘mar’字符串的所有数据：
```sql
mysql> SELECT name FROM person_tbl WHERE name REGEXP 'mar';
```
查找name字段中以元音字符开头或以‘ok’字符串结尾的所有数据：
```sql
mysql> SELECT name FROM person_tbl WHERE name REGEXP '^[aeiou]|ok$';
```

## 5.19 MySQL ALTER命令

当我们需要修改数据表名或者修改数据表字段时，就需要使用到MySQL ALTER命令。
先创建一张表testalter_tbl：

```sql
mysql> CREATE TABLE testalter_tbl(i INT, C CHAR(10));
Query OK, 0 rows affected (0.02 sec)

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| i     | int(11)  | YES  |     | NULL    |       |
| C     | char(10) | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```
### 5.19.1 删除，添加或修改字段

1. 使用ALTER 命令及 DROP 子句来删除以上创建表的 i 字段：

```sql
mysql> ALTER TABLE testalter_tbl DROP i;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| C     | char(10) | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
1 row in set (0.00 sec)
```
**如果数据表中只剩余一个字段则无法使用DROP来删除字段。**

2. 使用 ADD 子句来向表 testalter_tbl 中添加 i 字段，并定义数据类型:

```sql
mysql> ALTER TABLE testalter_tbl ADD i INT;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| C     | char(10) | YES  |     | NULL    |       |
| i     | int(11)  | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```
执行以上命令后，i 字段会自动添加到数据表字段的末尾。

3. 如果你需要指定新增字段的位置，可以使用MySQL提供的关键字 FIRST (设定位第一列)， AFTER 字段名（设定位于某个字段之后）:

```sql
mysql> ALTER TABLE testalter_tbl ADD a INT FIRST;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| a     | int(11)  | YES  |     | NULL    |       |
| C     | char(10) | YES  |     | NULL    |       |
| i     | int(11)  | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.00 sec)

mysql> ALTER TABLE testalter_tbl ADD b INT AFTER a;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| a     | int(11)  | YES  |     | NULL    |       |
| b     | int(11)  | YES  |     | NULL    |       |
| C     | char(10) | YES  |     | NULL    |       |
| i     | int(11)  | YES  |     | NULL    |       |
+-------+----------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```

### 5.19.2 修改字段类型及名称

如果需要修改字段类型及名称, 你可以在ALTER命令中使用 MODIFY 或 CHANGE 子句 。

**实例：**把字段 c 的类型从 CHAR(10) 改为 CHAR(1)，可以执行以下命令:

```sql
mysql> ALTER TABLE testalter_tbl MODIFY c CHAR(1);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| a     | int(11) | YES  |     | NULL    |       |
| b     | int(11) | YES  |     | NULL    |       |
| c     | char(1) | YES  |     | NULL    |       |
| i     | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```
使用 CHANGE 子句, 语法有很大的不同。 在 CHANGE 关键字之后，紧跟着的是你要修改的字段名，然后指定新字段名及类型。
**实例：**将字段i改为j并将数据类型改为BIGINT，再将BIGINT改为INT。

```sql
mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| a     | int(11) | YES  |     | NULL    |       |
| b     | int(11) | YES  |     | NULL    |       |
| c     | char(1) | YES  |     | NULL    |       |
| i     | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
4 rows in set (0.00 sec)

mysql> ALTER TABLE testalter_tbl CHANGE i j BIGINT;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| a     | int(11)    | YES  |     | NULL    |       |
| b     | int(11)    | YES  |     | NULL    |       |
| c     | char(1)    | YES  |     | NULL    |       |
| j     | bigint(20) | YES  |     | NULL    |       |
+-------+------------+------+-----+---------+-------+
4 rows in set (0.01 sec)

mysql> ALTER TABLE testalter_tbl CHANGE j j INT;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| a     | int(11) | YES  |     | NULL    |       |
| b     | int(11) | YES  |     | NULL    |       |
| c     | char(1) | YES  |     | NULL    |       |
| j     | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```
### 5.19.3 ALTER TABLE 对 Null 值和默认值的影响

当你修改字段时，你可以指定是否包含值或者是否设置默认值。
**实例：**指定字段 j 为 NOT NULL 且默认值为100 。

```sql
mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| a     | int(11) | YES  |     | NULL    |       |
| b     | int(11) | YES  |     | NULL    |       |
| c     | char(1) | YES  |     | NULL    |       |
| j     | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
4 rows in set (0.00 sec)

mysql> ALTER TABLE testalter_tbl
    -> MODIFY j BIGINT NOT NULL DEFAULT 100;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| a     | int(11)    | YES  |     | NULL    |       |
| b     | int(11)    | YES  |     | NULL    |       |
| c     | char(1)    | YES  |     | NULL    |       |
| j     | bigint(20) | NO   |     | 100     |       |
+-------+------------+------+-----+---------+-------+
4 rows in set (0.01 sec)
```
如果不设置默认值，MySQL会自动设置该字段默认为 NULL。

### 5.19.4 修改字段默认值

以使用 ALTER 来修改字段的默认值。
**实例：**
```sql
mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| a     | int(11)    | YES  |     | NULL    |       |
| b     | int(11)    | YES  |     | NULL    |       |
| c     | char(1)    | YES  |     | NULL    |       |
| j     | bigint(20) | NO   |     | 100     |       |
+-------+------------+------+-----+---------+-------+
4 rows in set (0.01 sec)

mysql> ALTER TABLE testalter_tbl ALTER a SET DEFAULT 1000;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| a     | int(11)    | YES  |     | 1000    |       |
| b     | int(11)    | YES  |     | NULL    |       |
| c     | char(1)    | YES  |     | NULL    |       |
| j     | bigint(20) | NO   |     | 100     |       |
+-------+------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```
使用 ALTER 命令及 DROP子句来删除字段的默认值。
**实例：**
```sql
mysql> ALTER TABLE testalter_tbl ALTER j DROP DEFAULT;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW COLUMNS FROM testalter_tbl;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| a     | int(11)    | YES  |     | 1000    |       |
| b     | int(11)    | YES  |     | NULL    |       |
| c     | char(1)    | YES  |     | NULL    |       |
| j     | bigint(20) | NO   |     | NULL    |       |
+-------+------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```

### 5.19.5 修改数据表类型

使用 ALTER 命令及 TYPE 子句修改数据表类型，将数据表testalter_tbl的类型改为MYSAM：
**注意：**查看数据表类型可以使用 SHOW TABLE STATUS 语句。
```sql
mysql> SHOW TABLE STATUS LIKE 'testalter_tbl'\G;
*************************** 1. row ***************************
           Name: testalter_tbl
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 0
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2018-01-24 11:06:35
    Update_time: NULL
     Check_time: NULL
      Collation: latin1_swedish_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.01 sec)

mysql> ALTER TABLE testalter_tbl ENGINE=MYISAM;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW TABLE STATUS LIKE 'testalter_tbl'\G;
*************************** 1. row ***************************
           Name: testalter_tbl
         Engine: MyISAM
        Version: 10
     Row_format: Fixed
           Rows: 0
 Avg_row_length: 0
    Data_length: 0
Max_data_length: 5066549580791807
   Index_length: 1024
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2018-01-24 11:17:33
    Update_time: 2018-01-24 11:17:33
     Check_time: NULL
      Collation: latin1_swedish_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.00 sec)
```

### 5.19.6 修改表名

如果需要修改数据表的名称，可以在 ALTER TABLE 语句中使用 RENAME 子句来实现。
**实例：**修改表test_NULL为testnull_tbl:

```sql
mysql> ALTER TABLE test_NULL RENAME TO testnull_tbl;
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| bili           |
| test_tbl       |
| testalter_tbl  |
| testnull_tbl   |
| url            |
+----------------+
5 rows in set (0.00 sec)
```

