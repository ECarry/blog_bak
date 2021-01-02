---
layout: post
title: Linux LVM
tags: [Linux]
index_img: https://th.wallhaven.cc/small/4l/4l36rl.jpg
banner_img: https://w.wallhaven.cc/full/4l/wallhaven-4l36rl.png
categories: [Linux]
date: 2020-03-12 14:32:14
---

# 一、LVM (Logical Volume Manager)

LVM是逻辑盘卷管理（Logical Volume Manager）的简称，它是Linux环境下对磁盘分区进行管理的一种机制，LVM 是建立在硬盘和分区之上的一个逻辑层，来提高磁盘分区管理的灵活性。

# 二、LVM 特点

LVM 将存储**虚拟化**，使用逻辑卷，不会受限于物理磁盘的大小。另外，和硬件相关的存储设置被其隐藏，能不用停止应用或卸载文件系统来调整卷大小或数据迁移。这样能减少操作成本，LVM最大的特点就是可以对磁盘进行动态管理。因为逻辑卷的大小是可以动态调整的，而且不会丢失现有的数据。如果我们新增加了硬盘，其也不会改变现有上层的逻辑卷。作为一个动态磁盘管理机制，逻辑卷技术大大提高了磁盘管理的灵活性。

# 三、LVM 工作机制

LVM就是通过将底层的物理硬盘抽象的封装起来，然后以逻辑卷的方式呈现给上层应用。在传统的磁盘管理机制中，我们的上层应用是直接访问文件系统，从而对底层的物理硬盘进行读取。而在LVM中，其通过对底层的硬盘进行封装，当我们对底层的物理硬盘进行操作时，其不再是针对于分区进行操作，而是通过一个叫做逻辑卷的东西来对其进行底层的磁盘管理操作。

## 3.1 逻辑卷管理

### 3.1.1 物理卷（PV，Physical Volume）

物理卷就是指磁盘，磁盘分区或从逻辑上和磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有和LVM相关的管理参数。当前 LVM 允许你在每个物理卷上保存这个物理卷的0至2份元数据拷贝，默认为1，保存在设备的开始处。为2时，在设备结束处保存第二份备份。

### 3.1.1 卷组（VG，Volume Group）

LVM 卷组类似于非 LVM 系统中的物理硬盘，其由物理卷组成。能在卷组上创建一个或多LVM分区（逻辑卷）， LVM卷组由一个或多个物理卷组成。

### 3.1.1 逻辑卷（LV，Logical Volume）

LVM的逻辑卷类似于非 LVM 系统中的硬盘分区，在逻辑卷之上能建立文件系统(比如/home或/usr等)。

关系图：

![](/img/linux_lvm/linux_lvm_1.jpg)

# 四、创建 LVM

## 4.1 检查系统是否安装 LVM 管理工具

```bash
[root@localhost ~]# rpm -qa |grep lvm
lvm2-2.02.186-7.el7_8.2.x86_64
lvm2-libs-2.02.186-7.el7_8.2.x86_64
#未安装可使用 yum 安装
[root@localhost ~]# yum install lvm*
```

## 4.2 创建物理卷 PV

基本 PV 命令：

- pvcreate 将物理分区新建为 PV
- pvs/pvscan 查看系统里有 PV 的磁盘
- pvdisplay 显示系统上 PV 的状态
- pvremove 删除 PV

```bash
[root@localhost ~]# lsblk 
NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sdb               8:16   0   2G  0 disk 
sdc               8:32   0   2G  0 disk 
sdd               8:48   0   2G  0 disk 
sde               8:64   0   2G  0 disk 
sdf               8:80   0   2G  0 disk
```

准备使用 **sdb**、**sdc** 来创建 LVM：

```bash
[root@localhost ~]# pvcreate /dev/sdb /dev/sdc
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
#查看创建的 pv
[root@localhost ~]# pvs
  PV         VG     Fmt  Attr PSize   PFree
  /dev/sdb          lvm2 ---    2.00g 2.00g
  /dev/sdc          lvm2 ---    2.00g 2.00g
```

## 4.3 创建卷组 VG

基本 VG 命令：

- vgcreat [-s xM] vg_name /dev/sd.. 新建 vg，-s 后面接 pe 的大小（可选），单位是 M、G。可放多块 pv
- vgextend 扩展 vg，就是增加 pv
- vgs/vgscan 查看系统里有 vg 的磁盘
- vgdisplay 显示系统上的 vg 状态
- vgremove 删除 vg
- vgreduce 在 vg 里删除 pv

创建名为 vg_test ：

```bash
[root@localhost ~]# vgcreate -s 16M vg_test /dev/sdb
  Volume group "vg_test" successfully created
#查看创建好的 vg
[root@localhost ~]# vgs
  VG      #PV #LV #SN Attr   VSize   VFree
  vg_test   1   0   0 wz--n-   1.98g 1.98g
```

