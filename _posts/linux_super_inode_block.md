---
layout: post
title: Linux 超级块和 i-node
tags: [Linux]
index_img: https://th.wallhaven.cc/small/0w/0wv2p6.jpg
banner_img: https://w.wallhaven.cc/full/0w/wallhaven-0wv2p6.jpg
categories: [Linux]
date: 2020-09-2 10:33:36
---

# 一、从物理磁盘到文件系统

文件系统用来存储文件内容，文件属性和目录。且在 Linux 里万物为文件

Linux 文件系统中，是以块为单位存储信息，把硬盘分为三个部分：**超级块（Super block）**，**i-节点表（inode table）**，**数据块（data block）**。

要实现文件系统组件，首先需要了解 SimpleFS 磁盘布局。如前所述，此项目假定每个磁盘块的大小为4KB。磁盘的第一个块是描述文件系统其余部分布局的超级块。超级块之后的一定数量的块包含 inode 数据结构(即 i-node 表)。通常，磁盘块总数的10% 用作 inode 块(即保留用于存储 inode 表的块)。文件系统中剩余的块用作普通数据块，有时用作间接指针块，如下面的例子所示:

![](/img/linux_super_inode_block/linux_inode_1.jpg)

**Magic**: The first field is always the MAGIC_NUMBER or 0xf0f03410. The format routine places this number into the very first bytes of the superblock as a sort of filesystem “signature”. When the filesystem is mounted, the OS looks for this magic number. If it is correct, then the disk is assumed to contain a valid filesystem. If some other number is present, then the mount fails, perhaps because the disk is not formatted or contains some other kind of data.（第一个字段总是 MAGIC 数字或0xf0f03410。Format 例程将这个数字作为一种文件系统“签名”放在超级块的最初字节中。当文件系统被安装时，操作系统会寻找这个神奇的数字。如果是正确的，则假定磁盘包含有效的文件系统。如果存在其他数字，则挂载失败，可能是因为磁盘没有格式化或包含其他类型的数据。）

>参考资料：https://www3.nd.edu/~pbui/teaching/cse.30341.fa18/project06.html

## 1.1 超级块（Super block）
文件系统中第一个块被称为**超级块**。超级块记录了该 **filesystem** 的整体信息，其中包含：

- block 与 inode 的总量
- 未使用与已使用的 inode/block 数量
- 一个 block 与 一个 inode 的大小（block 在ext2 中为1，2，4k；inode 为 128bytes或256bytes）
- filesystem 的挂载时间，最近一次写入资料的时间，最近一次检验磁盘（fsck）的时间等档案系统的相关信息
- 一个 valid bit 数值，若该文件系统已被挂载，则 valid bit 为0，若未被挂载，则 valid bit 为1

一般来说， superblock 的大小为 **1024bytes**。

## 1.2 i-节点表（inode table）

超级块下一部分就是 **i-节点表** 了，该数据结构管理所有文件的属性。每个Linux文件或目录 (从技术角度讲，它们之间没有本质的区别，都为文件) 都有一个inode，而这个inode包含了所有文件的元数据 (也就是说，读取文件所需的管理数据都存储在inode中)。例如，inode包含存储文件的所有块的列表，该文件所有者信息、权限以及为该文件设置的所有其他属性。

所有i-节点都有相同的大小，并且i-节点表是这些结构的一个列表，文件系统中每个文件在该表中都有一个i-节点。

### 1.2.1 inode number
**inode** 是 inode table 中的一个条目，包含有关目录和常规文件的元数据。inode是传统Unix风格文件系统 (比如ext3/ext4) 上的数据结构。Linux扩展文件系统 (如ext2/ext3) 维护了一个inode的数组：inode table。inode table 包含该文件系统中所有文件的列表。inode table 中的各个inode项具有唯一的编号 (该文件系统唯一)，即inode number。深入inode数据结构，我们发现它存储了如下信息：

- 文件类型： 普通文件，目录，管道等等
- 权限：可读，可写，可执行(read/write/excute)
- 链接数：链接到该inode的硬链接数
- User ID：文件所有者
- Group ID：所有者组ID
- 文件大小
- 时间信息：创建时间或状态改变时间（ctime）、最近修修改时间（mtime）、最近读取时间（atime）
- 属性：比如，不可改变位
- 访问控制列表
- 文件数据存储的实际位置
- 其他元数据

<font color=red>注意：inode中不存储文件名数据，文件名存储在目录</font>

**特性：**

- 每个 inode 大小均固定为 128bytes（ext4和xfs可设定到256bytes）
- 每个档案仅会占用一个 inode
- 所以，文件系统能够创建的文件数与 inode 的数量有关
- 系统读取档案时需要先找到 inode，并分析 inode 所记录的权限与使用者是否相符，符合才能够开始实际读取 data blcok 里文件的内容

## 1.3 数据块（data block）
文件系统的第三个部分就是**数据块区**。文件的本身内容就保存在该区域。磁盘上所有的块大小都一样，如在**Ext2档案系统中所支持的block大小有1K, 2K及4K三种**。格式化时block的大小就固定了，且每个block都有编号，以方便inode的记录。<font color=red>如果文件包含超过一个数据块的内容，则文件内容会存放在多个磁盘块中</font>。

# 二、文件及目录的创建过程

## 2.1 文件创建过程

使用 `touch filename` 创建文件：

文件的属性和文件的内容：<font color=red>内核将文件的内容放入data block，将文件的属性存放在 i-节点，文件名存放在目录</font>。

下图显示了创建一个文件的例子，该文件占用了3个数据块：

![](/img/linux_super_inode_block/linux_inode_2.jpg)

步骤：

1. 文件属性存储：内核找到一块空的 i-节点，该节点 i-number 为47。内核把该文件的信息记录其中，这些信息详见 1.2.1 inode number

2. 文件内容的存储：由于该文件需要3个数据块，内核需要在自由块列表中找到3个自由块。

3. 记录分配情况，数据保存到了3个数据块中，所以需要记录下来。分配情况记录在 i-节点中的磁盘序号列表里。这3个编号分别放在最开始的3个位置

4. 添加文件名到目录，新的文件名为 filename，内核将文件的入口（47，filename）添加到目录文件里。文件名和 i-节点号之间的对应关系将文件名和文件的内容属性🔗起来，找到文件名就找到了文件的 i-节点号，通过 i-节点号就能找到文件的属性及内容

## 2.2 目录创建过程

目录也是文件，内容比较特殊，它包含了文件名字列表，而列表一般包含两个部分：**i-节点** 和 **文件名**。