---
layout: post
title: Linux 运维初级学习笔记
tags: [Linux]
index_img: https://th.wallhaven.cc/small/md/mdzdok.jpg
banner_img: https://w.wallhaven.cc/full/md/wallhaven-mdzdok.jpg
categories: [Linux]
date: 2020-08-18 17:03:07
---

# 一、 了解 Linux

## 1.1 Linux 的基本原则

1. 由目的单一的小程序组成；组合小程序完成复杂任务
2. 一切皆文件
3. 尽量避免捕获用户接口
4. 配置文件保存为纯文本格式

## 1.2 GUI

```bash
Graphic User Interface
Windows
X-Window
    Gnome: C
    RDE: C++
    XFace
```

## 1.3 CLI

```bash
Command-line Interface
sh
bash
csh
zsh
ksh
ksh
tcsh
命令提示符，prompt，bash(shell)
    #: root
    $: 普通用户
命令格式
    命令   选项     参数
            选项：
                短选项：-
                    多个选项可以组合：ls -a -l = ls -al
                长选项：--

            参数：命令作用的对象
```  

## 1.4 su(SwitchUser，切换用户)

```bash
    su - run a command with substitute user and group ID

[root@localhost ~]# su [-l] 用户名
```

## 1.5 API(Application program Interface)

 .so 文件：Shared Object 动态链接库，库文件，应用程序所需要的

# 二、 常用命令

## 2.1 ls

```bash
ls
    -l：长格式
        文件类型：
            -：普通文件（file）
            d：目录文件（directory）
            b：块设备文件（block）
            c：字符设备文件（character）
            l：符号链接文件（symbolic link file）
            p：命令管道文件（pipe）
            s：套接字文件（socket）
[root@localhost ~]# ls -l /dev
crw-------. 1 root root     10, 235 7月  20 18:39 autofs
drwxr-xr-x. 2 root root         200 8月   9 16:36 block
lrwrwxxrwx. 1 root root           3 7月  20 18:39 cdrom -> sr0
brw-rw----. 1 root cdrom    11,   0 7月  20 18:39 sr0
srw-rw-rw-. 1 root root           0 7月  20 18:39 log

    -a：显示以 . 开头的隐藏文件
    -i：index node，inode
        [root@localhost ~]# ls -li
        34051578 test
    -R：递归（recursive）显示

```

## 2.2 cd

```bash
cd：change directory
    cd ~USERNAME：进入指定用户的 home 目录
    cd -：在当前目录和前一次所在目录之间来回切换
```

## 2.3 type

显示指定命令属于哪种类型
内置命令（shell 内置）：内部的，内建
外部命令：在文件系统的某个路径下有一个命令名称相应的可执行文件

```bash
[root@localhost ~]# type cd
cd 是 shell 内嵌        #内置命令
[root@localhost ~]# which ls
alias ls='ls --color=auto'  #外部命令
        /usr/bin/ls
```

## 2.4 打印环境变量

```bash
显示环境变量
[root@localhost ~]# printevn
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

## 2.5 命令选项

* <>：必选项
* []：可选项
* ...：可多次出现
* |：多选一
* {}：分组

# 三、 文件系统操作

## 3.1 目录详解

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
/opt：可选目录
/proc：伪文件系统，内核映射文件
/sys：伪文件系统，跟硬件设备相关的属性映射文件
/tmp：临时文件
/var：可变化文件，如存放日志文件
/bin：可执行文件，用户命令
/sbin：管理命令
/usr：Unix Software Resource(shared,read-only),操作系统软件资源所放置的目录,所有系统默认的软件都会放置到/usr
```

## 3.2 管理

### 3.2.1 文件管理

#### touch

* touch：change file timestamps
    -a：修改访问时间
    -m：修改更改时间
    -t：修改到指定时间

#### stat

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

#### cp

### 3.2.2 目录管理

* ls
* cd
* pwd
* mkdir：创建空目录
    -p：依次创建层级目录，如 mkdir -p /1/2/3
    -v：显示详情

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

* rmdir：删除空目录

### 3.2.3 文本处理

```bash
cut：
    -d：指定字段分隔符，默认是空格
    -f：指定要显示的字段
        -f 1，3 显示字段1和3的字符
        -f 1-3 显示字段1到3的字符
```

### 3.2.4 命令历史

```bash
history：查看历史命令
    -c：清空命令历史
    -d OFFSET [n]：删除指定位置的命令
    -w：保存命令历史至历史文件中
!n：执行命令历史的第N条命令
```

### 3.2.5 grep

根据模式，搜索文本，并将符合模式的文本行显示出来
Pattern：文本字符和正则表达式的元字符组合而成的匹配条件

```bash
grep [OPTIONS] PATTERN [FILE...]
        -i：忽略大小字母
        -v：显示没有被匹配到的行
        -o：只显示被模式匹配到的字符串
        -E：使用扩展正则表达式
        -A #：显示匹配到的后#行
        -B #：显示匹配到的前#行
        -C #：显示匹配到的上下#行
```

