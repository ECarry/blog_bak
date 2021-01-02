---
layout: post
title: Python操作MySQL数据库
tags: [Python, MySQL]
index_img: https://w.wallhaven.cc/full/5w/wallhaven-5wwzq5.png
banner_img: https://w.wallhaven.cc/full/5w/wallhaven-5wwzq5.png
categories: [Python]
date: 2018-01-25
---


# Python3连接MySQL数据库

使用Python3的**PyMySQL**库连接MySQL数据库，实现简单的增删改查。



## PyMySQL

PyMySQL是在Python3中用于连接MySQL服务器的一个库。（Python2使用的是mysqldb。）

## 数据库连接

**实例：**连接test数据库

```javascript
#encoding=utf-8

import pymysql


# 打开数据库连接"localhost":本地服务器,"root":用户名,"666666":密码,"test":数据库名
db = pymysql.connect("localhost","root","666666","test")


# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()


# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT VERSION()")



# 使用 fetchone() 方法获取单条数据.
data = cursor.fetchone()


print("Database version: %s " %data)

# 关闭数据库
db.close()
```
输出的结果为：`Database version: 5.6.39 `

## 创建数据库表

**实例：**用**execute()**方法来为数据库创建表

```javascript
# encoding=utf-8

import pymysql


# 打开数据库连接"localhost":本地服务器,"root":用户名,"666666":密码,"test":数据库名
db = pymysql.connect("localhost","root","666666","test")

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute() 方法执行 SQL，如果表存在则删除
cursor.execute("DROP TABLE IF EXISTS testPyMySQL_tbl")

sql = """CREATE TABLE testPyMySQL_tbl(
         FIRST_NAME  CHAR(20) NOT NULL,
         LAST_NAME  CHAR(20),
         AGE INT,  
         SEX CHAR(1),
         INCOME FLOAT)"""

cursor.execute(sql)

# 关闭数据库连接
db.close()

```

## 插入数据

**实例：**向数据表testPyMySQL_tbl插入数据：
```javascript
#!/usr/bin/python3
#encoding=utf-8

#使用执行 SQL INSERT 语句向表 testPyMySQL_tbl 插入记录：


import pymysql

# 打开数据库连接
db = pymysql.connect("localhost","root","666666","test" )

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 插入语句
sql = """INSERT INTO testPyMySQL_tbl(FIRST_NAME,
         LAST_NAME, AGE, SEX, INCOME)
         VALUES ('SAO', 'A', 25, 'M', 10000)"""
try:
   # 执行sql语句
   cursor.execute(sql)
   # 提交到数据库执行
   db.commit()
except:
   # 如果发生错误则回滚
   db.rollback()

# 关闭数据库连接
db.close()
```

## 数据库查询
Python查询Mysql使用 fetchone() 方法获取单条数据, 使用fetchall() 方法获取多条数据。
* **fetchone()**: 该方法获取下一个查询结果集。结果集是一个对象
* **fetchall()**: 接收全部的返回结果行.
* **rowcount**: 这是一个只读属性，并返回执行execute()方法后影响的行数。

**实例：**查询testPyMySQL_tbl表中工资大于1000的所有数据：
```javascript
mysql> SELECT * FROM testPyMySQL_tbl;
+------------+-----------+------+------+--------+
| FIRST_NAME | LAST_NAME | AGE  | SEX  | INCOME |
+------------+-----------+------+------+--------+
| SAO        | A         |   25 | M    |  10000 |
| SAO        | B         |   25 | M    |  10000 |
| SAO        | C         |   25 | M    |  10000 |
| SAO        | D         |   25 | M    |  10000 |
| SAO        | F         |   25 | M    |    500 |
| SAO        | G         |   25 | M    |   1000 |
| SAO        | H         |   25 | M    |   1100 |
+------------+-----------+------+------+--------+
7 rows in set (0.00 sec)

# encoding=utf-8

import pymysql

# 打开数据库连接
db = pymysql.connect("localhost","root","666666","test" )

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 查询语句
sql = "SELECT * FROM testPyMySQL_tbl \
       WHERE INCOME > '%d'" % (1000)
try:
   # 执行SQL语句
   cursor.execute(sql)
   # 获取所有记录列表
   results = cursor.fetchall()
   for row in results:
      fname = row[0]
      lname = row[1]
      age = row[2]
      sex = row[3]
      income = row[4]
       # 打印结果
      print ("fname=%s,lname=%s,age=%d,sex=%s,income=%d" % \
             (fname, lname, age, sex, income ))
except:
   print ("Error: unable to fetch data")

# 关闭数据库连接
db.close()
```
结果：
fname=SAO,lname=A,age=25,sex=M,income=10000
fname=SAO,lname=B,age=25,sex=M,income=10000
fname=SAO,lname=C,age=25,sex=M,income=10000
fname=SAO,lname=D,age=25,sex=M,income=10000
fname=SAO,lname=H,age=25,sex=M,income=1100

## 数据库更新操作

**实例：**#更新操作用于更新数据表的的数据，以下实例将 testPyMySQL_tbl表中的 SEX 为 'M'的AGE 字段递增1：

```javascript
#encoding=utf-8
import pymysql

# 打开数据库连接
db = pymysql.connect("localhost","root","666666","test" )

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 更新语句
sql = "UPDATE testPyMySQL_tbl SET AGE = AGE + 1 WHERE SEX = '%c'" % ('M')
try:
   # 执行SQL语句
   cursor.execute(sql)
   # 提交到数据库执行
   db.commit()
except:
   # 发生错误时回滚
   db.rollback()

# 关闭数据库连接
db.close()

```

## 删除操作

删除表中的数值
**实例：**删除表中AGE小于18的所有数据：

```javascript
mysql> SELECT * FROM testPyMySQL_tbl;
+------------+-----------+------+------+--------+
| FIRST_NAME | LAST_NAME | AGE  | SEX  | INCOME |
+------------+-----------+------+------+--------+
| SAO        | A         |   16 | M    |  10000 |
| SAO        | B         |   27 | M    |  10000 |
| SAO        | C         |   18 | M    |  10000 |
| SAO        | D         |   15 | M    |  10000 |
| SAO        | F         |   27 | M    |    500 |
| SAO        | G         |   27 | M    |   1000 |
| SAO        | H         |   17 | M    |   1100 |
+------------+-----------+------+------+--------+
7 rows in set (0.00 sec)

# encoding=utf-8
import pymysql


# 打开数据库连接
db = pymysql.connect("localhost","root","666666","test" )

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 删除语句
sql = "DELETE FROM testPyMySQL_tbl WHERE AGE < '%d'" % (18)
try:
   # 执行SQL语句
   cursor.execute(sql)
   # 提交修改
   db.commit()
except:
   # 发生错误时回滚
   db.rollback()

# 关闭连接
db.close()

mysql> SELECT * FROM testPyMySQL_tbl;
+------------+-----------+------+------+--------+
| FIRST_NAME | LAST_NAME | AGE  | SEX  | INCOME |
+------------+-----------+------+------+--------+
| SAO        | B         |   27 | M    |  10000 |
| SAO        | C         |   18 | M    |  10000 |
| SAO        | F         |   27 | M    |    500 |
| SAO        | G         |   27 | M    |   1000 |
+------------+-----------+------+------+--------+
4 rows in set (0.00 sec)
```