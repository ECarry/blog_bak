---
layout: post
title: 怎么写出一手漂亮的代码-转载（菜鸟学Python）
tags: [Python]
index_img: https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1091978681,1253128401&fm=26&gp=0.jpg
banner_img: https://w.wallhaven.cc/full/w8/wallhaven-w8531r.png
categories: [Python]
date: 2018-02-08 15:45:41
---

>菜鸟学Python的第138篇原创文章

# 1.打印index

对于一个列表，或者说一个序列我们经常需要打印它的index,一般传统的做法或者说比较low的写法：

```python
# Bad way
cities = ['Shanghai','Beijing','Shenzhen','Chengdu']
i = 0
for c in cities:
    print(i + 1, '-->',c)
    i += 1
>>>
1 --> Shanghai
2 --> Beijing
3 --> Shenzhen
4 --> Chengdu
```

优雅写法是多用**enumerate**

```python
# Good way
for i ,city in enumerate(cities):
    print(i + 1,'-->',city)
>>>
1 --> Shanghai
2 --> Beijing
3 --> Shenzhen
4 --> Chengdu
```

## Python内置enumerate()函数

### 描述

enumerate() 函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。

### 语法

`enumerate(sequence, [start=0])`
* sequence -- 一个序列、迭代器或其他支持迭代对象。
* start -- 下标起始位置。

### 返回值

返回 enumerate(枚举) 对象。

# 2.两个序列的循环

我们会经常对两个序列进行计算或者处理，比较low的方法是用下标去循环处理：

```python
names = ['leo','jack','john','james']
colors = ['red','green','blue','yellow']

# Bad way
n = min(len(names),len(colors))
for i in range(n):
    print(names[i],'-->',colors[i])
>>>
leo --> red
jack --> green
john --> blue
james --> yellow
```

更优雅一点的方法:用**zip**轻松搞定:

```python
names = ['leo','jack','john','james']
colors = ['red','green','blue','yellow']

# Good way
for name ,color in zip(names,colors):
    print(name,'-->',color)
>>>
leo --> red
jack --> green
john --> blue
james --> yellow
```

# 3.变换变量

多个变量之间的交换，相信很多有c,c++语言基础的同学对这个再熟悉不过了，比如我们经典的冒泡排序，就会用这一招，看看比较传统的做法先:

```python
x = 1
y = 2

# Bad way
print('>>Before:x={},y={}'.format(x,y))
tmp = y
y = x
x = tmp
print('>>After:x={},y={}'.format(x,y))
>>>
>>Before:x=1,y=2
>>After:x=2,y=1
```

更优雅的做法是：

```python
x = 1
y = 2

# Good way
print('>>Before:x={},y={}'.format(x,y))
x,y = y,x
print('>>After:x={},y={}'.format(x,y))
>>>
>>Before:x=1,y=2
>>After:x=2,y=1
```

# 4.字典的读取

字典是我们经常使用的数据结构，对于字典的访问和读取，如果我们的读取的字典的key为空怎么办，一般我们需要一个缺省值,菜鸟的写法:

```python
students={
    'LILI':18,
    'Sam':25
}
#Bad way
if 'Susan' in students:
    student = students['Susan']
else:
    student='unknow'

print('Susan is {} years old'.format(student))
>>>Susan is unknow years old
```

比较优雅的做法是:

```python
students={
    'LILI':18,
    'Sam':25
}
# Good way
student = students.get('Susan','unknow')
print('Susan is {} years old'.format(student))
```

巧妙的利用了字典get的用法，如果字典里面没有Susan这个key,则用unknow来表示缺省值！

# 5.文件读取查找

通常来说，我们要打开一个文件，然后对文件的内容进行循环读取和处理，菜鸟的写法如下：

```python
#Bad way
f = open('data.txt')
try:
    text = f.read()
    for line in text.split('\n'):
        print(line)
finally:
    f.close()
```

更优雅的写法：

```python
#Good way
with open('data.txt') as f:
    for line in f:
        print(line.strip('\n'))
```

# 6.循环查找

我们经常会在一个大的循环中作搜索业务，比如从一个文件中搜索关键字，比如从文件名列表中查找一些特殊的文件名，想当然的写法如下：

```python
target_letter = 'd'
letters = ['a','b','c']
#Bad way
found = False
for letter in letters:
    if letter == target_letter:
        print('Found')
        found = True
        break
if not found:
    print('Not found')
>>>
Not found
```

更优雅的写法：

```python
target_letter = 'd'
letters = ['a','b','c']
#Good way
for letter in letters:
    if letter == target_letter:
        print('Found')
        break
else:
    print('Not found')
>>>
Not found
```