### 3.2.6 正则表达式（REGular EXPression, REGEXP）

**元字符：**

.：匹配其前面的字符任意次

[ ]：匹配指定范围内的任意单字符

[^]：匹配指定范围外的任意单个字符

|字符簇|描述|
|:----|:----|
|[[:alpha:]]|任何字母|
|[[:digit:]]|任何数字|
|[[:alnum:]]|任何字母和数字|
|[[:space:]]|任何空白字符|
|[[:upper:]]|任何大写字母|
|[[:lower:]]|任何小写字母|
|[[:punct:]]|任何标点符号|
|[[:xdigit:]]|任何16进制的数字，相当于[0-9a-fA-F]|

**匹配次数：**

```bash
*：匹配其前面的字符任意次

\?：匹配其前面的的字符1次或0次

\{m,n\}：匹配其前面的字符至少m次，至多n次
```

**位置锚定：**

```bash
^：锚定行首，此字符后面的任意内容必须出现在行首
[root@localhost learing]# cat test_grep_fenzu.txt 
 1space
  2space
   3space
#显示行首至少有一个空白字符
[root@localhost learing]# grep -E '^[[:space:]]+' test_grep_fenzu.txt 
 1space
  2space
   3space

$：锚定行尾，此字符后面的任意内容必须出现在行尾

^$：空白行

\<或\b：锚定词首，其后面的任意字符必须作为单次首部出现

\>或\b：锚定词尾，其前面的任意字符必须作为单次尾部出现
#使用 \<()\> 找出1-255的数字
[root@localhost learing]# ifconfig | grep -E '\<([1-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>'
        inet 192.168.2.22  netmask 255.255.255.0  broadcast 192.168.2.255
        inet6 fd2a:2d84:d032:0:650f:b141:8e42:b014  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::3416:22f7:9cb1:88ba  prefixlen 64  scopeid 0x20<link>
        inet6 fd2a:2d84:d032::778  prefixlen 128  scopeid 0x0<global>
        RX packets 976525  bytes 271923354 (259.3 MiB)
        TX packets 884321  bytes 110229028 (105.1 MiB)

#查找 ip 地址
[root@localhost learing]# ifconfig | grep -E '(\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>\.){3}\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>'
        inet 192.168.2.22  netmask 255.255.255.0  broadcast 192.168.2.255
        inet 127.0.0.1  netmask 255.0.0.0

```

**分组：**

```bash
\(\)
    \(ab\)：ab为一整体出现
[root@localhost learing]# cat test_grep_fenzu.txt 
cat
Cat
china
c
C
China
 1space
  2space
   3space

#not use ()
[root@localhost learing]# grep -E 'C|cat' test_grep_fenzu.txt 
cat
Cat
C
China

#use ()
[root@localhost learing]# grep -E '(C|c)at' test_grep_fenzu.txt 
cat
Cat

```


# 四、 bash 

## 4.1 变量类型

**变量算法：**

$[$A-$B]：变量 A 减去 变量 B

## 4.2 引用变量

${VAR_NAME}，一般花括号可省略，某些情况无法省略
```bash
#无法省略
[root@localhost learing]# ANIMAL=pig
[root@localhost learing]# echo "there are some $ANIMALs."
there are some .
[root@localhost learing]# echo "there are some ${ANIMAL}s."
there are some pigs.
```

## 4.3 程序执行返回代码

程序执行返回状态码（0-255）：

- 0：正确执行
- 1-255：错误执行，1、2、127 为系统预留

## 4.4 条件判断

### 4.4.1 条件测试类型

- 整数测试
- 字符测试
- 文件测试

### 4.4.2 条件测试表达式

- [ expression ]
- [[ expression]]

### 4.4.3 命令逻辑关系

**逻辑与：&&**
- 第一个条件为假时，第二个条件就不用再判断，最总结果为假
- 第一个条件为真是，还需要判断第二个条件
  
例子：

```bash
#判断用户 user1 是否存在，不存在就增加 user1 用户
! id user1 && useradd user1
```

**逻辑或：||**
有个真，就是真
例子：
```bash
#判断用户 user1 存在，就显示用户已存在，否则，就添加用户
id user1 && echo 'user1 exists.' || useradd user1
#判断用户 user1 不存在，就添加用户，否则，显示已存在
! id user1 && useradd user1 || echo 'user1 exists.'
```

### 4.4.4 shell 算术运算

1. let 算数表达式
   - let C=\$A+\$B
2. $[算术表达式]
   - \$[$A-$B]
3. $((算术表达式))
   - \$((算术表达式))

### 4.4.5 exit

退出脚本

### 4.4.6 文件测试
- -e File：测试文件是否存在
- -f File：测试文件是否为普通文件
- -d File：测试指定目录是否为目录
- -r File：测试当前用户对指定文件是否有可读权限
- -w
- -x


## 4.5 sed (Stream EDitor)

<font color=red>默认不编辑原文件，仅对模式空间的数据做处理</font>