## 4.4 创建逻辑卷 lv

基本 LV 命令：

- lvcreate (-l pe num)/(-L size) -n lv_name vg_name 新建 lv，lv 大小两种可选：1. -l 指定 pe 的个数，创建vg时有指定 pe大小 2.-L直接指定要创建 lv 的容量，单位为 M、G
- lvextend 扩容
- lvs/lvscan 查看系统里有 lv 的磁盘
- lvdisplay 显示系统上 lv 的状态
- lvremove 删除 lv
- lvreduce 在 lv 里减少容量

创建 lv_test

```bash
#查看 vg_test 状态
[root@localhost ~]# vgdisplay 
  --- Volume group ---
  VG Name               vg_test
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1.98 GiB
  PE Size               16.00 MiB
  Total PE              127
  Alloc PE / Size       0 / 0   
  Free  PE / Size       127 / 1.98 GiB
  VG UUID               Oge6D2-KPLh-RqNA-cRJ8-mc1y-0rkC-aoXmVt

#见 pe可用127,我们使用100个 pe
[root@localhost ~]# lvcreate -l 100 -n lv_test vg_test
  Logical volume "lv_test" created.
[root@localhost ~]# lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert                                                 
  lv_test vg_test -wi-a-----   1.56g
```

**格式化刚创建的 lv_test**

```bash
[root@localhost ~]# mkfs.ext4 /dev/vg_test/lv_test 
mke2fs 1.42.9 (28-Dec-2013)
Discarding device blocks: done                            
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
102544 inodes, 409600 blocks
20480 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=419430400
13 block groups
32768 blocks per group, 32768 fragments per group
7888 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done
```

**挂载使用；**

```bash
[root@localhost ~]# mount /dev/vg_test/lv_test /mnt
[root@localhost ~]# mount
/dev/mapper/vg_test-lv_test on /mnt type ext4 (rw,relatime,seclabel,data=ordered)
```

# 五、LVM 在线扩展

先在 lv_test（/mnt） 下写入数据，测试在线扩展

```bash
[root@localhost mnt]# touch hello_lvm
[root@localhost mnt]# ls
hello_lvm
```

<font color=red>扩容 lv 分两种情况，第一是在 vg 容量还有的情况下，也就是Free PE Size 够用情况下。第二是在 vg 不够用，得先扩容 vg 的情况</font>

## 5.1 在 vg 下直接扩容

```bash
#查看 vg 还剩27个 pe
 [root@localhost ~]# vgdisplay 
  Free  PE / Size       27 / 432.00 MiB
#现在 lv 大小
 [root@localhost ~]#lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert                                                  
  lv_test vg_test -wi-ao----   1.56g   

#lv 再扩容 20 个 pe，所以要 -l 120，之前给了 100，再扩容加上20，则120
[root@localhost ~]# lvextend -l 120 /dev/vg_test/lv_test 
  Size of logical volume vg_test/lv_test changed from 1.56 GiB (100 extents) to <1.88 GiB (120 extents).
  Logical volume vg_test/lv_test successfully resized.
#查看实际的磁盘容量，发现并没改变
[root@localhost ~]# df -Th
Filesystem                  Type      Size  Used Avail Use% Mounted on
/dev/mapper/vg_test-lv_test ext4      1.6G  4.7M  1.5G   1% /mnt
#需要对文件系统进行扩容
[root@localhost ~]# resize2fs /dev/vg_test/lv_test 
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vg_test/lv_test is mounted on /mnt; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg_test/lv_test is now 491520 blocks long.
[root@localhost ~]# df -Th
Filesystem                  Type      Size  Used Avail Use% Mounted on
/dev/mapper/vg_test-lv_test ext4      1.9G  4.7M  1.8G   1% /mnt
```

#### tips

- resize2fs /dev/vg_test/lv_test　　＃更新文件系统的大小，即激活
- resize2fs -f　/dev/vg_test/lv_test　500M　＃强制设置大小
- dump2fs /dev/vgtest/lvtest　　＃查看ext系列文件系统

## 5.2 vg 不够用情况，扩容 lv

**vg 空间不够，需先扩展 vg，扩展 vg 就是往 vg 中加 pv**

刚好开始创建 pv 时，多创建了个 sdc，将 pv sdc 加入到 vg_test 里面：

