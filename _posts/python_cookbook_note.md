---
layout: post
title: Python Cookbook Note
tags: [Python]
index_img: https://th.wallhaven.cc/small/43/43x126.jpg
banner_img: https://w.wallhaven.cc/full/43/wallhaven-43x126.png
categories: [Python]
date: 2018-02-26
---

# 1、数据结构和算法

## 1.1 将不用的变量使用任意变量名去占位，然后丢弃，取其想要的部分

```python
data = ['ACME',50,91.1,(2018, 2, 26)]
_,shares,price,_ = data
print(shares)
>>>50
```

## 1.2 星号解压语法在字符串操作

```python
line = 'nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false'
uname,*fields,homedir,sh = line.split(':')                         
print(uname)                                                       
print(homedir)    
>>>nobody
>>>/var/empty                                                 
```

## 1.3 当要解压一写元素后丢弃它们，不能使用*的时候，可以使用普通废弃名称，比如_或ign(ignore)

```python
c = [(2,16,2018),'星期一','晴']   
(_,_,d),e,f = c               
print(d)                      
>>> 2018                                                              
```
## 1.4 使用heapq模块来查找最大或最小的N个元素列表  

```python
import heapq                         
nums = [1,4,5,2,7,3,21,-2,63,-14]   
#打印出最大或最小的3个元素 
print(heapq.nlargest(3,nums))        
print(heapq.nsmallest(3,nums))
>>>[63, 21, 7]
>>>[-14, -2, 1]                                                                    
```
