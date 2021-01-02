---
layout: post
title: Linux 运维基本功学习路径
tags: [Linux]
index_img: https://th.wallhaven.cc/small/45/45j2k1.jpg
banner_img: https://w.wallhaven.cc/full/45/wallhaven-45j2k1.jpg
categories: [Linux]
date: 2020-09-24 09:43:03
---

# 一、 安装、配置、基础命令

## 1.1 镜像文件下载

### 1.1.1 CentOS 镜像

#### 简介

CentOS，是基于 Red Hat Linux 提供的可自由使用源代码的企业级 Linux 发行版本；是一个稳定，可预测，可管理和可复制的免费企业级计算平台。

#### 相关链接

下载地址：
- [阿里源](https://mirrors.aliyun.com/centos/)
- [163源](http://mirrors.163.com/centos/)

### 1.1.2 Ubuntu 镜像

#### 简介

Ubuntu，是一款基于 Debian Linux 的以桌面应用为主的操作系统，内容涵盖文字处理、电子邮件、软件开发工具和 Web 服务等，可供用户免费下载、使用和分享。

#### 相关链接

- [阿里源](https://mirrors.aliyun.com/ubuntu/)
- [163源](http://mirrors.163.com/ubuntu-releases/)

## 1.2 安装

### 1.2.1 分区

关于分区详细：[关于 Linux 分区的小事](https://ecarry.cc/2020/06/09/Linux_part/#)

### 1.2.2 强制使用 GPT 来分区

>如果硬盘容量小于 2TB 的话，系统预设会使用 MBR 模式来安装。

1. 在安装界面，如图所示，按下 `Tab` 键，在后面输入 `inst.gpt`：

![](/img/linux_base/linux_base_1.jpg)

2. 在分区界面需要添加 `biosboot` 分区，此分区供 **bios** 使用，必须创建，而使用 MBR 分区就无需此分区。

![](/img/linux_base/linux_base_2.jpg)

参考资料: 

wiki: [BIOS boot partition](https://en.wikipedia.org/wiki/BIOS_boot_partition)

3. 其余分区按需分配

### 1.2.3 测试安装设备的稳定性

CentOS 安装镜像自带烧机功能，如图：

![](/img/linux_base/linux_base_3.jpg)

测试界面：

![](/img/linux_base/linux_base_4.jpg)

## 1.3 基础命令

### 1.3.1 locale 

运行时地区、语言、字符集和时区

```bash
[root@localhost ~]# locale
LANG=en_US.UTF-8                    <--- 语言，只与输出信息有关
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"               <--- 时区
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=                             <--- 修改所有
```

修改输出信息为中文：

```bash
[root@localhost ~]# LANG=zh_CN.UTF-8
```

### 1.3.2 date 

时间命令

```bash
[root@localhost ~]# date
Thu Sep 24 21:50:28 CST 2020
```

格式化时间：

```bash
[root@localhost ~]# date '+%Y-%m-%d %H:%M:%S'
2020-09-24 21:52:49
```

### 1.3.3 cal 

日历

```bash
[root@localhost ~]# cal
#显示当月月份
   September 2020   
Su Mo Tu We Th Fr Sa
       1  2  3  4  5
 6  7  8  9 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30

[root@localhost ~]# cal 2020
#显示2020年月份
                               2020                               

       January               February                 March       
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
          1  2  3  4                      1    1  2  3  4  5  6  7
 5  6  7  8  9 10 11    2  3  4  5  6  7  8    8  9 10 11 12 13 14
12 13 14 15 16 17 18    9 10 11 12 13 14 15   15 16 17 18 19 20 21
19 20 21 22 23 24 25   16 17 18 19 20 21 22   22 23 24 25 26 27 28
26 27 28 29 30 31      23 24 25 26 27 28 29   29 30 31

        April                   May                   June        
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
          1  2  3  4                   1  2       1  2  3  4  5  6
 5  6  7  8  9 10 11    3  4  5  6  7  8  9    7  8  9 10 11 12 13
12 13 14 15 16 17 18   10 11 12 13 14 15 16   14 15 16 17 18 19 20
19 20 21 22 23 24 25   17 18 19 20 21 22 23   21 22 23 24 25 26 27
26 27 28 29 30         24 25 26 27 28 29 30   28 29 30
                       31
        July                  August                September     
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
          1  2  3  4                      1          1  2  3  4  5
 5  6  7  8  9 10 11    2  3  4  5  6  7  8    6  7  8  9 10 11 12
12 13 14 15 16 17 18    9 10 11 12 13 14 15   13 14 15 16 17 18 19
19 20 21 22 23 24 25   16 17 18 19 20 21 22   20 21 22 23 24 25 26
26 27 28 29 30 31      23 24 25 26 27 28 29   27 28 29 30
                       30 31
       October               November               December      
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
             1  2  3    1  2  3  4  5  6  7          1  2  3  4  5
 4  5  6  7  8  9 10    8  9 10 11 12 13 14    6  7  8  9 10 11 12
11 12 13 14 15 16 17   15 16 17 18 19 20 21   13 14 15 16 17 18 19
18 19 20 21 22 23 24   22 23 24 25 26 27 28   20 21 22 23 24 25 26
25 26 27 28 29 30 31   29 30                  27 28 29 30 31


[root@localhost ~]# cal 10 2020
#显示2020年10月
    October 2020    
Su Mo Tu We Th Fr Sa
             1  2  3
 4  5  6  7  8  9 10
11 12 13 14 15 16 17
18 19 20 21 22 23 24
25 26 27 28 29 30 31

```

### 1.3.4 man 

**查看某个命令介绍及使用方法**，用法：

```bash
[root@localhost ~]# man date
DATE(1)                                                             User Commands                                                             DATE(1)

NAME
       date - print or set the system date and time

SYNOPSIS
       date [OPTION]... [+FORMAT]
       date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]

DESCRIPTION
       Display the current time in the given FORMAT, or set the system date.

       Mandatory arguments to long options are mandatory for short options too.
```

### 1.3.5 sync

写盘命令，将还在内存中的数据写入磁盘。


# 二、 文件及目录管理

## 2.1 文件/目录命名规则

### 2.1.1 文件/目录命名长度限制

在Linux底下，使用传统的 **Ext2/Ext3/Ext4** 文件系统以及近来被 CentOS 7 当作预设文件系统的 **xfs** 而言，针对文件名长度限制为：

- 单一文件或目录的最大容许名为 **255bytes**，以一个 ASCII 英文占用一个 bytes 来说，则大约可达 **255** 个字元长度。若是以每个中文字占用 2bytes 来说， 最大档名就是大约在 **128** 个中文字之谱！

### 2.1.2 文件/目录 命名规则

避免特殊符号：

- \* ? > < ; & ! [ ] | \ ' " ` ( ) { }
- 避免使用 + - 或 . 作为普通文件名的第一个字符(在Linux下以.开头的文件是属于隐藏文件)
 
不能使用的符号：

- /

在 linux 下，是区分大小写的，如 **test.txt** 和 **TEST.txt** 是完全两个文件。

## 2.2 目录与路径

#### 鸟哥的目录结构图
![鸟哥的目录结构图](/img/linux_base/linux_base_5.jpg)

### 2.2.1 绝对路径

路径的写法『一定由根目录 / 写起』，例如： /usr/share/doc 这个目录。

### 2.2.2 相对路径

路径的写法『不是由 / 写起』，例如由 /usr/share/doc 要到 /usr/share/man 底下时，可以写成： 『cd ../man』这就是相对路径的写法啦！相对路径意指『相对于目前工作目录的路径！』
 
## 2.3 目录管理

* . :表示当前目录
* ..：上级目录
* -：前一个工作区
* ~：当前用户的home目录
* ls：列出目录下的文件
* cd：切换目录
* pwd：打印当前目录
* mkdir：创建空目录
    -p：依次创建层级目录，如 mkdir -p /1/2/3
    -v：显示详情
* rmdir：删除空目录

```bash
#例1：在目录下创建多个目录，如在空目录 /mnt/ 下创建 /mnt/test/x/m,/mnt/test/y
[root@localhost ~]# mkdir -pv /mnt/test/{x/m,y}
mkdir: 已创建目录 "/mnt/test"
mkdir: 已创建目录 "/mnt/test/x"
mkdir: 已创建目录 "/mnt/test/x/m"
mkdir: 已创建目录 "/mnt/test/y"
#例2：在目录下创建 a_b,a_c,d_b,d_C 文件
[root@localhost ~]# mkdir -pv /mnt/test2/{a,d}_{b,c}
mkdir: 已创建目录 "/mnt/test2"
mkdir: 已创建目录 "/mnt/test2/a_b"
mkdir: 已创建目录 "/mnt/test2/a_c"
mkdir: 已创建目录 "/mnt/test2/d_b"
mkdir: 已创建目录 "/mnt/test2/d_c"
```



## 2.4 linux 下目录详解

rootfs：根文件系统

```bash
/boot：系统启动相关文件，如内核、initrd 以及 grub(bootloader)
/dev：设备文件
        块设备：随机访问，将数据分为块，称为数据块，如硬盘
        字符设备：线性访问，按字符为单位
        设备号：主设备号（major）和次设备号（minor）
/etc：配置文件
/lib64：库文件
    静态库
    动态库：.so(shared object)
    /modules：内核模块文件
/opt：可选目录，第三方软件存放处
/proc：伪文件系统，内核映射文件
/sys：伪文件系统，跟硬件设备相关的属性映射文件
/tmp：临时文件
/var：与系统运作过程有关，可变化文件，如存放日志文件
/bin：可执行文件，用户命令
/sbin：管理命令
/usr：Unix Software Resource(shared,read-only),操作系统软件资源所放置的目录,所有系统默认的软件都会放置到/usr
```

## 2.5 文件管理

### 2.5.1 新建文件

#### touch

* touch：change file timestamps
    -a：修改访问时间
    -m：修改更改时间
    -t：修改到指定时间
* touch:  filename 创建名为 filename 的文件

#### vi

* vi : filename 创建文件或者修改文件

### 2.5.2 stat

* stat：显示文件或文件系统状态

```bash
[root@localhost ~]# stat test
  File: ‘test’
  Size: 4             Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d  Inode: 34051578    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2020-08-18 23:29:02.280000000 +0800
Modify: 2020-08-18 23:26:58.260000000 +0800
Change: 2020-08-18 23:29:02.280000000 +0800
 Birth: -
```

### 2.5.3 cp 复制文件

### 2.5.4 mv 移动或者重命名

### 2.5.5 显示文件

#### cat

```bash
[root@localhost ~]# cat
选项与参数：
-A  ：相当于 -vET 的整合选项，可列出一些特殊字符而不是空白而已；
-b  ：列出行号，仅针对非空白行做行号显示，空白行不标行号！
-E  ：将结尾的断行字元 $ 显示出来；
-n  ：列印出行号，连同空白行也会有行号，与 -b 的选项不同；
-T  ：将 [tab] 按键以 ^I 显示出来；
-v  ：列出一些看不出来的特殊字符
```

#### tac 反向显示

从文件最后一行显示到第一行

```bash
[root@localhost ~]# tac /etc/issue

Kernel \r on an \m
\S
[root@localhost ~]# cat /etc/issue
\S
Kernel \r on an \m
```

#### nl 添加行号打印

```bash
[root@localhost ~]# nl
选项与参数：
-b  ：指定行号指定的方式，主要有两种：
      -b a ：表示不论是否为空行，也同样列出行号(类似 cat -n)；
      -b t ：如果有空行，空的那一行不要列出行号(预设值)；
-n  ：列出行号表示的方法，主要有三种：
      -n ln ：行号在萤幕的最左方显示；
      -n rn ：行号在自己栏位的最右方显示，且不加 0 ；
      -n rz ：行号在自己栏位的最右方显示，且加 0 ；
-w  ：行号栏位的占用的字元数。
[root@localhost ~]# nl /etc/issue
     1  \S
     2  Kernel \r 
```

#### more 

可以一页一页翻动文档

```bash
[root@localhost ~]# more /etc/sysconfig/network-scripts/ifdown-routes 
#! /bin/bash
#
# Drops static routes which go through device $1

--More--(68%)   <---命令行模式
```

操作：

- 空格键 (space)：代表向下翻一页；
- Enter         ：代表向下翻『一行』；
- /字串         ：代表在这个显示的内容当中，向下搜寻『字串』这个关键字；
- :f            ：立刻显示出档名以及目前显示的行数；
- q             ：代表立刻离开 more ，不再显示该文件内容。
- b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管线无用。

#### less

```bash
less  /etc/sysconfig/network-scripts/ifdown-routes
#! /bin/bash
#
# Drops static routes which go through device $1

:     <-- 指令输入处
```

操作：
- 空格键    ：向下翻动一页；
- [pagedown]：向下翻动一页；
- [pageup]  ：向上翻动一页；
- /字串     ：向下搜寻『字串』的功能；
- ?字串     ：向上搜寻『字串』的功能；
- n         ：重复前一个搜寻 (与 / 或 ? 有关！)
- N         ：反向的重复前一个搜寻 (与 / 或 ? 有关！)
- g         ：前进到这个资料的第一行去；
- G         ：前进到这个资料的最后一行去 (注意大小写)；
- q         ：离开 less 这个程式；
- 方向键     ：逐行显示

#### head 取出开头几行

```bash
#取第一行
[root@localhost ~]# head -n 1 /etc/issue
\S
```

#### tail 取出后几行

```bash
#取出后两行，最后一行为空白行
[root@localhost ~]# tail -n 2 /etc/issue
Kernel \r on an \m

```

## 2.6 命令与文件搜索

### 2.6.1 命令搜索

#### which

```bash
[root@localhost ~]# which ls
alias ls='ls --color=auto'
        /usr/bin/ls
```

### 2.6.2 文件搜索

#### find

```bash
[root@localhost ~]# find [PATH] [option] [action]
选项与参数：
1. 与时间有关的选项：共有 -atime, -ctime 与 -mtime ，以 -mtime 说明
   -mtime  n ：n 为数字，意义为在 n 天之前的『一天之内』被更动过内容的文件；
   -mtime +n ：列出在 n 天之前(不含 n 天本身)被更动过内容的文件档名；
   -mtime -n ：列出在 n 天之内(含 n 天本身)被更动过内容的文件档名。
   -newer file ：file 为一个存在的文件，列出比 file 还要新的文件档名

 #查找   /root/Console_setup/ 目录下 24小时修改过的文件
[root@localhost ~]# find /root/Console_setup/ -mtime 0
/root/Console_setup/main_menu.sh
/root/Console_setup/.git
/root/Console_setup/.git/FETCH_HEAD
```
- +4代表大于等于5天前的档名：ex> find /var -mtime +4
- -4代表小于等于4天内的文件档名：ex> find /var -mtime -4
- 4则是代表4-5那一天的文件档名：ex> find /var -mtime 4

```bash
[root@localhost ~]# find [PATH] [option] [action]
选项与参数：
2. 与使用者或群组名称有关的参数：
   -uid n ：n 为数字，这个数字是使用者的帐号 ID，亦即 UID ，这个 UID 是记录在
            /etc/passwd 里面与帐号名称对应的数字。这方面我们会在第四篇介绍。
   -gid n ：n 为数字，这个数字是群组名称的 ID，亦即 GID，这个 GID 记录在
            /etc/group，相关的介绍我们会第四篇说明～
   -user name ：name 为使用者帐号名称喔！例如 dmtsai 
   -group name：name 为群组名称喔，例如 users ；
   -nouser    ：寻找文件的拥有者不存在 /etc/passwd 的人！
   -nogroup   ：寻找文件的拥有群组不存在于 /etc/group 的文件！
                当你自行安装软体时，很可能该软体的属性当中并没有文件拥有者，
                这是可能的！在这个时候，就可以使用 -nouser 与 -nogroup 搜寻。

#找出 /PATH 下属于 username 的文件
[root@localhost ~]# find /PATH -user USERNAME

#寻找系统中不属于任何人的文件
[root@study ~]# find / -nouser
```

```bash
选项与参数：
3. 与文件权限及名称有关的参数：
   -name filename：搜寻文件名称为 filename 的文件；
   -size [+-]SIZE：搜寻比 SIZE 还要大(+)或小(-)的文件。这个 SIZE 的规格有：
                   c: 代表 byte， k: 代表 1024bytes。所以，要找比 50KB
                   还要大的文件，就是『 -size +50k 』
   -type TYPE    ：搜寻文件的类型为 TYPE 的，类型主要有：一般正规文件 (f), 装置文件 (b, c),
                   目录 (d), 连结档 (l), socket (s), 及 FIFO (p) 等属性。
   -perm mode  ：搜寻文件权限『刚好等于』 mode 的文件，这个 mode 为类似 chmod
                 的属性值，举例来说， -rwsr-xr-x 的属性为 4755 ！
   -perm -mode ：搜寻文件权限『必须要全部囊括 mode 的权限』的文件，举例来说，
                 我们要搜寻 -rwxr--r-- ，亦即 0744 的文件，使用 -perm -0744，
                 当一个文件的权限为 -rwsr-xr-x ，亦即 4755 时，也会被列出来，
                 因为 -rwsr-xr-x 的属性已经囊括了 -rwxr--r-- 的属性了。
   -perm /mode ：搜寻文件权限『包含任一 mode 的权限』的文件，举例来说，我们搜寻
                 -rwxr-xr-x ，亦即 -perm /755 时，但一个文件属性为 -rw-------
                 也会被列出来，因为他有 -rw.... 的属性存在！
# 找出名为 passwd 的文件
[root@localhost ~]# find / -name passwd
/sys/fs/selinux/class/passwd
/sys/fs/selinux/class/passwd/perms/passwd
/etc/passwd
/etc/pam.d/passwd
/usr/bin/passwd

#找出文件名包含了 passwd 这个关键字的文件
[root@localhost ~]# find / -name "*passwd*"

#找出系统中，大于 1MB 的文件
[root@localhost ~]# find / -size +1M
```

## 2.7 文件的打包和压缩

### 2.7.1 压缩文件

```bash
*.Z         compress 程式压缩的文件；
*.zip       zip 程式压缩的文件；
*.gz        gzip 程式压缩的文件；
*.bz2       bzip2 程式压缩的文件；
*.xz        xz 程式压缩的文件；
*.tar       tar 程式打包的资料，并没有压缩过；
*.tar.gz    tar 程式打包的文件，其中并且经过 gzip 的压缩
*.tar.bz2   tar 程式打包的文件，其中并且经过 bzip2 的压缩
*.tar.xz    tar 程式打包的文件，其中并且经过 xz 的压缩
```

### 2.7.2 gzip, zcat/zmore/zless/zgrep

```bash
[root@localhost ~]# gzip [-cdtv#] 文件名
[root@localhost ~]# zcat 文件名.gz
选项与参数：
-c  ：将压缩的资料输出到屏幕上，可透过资料流重导向来处理；
-d  ：解压缩的参数；
-t  ：可以用来检验一个压缩文件的一致性～检测压缩文件有无错误；
-v  ：可以显示出原文件/压缩文件的压缩比等信息；
-#  ：# 为数字的意思，代表压缩等级，-1 最快，但是压缩比最差、-9 最慢，但是压缩比最好！预设是 -6

#压缩文件，压缩后原文件将不存在
[root@blog test]# ls
test.txt
[root@blog test]# gzip -v test.txt 
test.txt:         0.0% -- replaced with test.txt.gz
[root@blog test]# ls
test.txt.gz
#解压后，压缩文件也不存在
[root@blog test]# gzip -d test.txt.gz 
[root@blog test]# ls
test.txt
#使用 zcat zmore zless 读取压缩过的文本文件
[root@blog test]# echo "Hello World" >> test.txt 
[root@blog test]# cat test.txt 
Hello World
[root@blog test]# gzip -v test.txt 
test.txt:       -16.7% -- replaced with test.txt.gz
[root@blog test]# zcat test.txt.gz 
Hello World
[root@blog test]# 
```

### 2.7.3 bzip2, bzcat/bzmore/bzless/bzgrep

若说 gzip 是为了取代 compress 并提供更好的压缩比而成立的，那么 bzip2 则是为了取代 gzip 并提供更佳的压缩比而来的。 bzip2 真是很不错用的东西～这玩意的压缩比竟然比 gzip 还要好～至于 bzip2 的用法几乎与 gzip 相同！看看底下的用法吧！

```bash
[root@blog test]# bzip2 [-cdkzv#] 文件名
[root@blog test]# bzcat 文件名.bz2
选项与参数：
-c  ：将压缩的过程产生的资料输出到屏幕上！
-d  ：解压缩的参数
-k  ：保留原始文件，而不会删除原始的文件喔！
-z  ：压缩的参数 (预设值，可以不加)
-v  ：可以显示出原文件/压缩文件的压缩比等信息；
-#  ：与 gzip 同样的，都是在计算压缩比的参数， -9 最佳， -1 最快！

#压缩文件，压缩后原文件将不存在
[root@blog test]# bzip2 -v test.txt 
  test.txt:  0.214:1, 37.333 bits/byte, -366.67% saved, 12 in, 56 out.
[root@blog test]# ls
test.txt.bz2
#使用 bzcat bzmore bzless 读取压缩过的文本文件
[root@blog test]# bzcat test.txt.bz2 
Hello World
#解压
[root@blog test]# bzip2 -d test.txt.bz2 
[root@blog test]# ls
test.txt
```

### 2.7.4 xz, xzcat/xzmore/xzless/xzgrep

用法跟上面差不多

```bash
[root@blog test]# xz [-dtlkc#] 档名
[root@blog test]# xcat 档名.xz
选项与参数：
-d  ：解压缩
-t  ：测试压缩档的完整性，看有没有错误
-l  ：列出压缩档的相关信息
-k  ：保留原本的文件不删除～
-c  ：同样的，就是将资料由屏幕上输出的意思
-#  ：同样的，也有较佳的压缩比的意思

#压缩文件，压缩后保留原文件
[root@blog test]#  xz -k test.txt 
[root@blog test]# ls
test.txt  test.txt.xz
```

## 2.8 tar

 tar 可以将多个目录或档案打包成一个大档案，同时还可以透过 gzip/bzip2/xz 的支持，将该档案同时进行压缩

 ```bash
[root@blog test]# tar [-z|-j|-J] [cv] [-f 待建立的新档名] filename... <==打包与压缩
[root@blog test]# tar [-z|-j|-J] [tv] [-f 既有的 tar档名]             <==察看档名
[root@blog test]# tar [-z|-j|-J] [xv] [-f 既有的 tar档名] [-C 目录]   <==解压缩
选项与参数：
-c  ：建立打包档案，可搭配 -v 来察看过程中被打包的档名(filename)
-t  ：察看打包档案的内容含有哪些档名，重点在察看『档名』就是了；
-x  ：解打包或解压缩的功能，可以搭配 -C (大写) 在特定目录解开
      特别留意的是， -c, -t, -x 不可同时出现在一串指令列中。
-z  ：透过 gzip  的支援进行压缩/解压缩：此时档名最好为 *.tar.gz
-j  ：透过 bzip2 的支援进行压缩/解压缩：此时档名最好为 *.tar.bz2
-J  ：透过 xz    的支援进行压缩/解压缩：此时档名最好为 *.tar.xz
      特别留意， -z, -j, -J 不可以同时出现在一串指令列中
-v  ：在压缩/解压缩的过程中，将正在处理的档名显示出来！
-f filename：-f 后面要立刻接要被处理的档名！建议 -f 单独写一个选项啰！(比较不会忘记)
-C 目录    ：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。

其他后续练习会使用到的选项介绍：
-p(小写) ：保留备份资料的原本权限与属性，常用于备份(-c)重要的设定档
-P(大写) ：保留绝对路径，亦即允许备份资料中含有根目录存在之意；
--exclude=FILE：在压缩的过程中，不要将 FILE 打包！ 
 ```

**简单记忆：**
- 压　缩：tar -jcv -f filename.tar.bz2(压缩后文件名) filename(要被压缩的文件或目录名称)
- 查　询：tar -jtv -f filename.tar.bz2
- 解压缩：tar -jxv -f filename.tar.bz2 -C 欲解压缩的目录

```bash
#压缩 test.txt 文件和 test 目录
[root@blog test]# ll
total 4
drwxr-xr-x. 2 root root  6 Sep 25 21:25 test
-rw-r--r--. 1 root root 12 Sep 25 20:41 test.txt
[root@blog test]# tar -jcv -f test.tar.bz2 test.txt test
test.txt
test/
[root@blog test]# ls
test  test.tar.bz2  test.txt
#解压，先删除原文件
[root@blog test]# rm -rf test test.txt
[root@blog test]# tar -jxv -f test.tar.bz2 
test.txt
test/
[root@blog test]# ls
test  test.tar.bz2  test.txt
#查看压缩文件内容
[root@blog test]# tar -jtv -f test.tar.bz2 
-rw-r--r-- root/root        46 2020-09-25 21:29 test.txt
drwxr-xr-x root/root         0 2020-09-25 21:25 test/
```

# 三、 文件系统

## 3.1 分区

[关于 Linux 分区的小事](https://ecarry.cc/2020/06/09/Linux_part/)

## 3.2 文件系统选择

[Linux 文件系统的选择](https://ecarry.cc/2020/02/11/linux_fs/)

## 3.3 超级块和 i-node

[Linux 超级块和 i-node](https://ecarry.cc/2020/09/02/linux_super_inode_block/)

# 四、 VIM

**VIM 特点**

- 所有的 Unix Like 系统都会内建 vi 文书编辑器，其他的文书编辑器则不一定会存在；
- 很多个别软件的编辑介面都会主动呼叫 vi (例如未来会谈到的 crontab, visudo, edquota 等指令)；
- vim 具有程式编辑的能力，可以主动的以字体颜色辨别语法的正确性，方便程式设计；
- 编辑速度快

## 4.1 VIM 编辑器

## 4.2 VIM 模式

### 4.2.1 命令模式

### 4.2.2 编辑模式

### 4.2.3 可视化模式

### 4.2.4 末行模式

## 3.3 VIM 相关操作

## 3.4 VIM 项目实战

# 五、 用户和组

## 4.1 用户和组的概念


## 4.2 用户管理操作

## 4.3 用户组管理操作

## 4.4 企业级用户管理实战

# 五、 管道命令和 ssh

## 5.1 管道命令 |

## 5.2 grep 

## 5.3 网络配置

### 5.3.1 网络相关命令

## 5.4 SSH

### 5.4.1 ssh 协议

### 5.4.2 sshd 服务

## 5.5 远程管理和文件传输

# 六、 权限管理

## 6.1 文件权限描述

## 6.2 linux 权限与用户身份类别

## 6.4 文件权限设置

## 6.5 权限扩展

设置位 S 与 沾附位 T

## 6.6 ACL 权限

## 6.7 UMASK

# 七、 服务管理 

## 7.1 systemctl

## 7.2 ntpd 时间同步

## 7.3 firewalld 防火墙

## 7.4 rpm 包管理

## 7.5 crond 计划任务详解

# 八、 环境配置

## 8.1 LAMP 环境概述

## 8.2 LAMP 环境编译安装

## 8.3 YUM 源配置

## 8.4 YUM 配置 LAMP 环境