```bash
[root@localhost ~]# pvs
  PV         VG      Fmt  Attr PSize   PFree  
  /dev/sdb   vg_test lvm2 a--    1.98g 112.00m
  /dev/sdc           lvm2 ---    2.00g   2.00g
[root@localhost ~]# vgextend vg_test /dev/sdc 
  Volume group "vg_test" successfully extended
[root@localhost ~]# vgextend vg_test /dev/sdc 
  Volume group "vg_test" successfully extended
[root@localhost ~]# vgdisplay 
  --- Volume group ---
  VG Name               vg_test
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <3.97 GiB
  PE Size               16.00 MiB
  Total PE              254
  Alloc PE / Size       120 / <1.88 GiB
  Free  PE / Size       134 / 2.09 GiB          <-----可用容量变多了
  VG UUID               Oge6D2-KPLh-RqNA-cRJ8-mc1y-0rkC-aoXmVt
#将 pe 数量扩容到 220个
[root@localhost ~]# lvextend -l 220 /dev/vg_test/lv_test 
  Size of logical volume vg_test/lv_test changed from <1.88 GiB (120 extents) to <3.44 GiB (220 extents).
  Logical volume vg_test/lv_test successfully resized.
[root@localhost ~]# resize2fs /dev/vg_test/lv_test 
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vg_test/lv_test is mounted on /mnt; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg_test/lv_test is now 901120 blocks long.

[root@localhost ~]# df -Th
Filesystem                  Type      Size  Used Avail Use% Mounted on
/dev/mapper/vg_test-lv_test ext4      3.4G  6.3M  3.2G   1% /mnt
#查看数据是否还存在
[root@localhost ~]# ll /mnt/
-rw-r--r--. 1 root root     0 Aug 29 15:55 hello_lvm
```

# 六、LVM 缩减

LVM 的缩减需要卸载文件系统–`[root@localhost ~]# umount /mnt/`

**缩减操作：**

```bash
[root@localhost ~]# lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    centos  -wi-ao---- <17.00g                                                    
  swap    centos  -wi-ao----   2.00g                                                    
  lv_test vg_test -wi-a-----  <3.44g                                                    
[root@localhost ~]# resize2fs /dev/vg_test/lv_test 2G
resize2fs 1.42.9 (28-Dec-2013)
Please run 'e2fsck -f /dev/vg_test/lv_test' first.
#需要进行文件系统检测
[root@localhost ~]# e2fsck -f /dev/vg_test/lv_test 
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vg_test/lv_test: 12/220864 files (8.3% non-contiguous), 23666/901120 blocks
[root@localhost ~]# resize2fs /dev/vg_test/lv_test 2G
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/vg_test/lv_test to 524288 (4k) blocks.
The filesystem on /dev/vg_test/lv_test is now 524288 blocks long.

[root@localhost ~]# lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert                                                 
  lv_test vg_test -wi-a-----  <3.44g                                                    
[root@localhost ~]# lvreduce -l 150 /dev/vg_test/lv_test 
  WARNING: Reducing active logical volume to 2.34 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce vg_test/lv_test? [y/n]: y
  Size of logical volume vg_test/lv_test changed from <3.44 GiB (220 extents) to 2.34 GiB (150 extents).
  Logical volume vg_test/lv_test successfully resized.
[root@localhost ~]# lvs
  LV      VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    centos  -wi-ao---- <17.00g                                                    
  swap    centos  -wi-ao----   2.00g                                                    
  lv_test vg_test -wi-a-----   2.34g                                                    
[root@localhost ~]# mount /dev/vg_test/lv_test /mnt
[root@localhost ~]# df -Th
Filesystem                  Type      Size  Used Avail Use% Mounted on
devtmpfs                    devtmpfs  908M     0  908M   0% /dev
tmpfs                       tmpfs     919M     0  919M   0% /dev/shm
tmpfs                       tmpfs     919M  8.6M  911M   1% /run
tmpfs                       tmpfs     919M     0  919M   0% /sys/fs/cgroup
/dev/mapper/centos-root     xfs        17G  2.4G   15G  14% /
/dev/sda1                   xfs      1014M  211M  804M  21% /boot
tmpfs                       tmpfs     184M     0  184M   0% /run/user/0
/dev/mapper/vg_test-lv_test ext4      2.0G  4.7M  1.9G   1% /mnt
#查看测试问价
[root@localhost ~]# ll /mnt/
total 16
-rw-r--r--. 1 root root     0 Aug 29 15:55 hello_lvm
```

# 七、删除LVM

要彻底的来移除LVM的话，需要把创建的步骤反过来操作。

1. 卸载 文件系统
2. 删除lv
3. 删除vg
4. 删除pv

```bash
[root@localhost ~]# umount /mnt
[root@localhost ~]# lvremove /dev/vg_test/lv_test 
Do you really want to remove active logical volume vg_test/lv_test? [y/n]: y
  Logical volume "lv_test" successfully removed
[root@localhost ~]# vgremove /dev/vg_test
  Volume group "vg_test" successfully removed
[root@localhost ~]# pvremove /dev/sdb /dev/sdc
  Labels on physical volume "/dev/sdb" successfully wiped.
  Labels on physical volume "/dev/sdc" successfully wiped.
```

# 八、快照卷

TODO...