```bash
sed [options] 'AddressCommand' file1 [file2 ...]
    -n：静默模式，不再默认显示模式空间的内容
    -i：直接修改原文件
    -e script：可以同时执行多个脚本
    -f filename：读取 filename 里的脚本处理 sed /path/to/sed_script 作用file
    -r：表示使用扩展正则表达式
```

**Address**
1. StartLine,EndLine: 
    - 1,100 ：1到100行
    - $：最后一行

2. /模式定义/：/^root/
3. /pattern1/,/pattern2/：第一次被 pattern1 匹配到的行，至第一次被 pattern2 匹配到的行结束，中中间的行
4. LineNumber：指定的行数
5. StartLine，+N：从 StartLine 开始，向后的 N行

**Command**

### 4.5.1 d：删除符合条件的行

    - sed '1,3d' /etc/fstab ：删除 fstab 的第 1 到 第 3 行
    - sed '/oot/d' /etc/fstab ：删除包含 oot 的行

### 4.5.2 p：显示符合条件的行

    - sed -n '/^\//a\#sed test' /etc/fstab 
```bash
[root@localhost ~]# sed -n '/^\//p' /etc/fstab 
/dev/mapper/centos-root /                       xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

### 4.5.3 a \string：在指定的行后面追加新行，内容为 string

    - sed '/^\//a\#sed test' /etc/fstab ：在 / 开头的行下行添加一行内容为 #sed test 的行
    - \n 可以换行

```bash
[root@localhost ~]# sed '/^\//a\#sed test' /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Sat Jun  6 20:09:07 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
#sed test
UUID=39b6f37f-531f-41c2-a498-dbe216fcfc4b /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
#sed test
```

### 4.5.4 i \string：在指定的行前面添加新行，内容为 string

### 4.5.5 r filename：将指定文件的内容添加至符合条件的行处

```bash
[root@localhost ~]# cat /etc/issue
\S
Kernel \r on an \m

[root@localhost ~]# sed '1r /etc/issue' /etc/fstab 

\S
Kernel \r on an \m

#
# /etc/fstab
# Created by anaconda on Sat Jun  6 20:09:07 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=39b6f37f-531f-41c2-a498-dbe216fcfc4b /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

### 4.5.6 w filename：将指定范围内的内容另存至指定的文件中

```bash
[root@localhost ~]# sed -n '/^\//w /root/mount.txt' /etc/fstab 
[root@localhost ~]# cat /root/mount.txt 
/dev/mapper/centos-root /                       xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

### 4.5.7 s/pattern/string/[修饰符]：查找并替换，替换每行中第一次被匹配到的字符串

- 修饰符
    - g ：全局替换
    - i：忽略大小写
- s///,s###,s@@@：可使用其他分隔符

```bash
#查找 root 替换成 ROOT，查找能够用正则表达式
[root@localhost ~]# sed 's/root/ROOT/' /etc/fstab 
/dev/mapper/centos-ROOT /                       xfs     defaults        0 0
UUID=39b6f37f-531f-41c2-a498-dbe216fcfc4b /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

### 4.5.8 更改字符串

```bash
#将字符带有 l..k 后面添加 r
[root@localhost learing]# cat sed.txt 
like
love
#第一种，用 &
[root@localhost learing]# sed 's/l..e/&r/' sed.txt 
liker
lover
#第二种，用分组
[root@localhost learing]# sed 's/\(l..e\)/\1r/' sed.txt 
liker
lover
```

### 4.5.9 删除每行开头的空格

```bash
#可删除多个空格
[root@localhost learing]# history | sed 's/^[[:space:]]*//g'
1022  history | sed 's/ //'
1023  history | sed 's/^ //'
1024  history | sed 's/[[:space:]//'
1025  history | sed 's/[[:space:]]//'
1026  history | sed 's/^[[:space:]]+//'
1027  history | sed 's/^[[:space:]]*//'
1028  history | sed 's/^[[:space:]]+1//'
1029  history | sed 's/^[[:space:]]*//'
```

## 4.6 字符串测试

### 4.6.1 == 判断相等

```bash
[root@localhost ~]# A=HELLO
[root@localhost ~]# B=HI
[root@localhost ~]# [ $A == $B ]
[root@localhost ~]# echo $?
1
```

## 4.6.2 ！= 判断不等

```bash
[root@localhost ~]# A=HELLO
[root@localhost ~]# B=HI
[root@localhost ~]# [ $A != $B ]
[root@localhost ~]# echo $?
0
```

## 4.6.3 -n string 字符串为空

## 4.6.4 -s string 字符串不为空

# 五、 存储

## 5.1 接口

- IDE

理论速度：133Mbps，并行

- SATA

理论速度：300Mbps，串行

- SATA2

理论速度：600Mbps，串行

- SATA3

理论速度：6Gbps，串行

- USB3.0

理论速度：480Mbps，串行

- SCSI

SCSI：Small Computer System Interface，并行

## 5.2 RAID

TODO...

## 5.3 mdadm 实现软 RAID

# 六、 网络

# 七、 进程

## 7.1 内核数据结构

- task structure
  - PPID
  - PID
  - Name 
