---
layout: post
title: 关于 Linux 分区的小事
tags: [Linux]
index_img: https://th.wallhaven.cc/small/4y/4y1kkd.jpg
banner_img: https://w.wallhaven.cc/full/4y/wallhaven-4y1kkd.jpg
categories: [Linux]
date: 2020-06-09 15:32:54
---

# 一、分区

> 分区是为了在逻辑上将某些柱面隔开形成边界。它是以柱面为单位来划分的，但是从CentOS 7开始，是按照扇区进行划分的。
>
> 在磁盘数据量非常大的情况下，划分分区的好处是扫描块位图等更快速：不用再扫描整块磁盘的块位图，只需扫描对应分区的块位图。

## 1.1 分区方法（MBR、GPT）

### 1.1.1 MBR

**Master/Main Boot Record**

- 446 bytes: BootLoader
- 64 bytes: 
  - 16 bytes: 标识一个分区
- 2 bytes：[Magic Number](https://ecarry.cc/2020/09/02/linux_super_inode_block/#%E4%B8%80%E3%80%81%E4%BB%8E%E7%89%A9%E7%90%86%E7%A3%81%E7%9B%98%E5%88%B0%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)
  - 标记 MBR 是否有效

MBR格式的磁盘中，会维护磁盘第一个扇区——MBR扇区，在该扇区中第446字节之后的64字节是分区表，每个分区占用16字节，所以限制了一块磁盘最多只能有4个主分区(Primary,P)，如果多于4个区，只能将主分区少于4个，通过建立扩展分区(Extend,E)，然后在扩展分区建立逻辑分区(Logical,L)的方式来突破4个分区的限制，逻辑分区的数量不限制。

在Linux中，MBR格式的磁盘主分区号从1-4，扩展分区号从2-4，逻辑分区号从5开始。

**例如**，一块盘想分成6个分区，可以：

1P+5L：sda1+sda5+sda6+sda7+sda8+sda9

2P+4L：sda1+sda2+sda5+sda6+sda7+sda8

3P+3L：sda1+sda2+sda3+sda5+sda6+sda7

### 1.1.2 GPT

GPT格式突破了MBR的限制，它不再限制只能存储4个分区表条目，而是使用了类似MBR扩展分区表条目的格式，它允许有128个主分区，这也使得它可以对超过2TB的磁盘进行分区。

## 1.2 MBR 和 GPT 分区表信息

在MBR格式分区表中，MBR扇区占用512个字节，前446个字节是主引导记录，即boot loader。中间64字节记录着分区表信息，每个主分区信息占用16字节，因此最多只能有4个主分区，最后2个字节是有效标识位。如果使用扩展分区，则扩展分区对应的16字节记录的是指向扩展分区中扩展分区表的指针。

![](/img/linux_part/linux_part_1.png)

在MBR磁盘上，分区和启动信息是保存在一起的，如果这部分数据被覆盖或破坏，只能重建MBR。而GPT在整个磁盘上保存多个这部分信息的副本，因此它更为健壮，并可以恢复被破坏的这部分信息。GPT还为这些信息保存了循环冗余校验码(CRC)以保证其完整和正确，如果数据被破坏，GPT会发现这些破坏，并从磁盘上的其他地方进行恢复。

下面是GPT格式的分区表信息，大致约占17个字节。

![](/img/linux_part/linux_part_2.png)

EFI部分可以分为4个区域：EFI信息区(GPT头)、分区表、GPT分区区域和备份区域。

- EFI信息区(GPT头)：起始于磁盘的LBA1，通常也只占用这个单一扇区。其作用是定义分区表的位置和大小。GPT头还包含头和分区表的校验和，这样就可以及时发现错误。
- 分区表：分区表区域包含分区表项。这个区域由GPT头定义，一般占用磁盘LBA2～LBA33扇区，每扇区可存储4个主分区的分区信息，所以共能分128个主分区。分区表中的每个分区项由起始地址、结束地址、类型值、名字、属性标志、GUID值组成。分区表建立后，128位的GUID对系统来说是唯一的。
- GPT分区：最大的区域，由分配给分区的扇区组成。这个区域的起始和结束地址由GPT头定义。
- 备份区：备份区域位于磁盘的尾部，包含GPT头和分区表的备份。它占用GPT结束扇区和EFI结束扇区之间的33个扇区。其中最后一个扇区用来备份1号扇区的EFI信息，其余的32个扇区用来备份LBA2～LBA33扇区的分区表。

## 1.3 添加磁盘

正常情况下，添加磁盘后需要重启系统才能被内核识别，在 /dev/ 下才有对应的设备号，使用 fdisk -l 才会显示出来。但是有时候不方便重启，所以下面介绍一种磁盘热插拔方式。

### 1.3.1 Linux上磁盘热插拔

获取 SCSI 设备信息：

```
[root@localhost ~]# lsscsi
[1:0:0:0]    cd/dvd  QEMU     QEMU DVD-ROM     2.5+  /dev/sr0 
[2:0:0:0]    disk    WD       Elements SE 25FE 1021  /dev/sdb 
[2:0:0:1]    enclosu WD       SES Device       1021  -        
[3:0:0:0]    disk    QEMU     QEMU HARDDISK    2.5+  /dev/sda 
```

有些操作系统没有 lsscsi 命令，则可以使用下面的方法获取 scsi 设备信息：

```
[root@localhost ~]# ll /sys/bus/scsi/drivers/sd/
总用量 0
lrwxrwxrwx. 1 root root    0 8月   9 15:54 2:0:0:0 -> ../../../../devices/pci0000:00/0000:00:1e.0/0000:01:1b.0/usb3/3-1/3-1:1.0/host2/target2:0:0/2:0:0:0
lrwxrwxrwx. 1 root root    0 8月   9 15:54 3:0:0:0 -> ../../../../devices/pci0000:00/0000:00:05.0/virtio1/host3/target3:0:0/3:0:0:0
--w-------. 1 root root 4096 8月   9 15:54 bind
--w-------. 1 root root 4096 7月  20 18:39 uevent
--w-------. 1 root root 4096 8月   9 15:54 unbind
[root@localhost ~]# ll /sys/bus/scsi/drivers/sd/2\:0\:0\:0/block/
总用量 0
drwxr-xr-x. 8 root root 0 7月  20 18:39 sdb
```

然后查看 /proc/scsi/scsi 文件，获取对应scsi设备的详细信息：

```
[root@localhost ~]# cat /proc/scsi/scsi 
Attached devices:
Host: scsi1 Channel: 00 Id: 00 Lun: 00
  Vendor: QEMU     Model: QEMU DVD-ROM     Rev: 2.5+
  Type:   CD-ROM                           ANSI  SCSI revision: 05
Host: scsi3 Channel: 00 Id: 00 Lun: 00				<-----/dev/sda
  Vendor: QEMU     Model: QEMU HARDDISK    Rev: 2.5+
  Type:   Direct-Access                    ANSI  SCSI revision: 05
Host: scsi2 Channel: 00 Id: 00 Lun: 00				<-----/dev/sdb
  Vendor: WD       Model: Elements SE 25FE Rev: 1021
  Type:   Direct-Access                    ANSI  SCSI revision: 06
Host: scsi2 Channel: 00 Id: 00 Lun: 01				
  Vendor: WD       Model: SES Device       Rev: 1021
  Type:   Enclosure                        ANSI  SCSI revision: 06
```

在此处，有两块**直连**(**Direct-Access**)的 scsi 磁盘，一块通过光驱cd-rom连接的光盘。我们只考虑 scsi 磁盘，所以这两块磁盘在 scsi 中的定位符为2:0:0:0和3:0:0:0。**如果继续插入一块盘，那么新盘在scsi中的定位符可能为4:0:0:0**，这个数值串非常重要。

#### 热插

在向计算机中插入一块磁盘后，内核因为识别不了它所以不会产生任何事件通知，因此在 /sys 目录中不会产生任何文件，任何工具也就读取不了它。重启系统肯定是可以解决的，但是Linux支持热插。

热插新盘的方式是向 /proc/scsi/scsi 中写入新 scsi 设备的信息。方式如下：

`echo "scsi add-single-device a b c d" >/proc/scsi/scsi`

其中：

   a == hostadapter id (first one being 0)

   b == SCSI channel on hostadapter (first one being 0)

   c == ID

   d == LUN (first one being 0)

例如上面的例子，应该添加如下信息：

`[root@localhost ~]# echo "scsi add-single-device 4:0:0:0" >/proc/scsi/scsi`

重新扫描 scsi 总线也可以实现热插的功能，查看主机 SCSI 总线号：

```
[root@localhost ~]# ls /sys/class/scsi_host/
host0  host1  host2  host3
#重新扫描scsi总线以热插拔方式添加新设备。
[root@localhost ~]# echo "- - -" > /sys/class/scsi_host/host0/scan
[root@localhost ~]# echo "- - -" > /sys/class/scsi_host/host1/scan
[root@localhost ~]# echo "- - -" > /sys/class/scsi_host/host2/scan
[root@localhost ~]# echo "- - -" > /sys/class/scsi_host/host3/scan
[root@localhost ~]# fdisk -l      # 再查看就有了
```

如果 scsi_host 目录系很多hostN目录，则使用循环来完成：

```
[root@localhost ~]# ls /sys/class/scsi_host/
host0   host11  host14  host17  host2   host22  host25  host28  host30  host4  host7
host1   host12  host15  host18  host20  host23  host26  host29  host31  host5  host8
host10  host13  host16  host19  host21  host24  host27  host3   host32  host6  host9

[root@localhost ~]# for i in /sys/class/scsi_host/host*/scan;do echo "- - -" >$i;done
```

热插之后，fdisk -l等命令就可以识别到该磁盘了。



#### 热拔

热拔磁盘的方式是在 /proc/scsi/scsi 中移除对应 scsi 设备的信息。方式如下：

`echo "scsi remove-single-device a b c d" >/proc/scsi/scsi`

例如删除2:0:0:0这块磁盘：

`[root@localhost ~]# echo "scsi remove-single-device 2 0 0 0" >/proc/scsi/scsi`

因为要删除的设备已经存在，/sys中已经有它完整的信息，所以也从其自身设备上进行删除。

首先查看scsi设备信息:

```
[root@localhost ~]# lsscsi
[1:0:0:0]    cd/dvd  QEMU     QEMU DVD-ROM     2.5+  /dev/sr0 
[2:0:0:0]    disk    WD       Elements SE 25FE 1021  /dev/sdb 
[2:0:0:1]    enclosu WD       SES Device       1021  -        
[3:0:0:0]    disk    QEMU     QEMU HARDDISK    2.5+  /dev/sda 
```

 例如要删除 /dev/sdb，即2:0:0:0。先看看它的文件信息：

```
[root@localhost ~]# ls /sys/bus/scsi/drivers/sd/2\:0\:0\:0/
block           driver                        evt_mode_parameter_change_reported  ioerr_cnt      queue_depth  scsi_generic  uevent
bsg             eh_timeout                    evt_soft_threshold_reached          iorequest_cnt  queue_type   scsi_level    unpriv_sgio
delete          evt_capacity_change_reported  generic                             max_sectors    rescan       state         vendor
device_blocked  evt_inquiry_change_reported   inquiry                             modalias       rev          subsystem     vpd_pg80
device_busy     evt_lun_change_reported       iocounterbits                       model          scsi_device  timeout       vpd_pg83
dh_state        evt_media_change              iodone_cnt                          power          scsi_disk    type          wwid
```

在其中有3个文件：delete、rescan和state。其中state记录了该设备是否正在运行中。而delete和rescan文件则用于删除和重新扫描该设备。

例如，删除该设备，即热拔：

`[root@localhost ~]# echo 1 > /sys/bus/scsi/drivers/sd/2\:0\:0\:0/delete`



## 1.4 使用 fdisk 分区工具

fdisk 工具用来分 MBR 磁盘上的区。要分 GPT 磁盘上的区，可以使用 gdisk。parted工具对这两种格式的磁盘分区都支持。

如果一个存储设备已经分过区，那么它可能是mbr格式的，也可能是gpt格式的，如果已经是mbr格式的，则只能继续使用fdisk进行分区，如果已经是gpt格式的，则只能使用gdisk进行分区。当然，无论什么格式的都可以使用parted进行分区，只不过也只能划分和已存在分区格式一样的分区，因为无论何种格式的分区，它的分区表和分区标识是已经固定的。

使用fdisk分区，它只能实现MBR格式的分区：

```
[root@localhost ~]# fdisk /dev/sda
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：m
命令操作
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table	#新版本 fdisk 开始支持 GPT 分区了
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

命令(输入 m 获取帮助)：p

磁盘 /dev/sda：21.5 GB, 21474836480 字节，41943040 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000c7ad0

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM

命令(输入 m 获取帮助)：n			#新建分区
Partition type:
   p   primary (2 primary, 0 extended, 2 free)	#总共可建4个主分区，现有2个，还剩可建两个
   e   extended		#扩展分区
Select (default p): p
分区号 (3,4，默认 3)：
#建立完分区注意保存
Command (m for help): w  
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

上面的 fdisk 操作全部是在内存中执行的，必须保存生效。保存后，内核还未识别该分区，可以查看 /proc/partition 目录下存在的文件，这些文件是能被内核识别的分区。运行**partprobe**或**partx**命令重新读取分区表让内核识别新的分区，内核识别后才可以格式化。而且分区结束时按w保存分区表有时候会失败，提示重启，这时候运行partprobe命令可以代替重启就生效。

```
[root@localhost ~]# partprobe /dev/sdb	#也可指定在 /dev/sdb 上重加载分区表
```

## 1.5 使用 gdisk 分区工具

使用 gdsik 需要安装这个工具包：

```
[root@localhost ~]# yum -y install gdisk
```

分区的时候直接带上设备即可。以下是对新硬盘划分gpt分区的过程：

```
[root@localhost ~]# gdisk /dev/sdb
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): ?
b       back up GPT data to a file
c       change a partition's name
d       delete a partition                               # 删除分区
i       show detailed information on a partition         # 列出分区详细信息
l       list known partition types                       # 列出所以已知的分区类型
n       add a new partition                              # 添加新分区
o       create a new empty GUID partition table (GPT)    # 创建一个新的空的guid分区表
p       print the partition table                        # 输出分区表信息
q       quit without saving changes                      # 退出gdisk工具
r       recovery and transformation options (experts only) 
s       sort partitions                                
t       change a partition's type code                   # 修改分区类型
v       verify disk
w       write table to disk and exit                     # 将分区信息写入到磁盘
x       extra functionality (experts only)             
?       print this menu
Command (? for help): n   	#添加一个新分区
Partition number (1-128, default 1):
First sector (34-41943006, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-41943006, default = 41943006) or {+-}size{KMGTP}: +10G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'
 
Command (? for help): p
Disk /dev/sdb: 41943040 sectors, 20.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): F8AE925F-515F-4807-92ED-4109D0827191
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 41943006
Partitions will be aligned on 2048-sector boundaries
Total free space is 20971453 sectors (10.0 GiB)
 
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        20973567   10.0 GiB    8300  Linux filesystem

Command (? for help): i   # 查看分区详细信息
Using 1
Partition GUID code: 0FC63DAF-8483-4772-8E79-3D69D8477DE4 (Linux filesystem)
Partition unique GUID: B2452103-4F32-4B60-AEF7-4BA42B7BF089
First sector: 2048 (at 1024.0 KiB)
Last sector: 20973567 (at 10.0 GiB)
Partition size: 20971520 sectors (10.0 GiB)
Attribute flags: 0000000000000000
Partition name: 'Linux filesystem'

Command (? for help): w 		#保存分区表到磁盘

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.

#执行partprobe重新读取分区表信息
[root@localhost ~]# partprobe /dev/sdb
```

## 1.6 使用 parted 分区

parted 支持 mbr 格式和 gpt 格式的磁盘分区。它的强大在于可以一步到位而不需要不断的交互式输入(也可以交互式)。

parted分区工具是实时的，所以每一步操作都是直接写入磁盘而不是写进内存，它不像 fdisk/gdisk 还需要 w 命令将内存中的结果保存到磁盘中。

```
[root@localhost ~]# parted /dev/sdc
GNU Parted 2.1
Using /dev/sdc
Welcome to GNU Parted! Type 'help' to view a list of commands.
 
(parted) help                                                             
  align-check TYPE N                      check partition N for TYPE(min|opt) alignment
  check NUMBER                            do a simple check on the file system(centos 7上已删除该功能)
  cp [FROM-DEVICE] FROM-NUMBER TO-NUMBER  copy file system to another partition(centos 7上已删除该功能)
  help [COMMAND]                          print general help, or help on COMMAND
  mklabel,mktable LABEL-TYPE              create a new disklabel (partition table)
  mkfs NUMBER FS-TYPE                     make a FS-TYPE file system on partition NUMBER (centos 7上已删除改该功能) 
  mkpart PART-TYPE [FS-TYPE] START END    make a partition
  mkpartfs PART-TYPE FS-TYPE START END    make a partition with a file system(centos 7上已删除该功能)   
  move NUMBER START END                   move partition NUMBER(centos 7上已删除该功能) 
  name NUMBER NAME                        name partition NUMBER as NAME
  print [devices|free|list,all|NUMBER]    display the partition table,available devices,free space, all found partitions,or a particular partition
  quit                                    exit program
  rescue START END                        rescue a lost partition near START and END
  resize NUMBER START END                 resize partition NUMBER and its file system(修改分区大小(centos 7上已删除该功能))
  rm NUMBER                               delete partition NUMBER (删除分区)             
  select DEVICE                           choose the device to edit (重选磁盘进入parted状态)  
  set NUMBER FLAG STATE                   change the FLAG on partition NUMBER(设置分区状态，如将其off或on)  
  toggle [NUMBER [FLAG]]                  toggle the state of FLAG on partition NUMBER(修改文件系统类型，如swap、lvm)  
  unit UNIT                               set the default unit to UNIT(修改默认单位，kB/MB/GB等)
  version                                 display the version number and copyright information of GNU Parted
```

常用的命令是mklabel、rm、print、mkpart、help、quit

parted分区的前提是磁盘已经有分区表(partition table)或磁盘标签(disk label)，否则将显示"unrecognised disk label"，这是和fdisk/gdisk不同的地方，所以需要先使用mklabel创建标签或分区表，最常见的标签(分区表)为"msdos"和"gpt"，其中msdos分区就是MBR格式的分区表，也就是会有主分区、扩展分区和逻辑分区的概念和限制。

下面使用 parted 对 /dev/sdc 创建 msdos 的新分区：

```
[root@localhost ~]# parted /dev/sdc
GNU Parted 2.1
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel             # 创建磁盘分区标签(分区表类型)                                               
New disk label type? msdos   # 选择msdos即MBR类型    
                             # 上面的两步也可以直接一步进行：(parted) mklabel msdos      
(parted) mkpart              # 开始进行分区     
Partition type?  primary/extended? p     # 创建主分区
File system type?  [ext2]? ext4          # 创建ext4文件系统
                                         # (注意，这里虽然指明了文件系统，但没有任何意义，后面还是需要手动格式化并选择文件系统类型)
Start? 1                                 # 分区开始位置，默认是M为单位，表示从1M开始，也可直接指定1G这种方式
End? 1024                                # 分区结束位置，1024-1=1023M

(parted) p                   # print，查看分区信息
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  1024MB  1023MB  primary

# 可以一步完成一个命令中的多个动作
(parted) mkpart p ext4 1026M 4096M       # 可以一步完成，也可以一步完成到任何位置，然后继续交互下一步
                                         # 可能会提示分区未对齐"Warning: The resulting partition is not properly aligned for best performance."，忽略它
(parted) mkpart e 4098 -1    # 创建扩展分区，注意创建扩展分区时不指定文件系统类型；-1表示剩余的全部分配给该分区

(parted) p                                                               
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos 

Number  Start   End     Size    Type      File system  Flags
 1      1049kB  1024MB  1023MB  primary
 2      1026MB  4096MB  3070MB  primary
 3      4098MB  21.5GB  17.4GB  extended               lba

(parted) mkpart l ext4 4099 8194     # 创建逻辑分区，指定ext4
(parted) mkpart l ext4 8195 -1       # 继续创建逻辑分区
(parted) p
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type      File system  Flags
 1      1049kB  1024MB  1023MB  primary
 2      1026MB  4096MB  3070MB  primary
 3      4098MB  21.5GB  17.4GB  extended               lba
 5      4099MB  8194MB  4095MB  logical
 6      8195MB  21.5GB  13.3GB  logical

(parted) rm 5    # 删除5号分区
(parted) p
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type      File system  Flags
 1      1049kB  1024MB  1023MB  primary
 2      1026MB  4096MB  3070MB  primary
 3      4098MB  21.5GB  17.4GB  extended               lba
 5      8195MB  21.5GB  13.3GB  logical

(parted) quit                                    # 退出parted工具
Information: You may need to update /etc/fstab.  # 提示你要更新/etc/fstab中的配置，说明该工具是可以在线分区的
```

<font color=red>注：虽然parted工具中指定了文件系统，但是并没有意义，它仍需要手动进行格式化并指定分区类型。</font>

#  二、格式化分区

## 2.1 mkfs 工具

mkfs命令通常用于在设备硬件分区上创建linux文件系统。mkfs 命令支持建立多种 Linux 文件系统，如 ext系列，xfs等等。实际上mkfs是支持多种文件系统构建命令`mkfs. TYPE`的前部分，mkfs命令通常执行的时候也是调用`mkfs. TYPE`来执行，如mkfs.ext2，mkfs.vfs，mkfs.vfat等等。

```
[root@localhost ~]# mkfs
mkfs         mkfs.btrfs   mkfs.cramfs  mkfs.ext2    mkfs.ext3    mkfs.ext4    mkfs.minix   mkfs.xfs 
```



## 2.2 mke2fs 工具

mkfs.ext2、mkfs.ext3、mkfs.ext4 或 mkfs -t extX 其实都是在调用mke2fs工具。

该工具创建文件系统时，会从 /etc/mke2fs.conf 配置中读取默认的配置项：

```
mke2fs [ -c ] [ -b block-size ] [ -f fragment-size ] [ -g blocks-per-group ] [ -G number-of-groups ] 
       [ -i bytes-per-inode ] [ -I inode-size ] [ -j ] [ -N number-of-inodes ] [ -m reserved-blocks-percentage ] 
       [ -q ] [ -r fs-revision-level ] [ -v ] [ -L volume-label ] [ -S ] [ -t fs-type ] device [ blocks-count ]

选项说明：
-t fs-type         ：指定要创建的文件系统类型(ext2,ext3 ext4)，若不指定，则从/etc/mke2fs.conf中获取默认的文件系统类型。
-b block-size      ：指定每个block的大小，有效值有1024、2048和4096，单位是字节。
-I inode-size      ：指定inode大小，单位为字节。必须为2的幂次方，且大于等于128字节。值越大，说明inode的集合体inode table占用越多的空
                     间，这不仅会挤占文件系统中的可用空间，还会降低性能，因为要扫描inode table需要消耗更多时间，但是在linux kernel 2.6.10
                     之后，由于使用inode存储了很多扩展的额外属性，所以128字节已经不够用了，因此ext4默认的inode size已经变为256，尽管
                     inode大小增大了，但因为使用inode存储扩展属性带来的性能提升远高于inode size变大导致的负面影响，所以仍建议使用256字
                     节的inode。
-i bytes-per-inode ：指定每多少个字节就为其分配一个inode号。值越大，说明一个文件系统中分配的inode号越少，更适用于存储大量大文件，值越
                     小，inode号越多，更适用于存储大量小文件。该值不能小于一个block的大小，因为这样会造成inode多余。
                     注意，创建文件系统后该值就不能再改变了。
-c                 ：创建文件系统前先检查设备是否有bad blocks。
-f fragment-size   ：指定fragments的大小，单位字节。
-g blocks-per-group：指定每个块组中的block数量。不建议修改此项。
-G number-of-groups：该选项用于ext4文件系统(严格地说是启用了flex_bg特性)，指定虚拟块组(即一个extent)中包含的块组个数，必须为2的幂次方。
                     对于ext4文件系统来说，使用extent的功能能极大提升其性能。
-j                 ：创建带有日志功能的文件系统，即ext3。如果要指定关于日志方面的设置，在-j的基础上再使用-J指定，不过一般默认即可，具体可
                     指定的选项看man文档。 
-L new-volume-label：指定卷标名称，名称不得超出16字节。
-m reserved-blocks-percentage：指定文件系统保留block数量的比例，保留一部分block，可以降低物理碎片。默认比例为5%。
-N number-of-inodes ：强制指定该文件系统应该分配多少个inode号，它会覆盖通过计算得出inode数量的结果(根据block大小、数量和每多少字节分配
                      一个inode得出Inode数量)，但是不建议这么做。
-q                  ：安静模式，可用于脚本中
-S                  ：重建superblock和group descriptions。在所有的superblock和备份的superblock都损坏时有用。它会重新初始化superblock和
                      group descriptions，但不会改变inode table、bmap和imap(若真的改变，该分区数据就全丢了，还不如重新格式化)。在重建
                      superblock后，应该执行e2fsck来保证文件系统的一致性。但要注意，应该完全正确地指定block的大小，其改选项并不能完全保
                      证数据不丢失。
-v                  ：输出详细执行过程
```

**所以，有可能用到的选项也就 "-t" 指定文件系统类型，"-b" 指定 block 大小，"-I" 指定 inode 大小，"-i" 指定分配 inode 的比例。**

例如：

```
[root@localhost ~]# mke2fs -t ext4 -I 256 /dev/sdb2 -b 4096
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
655360 inodes, 2621440 blocks
131072 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2684354560
80 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Writing inode tables: done                           
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 39 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
```



# 三、查看文件系统状态信息

##  3.1 lsblk(list block devices)

> 用于列出设备及其状态，主要列出非空的存储设备。其实它只会列出/sys/dev/block中的主次设备号文件，且默认只列出非空设备。

```
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0  1.8T  0 disk 
└─sdb1            8:17   0  1.8T  0 part 
sr0              11:0    1  4.5G  0 rom 
```

其中上面的几列意义如下：

NAME：设备名称；

MAJ:MIN：主设备号和此设备号；

RM：是否为可卸载设备，1表示可卸载设备。可卸载设备如光盘、USB等。并非能够umount的就是可卸载的；

SIZE：设备总空间大小；

RO：是否为只读；

TYPE：是磁盘disk，还是分区part，亦或是rom，还有loop设备；

mountpoint：挂载点。

另外常用的一个选项是"-f"，它可以查看到文件系统类型，和文件系统的uuid和挂载点：

```
[root@localhost ~]# lsblk -f
NAME            FSTYPE      LABEL           UUID                                   MOUNTPOINT
sda                                                                                
├─sda1          xfs                         39b6f37f-531f-41c2-a498-dbe216fcfc4b   /boot
└─sda2          LVM2_member                 BVd9cM-wbZz-QBn3-8IOg-A8Xc-NZ5w-u98gYr 
  ├─centos-root xfs                         0a8b095d-743e-494b-a4e3-1dca4211d9bb   /
  └─centos-swap swap                        b83b56ce-37f2-460a-bbd1-8d4da4624493   [SWAP]
sdb                                                                                
└─sdb1          ext4                        2bc92060-a128-4ad5-bfbe-9cfb61dc1870   
sr0             iso9660     CentOS 7 x86_64 2020-04-22-00-54-00-00  
```

<font color=red>注：每个已经格式化的文件系统都有其类型和uuid，而没有格式化的设备(如/dev/sdb3)，将只显示一个Name结果，表示该设备还未进行格式化。</font>

## 3.2 blkid

查看文件系统类型和uuid：

```
[root@localhost ~]# blkid
/dev/sr0: UUID="2020-04-22-00-54-00-00" LABEL="CentOS 7 x86_64" TYPE="iso9660" PTTYPE="dos" 
/dev/sda1: UUID="39b6f37f-531f-41c2-a498-dbe216fcfc4b" TYPE="xfs" 
/dev/sda2: UUID="BVd9cM-wbZz-QBn3-8IOg-A8Xc-NZ5w-u98gYr" TYPE="LVM2_member" 
/dev/mapper/centos-root: UUID="0a8b095d-743e-494b-a4e3-1dca4211d9bb" TYPE="xfs" 
/dev/mapper/centos-swap: UUID="b83b56ce-37f2-460a-bbd1-8d4da4624493" TYPE="swap" 
/dev/sdb1: UUID="2bc92060-a128-4ad5-bfbe-9cfb61dc1870" TYPE="ext4" 
```

## 3.3 du

> du命令用于评估文件的空间占用情况，它会统计每个文件的大小，统计时会递归统计目录中的文件，也就是说，它会遍历整个待统计目录，所以统计速度上可能并不理想。

```
du [OPTION]... [FILE]...
选项说明：
-a, --all：列出目录中所有文件的统计信息，默认只会列出目录中子目录的统计信息，而不列出文件的统计信息
-h, --human-readable：人性化显示大小
-0, --null：以空字符结尾，即"\0"而非换行的"\n"
-S, --separate-dirs：不包含子目录的大小
-s, --summarize：对目录做总的统计，不列出目录内文件的大小信息
-c,--total：对给出的文件或目录做总计。在统计非同一个目录文件大小时非常有用。见下文例子。
-d,--max-depth：指定显示时的目录深度，默认会递归显示所有层次
--max-depth=N：只列出给定层次的目录统计，如果N=0，则等价于"-s"
-x, --one-file-system：忽略不同文件系统上的文件，不对它们进行统计
-X, --exclude-from=FILE：从文件中读取要排除的文件
--exclude=PATTERN：指定要忽略不统计的文件
```

```
[root@localhost ~]# du -sh /etc/
35M     /etc/
```

```
[root@localhost ~]# du -ah /tmp/
0       /tmp/.Test-unix
0       /tmp/.font-unix
0       /tmp/.ICE-unix
0       /tmp/.XIM-unix
0       /tmp/.X11-unix
4.0K    /tmp/oraysl.status
4.0K    /tmp/oraynewph.status
0       /tmp/systemd-private-dd51c630d55b47ac953c22e69a6c1240-mariadb.service-jPbRmD/tmp
0       /tmp/systemd-private-dd51c630d55b47ac953c22e69a6c1240-mariadb.service-jPbRmD
8.0K    /tmp/

```

## 3.4 df

df 用于报告磁盘空间使用率，默认显示的大小是1K大小block数量，也就是以k为单位。

和 du 不同的是，df 是读取每个文件系统的 superblock 信息，所以评估速度非常快。由于是读取 superblock，所以如果目录下挂载了另一个文件系统，是不会将此挂载的文件系统计入目录大小的。**注意，du 和 df 统计的结果是不一样的。**

如果用 df 统计某个文件的空间使用情况，将会转而统计该文件所在文件系统的空间使用情况。

```
df [OPTION]... [FILE]...
选项说明：
-h：人性化转换大小的显示单位
-i：统计inode使用情况而非空间使用情况
-l, --local：只列出本地文件系统的使用情况，不列出网络文件系统信息
-T, --print-type：同时输出文件系统类型
-t, --type=TYPE：只列出给定文件系统的统计信息
-x, --exclude-type=TYPE：指定不显示的文件系统类型的统计信息

[root@localhost ~]# df -hT
文件系统                类型      容量  已用  可用 已用% 挂载点
devtmpfs                devtmpfs  908M     0  908M    0% /dev
tmpfs                   tmpfs     919M     0  919M    0% /dev/shm
tmpfs                   tmpfs     919M   33M  887M    4% /run
tmpfs                   tmpfs     919M     0  919M    0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        17G  2.4G   15G   14% /
/dev/sda1               xfs      1014M  211M  804M   21% /boot
tmpfs                   tmpfs     184M     0  184M    0% /run/user/0

[root@localhost ~]# df -i
文件系统                  Inode 已用(I) 可用(I) 已用(I)% 挂载点
devtmpfs                 232254     405  231849       1% /dev
tmpfs                    235193       1  235192       1% /dev/shm
tmpfs                    235193     542  234651       1% /run
tmpfs                    235193      16  235177       1% /sys/fs/cgroup
/dev/mapper/centos-root 8910848   78419 8832429       1% /
/dev/sda1                524288     339  523949       1% /boot
tmpfs                    235193       1  235192       1% /run/user/0

```

# 四、挂载和卸载文件系统

## 4.1 mount

> mount用来显示挂载信息或者进行文件系统挂载，它的功能及其的强大(强大到离谱)，它不仅支持挂载非常多种文件系统，如ext/xfs/nfs/smbfs/cifs (win上的共享目录)等，还支持共享挂载点、继承挂载点(父子关系)、绑定挂载点、移动挂载点等等功能。在本文只介绍其最简单的挂载功能。

```
mount # 将显示当前已挂载信息
mount [-t 欲挂载文件系统类型 ] [-o 特殊选项] 设备名 挂载目录

选项说明：
-a  将/etc/fstab文件里指定的挂载选项重新挂载一遍。
-t  支持ext2/ext3/ext4/vfat/fat/iso9660(光盘默认格式)。 不用-t时默认会调用blkid来获取文件系统类型。
-n  不把挂载记录写在/etc/mtab文件中，一般挂载会在/proc/mounts中记录下挂载信息，然后同步到/etc/mtab，指定-n表示不同步该挂载信息。
-o  指定挂载特殊选项。下面是两个比较常用的：
    loop  挂载镜像文件，如iso文件
    ro  只读挂载
    rw  读写挂载
    auto  相当于mount -a
    dev 如果挂载的文件系统中有设备访问入口则启用它，使其可以作为设备访问入口
    default rw,suid,dev,exec,auto,nouser,async,and relatime
    async   异步挂载，只写到内存
    sync    同步挂载，通过挂载位置写入对方硬盘
    atime   修改访问时间，每次访问都修改atime会导致性能降低，所以默认是noatime
    noatime 不修改访问时间，高并发时使用这个选项可以减少磁盘IO
    nodiratime  不修改文件夹访问时间，高并发时使用这个选项可以减少磁盘IO
    exec/noexec  挂载后的文件系统里的可执行程序是否可执行，默认是可以执行exec， 优先级高于权限的限定
    remount  重新挂载，此时可以不用指定挂载点。
    suid/nosuid 对挂载的文件系统启用或禁用suid，对于外来设备最好禁用suid
    _netdev 需要网络挂载时默认将停留在挂载界面直到加载网络了。使用_netdev可以忽略网络正常挂载。如NFS开机挂载。
    user  允许普通用户进行挂载该目录，但只允许挂载者进行卸载该目录
    users  允许所有用户挂载和卸载该目录
    nouser  禁止普通用户挂载和卸载该目录，这是默认的，默认情况下一个目录不指定user/users时，将只有root能挂载
```

使用实例：

**(1).挂载CentOS的安装镜像到/mnt。**

```
mount /dev/cdrom /mnt
```

**(2).重新挂载。**

```
mount -t ext4 -o remount /dev/sdb1 /data1
```

**(3).重新挂载文件系统为可读写**

```
mount -t ext4 -o rw remount /dev/sdb1 /data1
```

**(4).挂载windows的共享目录。**

win上共享文件的文件系统是cifs类型，要在Linux上挂载，必须得有mount.cifs命令，如果没有则安装cifs-utils包。

假设win上共享目录的unc路径为\\192.168.100.8\test，共享给的用户名和密码分别为long3:123，要挂在linux上的/mydata目录上。

```
mount.cifs -o username="long3",password="123" //192.168.100.8/test /mydata
```

注意，如果是比较新版本的win10(2017年之后更新的版本)或较新版本的win server，直接mount.cifs会报错：

```
[root@localhost ~]# mount.cifs -o username="long3",password="123" //192.168.100.8/test /mnt         
mount error(112): Host is down
Refer to the mount.cifs(8) manual page (e.g. man mount.cifs)
```

这是因为2017年微软的一个补丁禁用了SMBv1协议，通过smbclient的报告可知：

```
[root@localhost ~]# yum -y install samba-client
[root@localhost ~]# smbclient -L //192.168.100.8
Enter root's password: 
protocol negotiation failed: NT_STATUS_CONNECTION_RESET
```

因此，在mount的时候指定cifs(SMB)的版本号为2.0即可：

```
[root@localhost ~]# mount.cifs -o username="long3",password="123",vers=2.0 //192.168.100.8/test /mnt
```

## 4.2 挂载镜像文件

有时候需要挂载CentOS的镜像文件，在虚拟机中经常是将镜像放入虚拟机的CD/DVD虚拟光驱中，然后在Linux上对/dev/cdrom进行挂载。其实/dev/cdrom是/dev/sr0的一个软链接，/dev/sr0是Linux中的光驱，所以上面的过程相当于是将镜像文件通过虚拟软件的虚拟光驱和linux的光驱连接起来，这样只需要挂载Linux中的光驱就可以了。但是，在非虚拟环境中没有虚拟光驱，而且在Linux中的一个镜像文件难道一定要拷贝到主机上通过虚拟光驱进行连接吗？

mount是一个极其强大的挂载工具，它支持挂载很多种文件类型，其中就支持挂载镜像文件，其实它连挂载目录都支持。

```
mount -o loop CentOS-6.6-x86_64-bin-DVD2.iso /mnt

[root@localhost ~]# lsblk
NAME     MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0      7:0    0   1.2G  0 loop /mnt
sda        8:0    0    20G  0 disk
├─sda1   8:1    0   250M  0 part /boot
├─sda2   8:2    0  17.8G  0 part /
└─sda3   8:3    0     2G  0 part [SWAP]
sr0       11:0    1  1024M  0 rom
```

## 4.3 umount

```
umount 设备名或挂载目录
umount -lf 强制卸载
```

## 4.4 开机自动挂载/etc/fstab

通过将挂载选项写入到 /etc/fstab 中，系统会自动挂载该文件中的配置项。但要注意，该文件在开机的前几个过程中就被读取，所以配置错误很可能会导致开机失败。

```
[root@localhost ~]# cat /etc/fstab 

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

其中最后两列，它们分别表示备份文件系统和开机自检，一般都可以设置为0。

由于能用的备份工具众多，没人会在这里设置备份，所以备份列设置为0。

最后一列是开机自检设置列，开机自检调用的是fsck程序，所有有些ext类文件系统作为"/"时，可能会设置为1，但是fsck是不支持xfs文件系统的，所以对于xfs文件系统而言，该项必须设置为0。

其实无需考虑那么多，直接将这两列设置为0就可以了。

## 4.5 修复错误的/etc/fstab

万一 /etc/fstab 配置错误，导致开机无法加载。这时提示输入 root 密码进入单人维护模式，只不过单人模式下根文件系统是只读的，哪怕是 root 也无法直接修改 /etc/fstab，所以应该将"/"文件系统进行重新挂载。

执行下面的命令，重挂载根分区，并给读写权限，再去修改错误的 fstab 文件记录，再重启。

```
mount -n -o remount,rw /
```

# 五、SWAP 分区

## 5.1 查看哪个分区在充当 swap 分区

```
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0  1.8T  0 disk 
└─sdb1            8:17   0  1.8T  0 part 
sr0              11:0    1  4.5G  0 rom 
[root@localhost ~]# swapon  -s
文件名                          类型            大小    已用    权限
/dev/dm-1                               partition       2097148 0       -2

```

## 5.2 添加 swap 分区

(1).可以新分一个区，在分区时指定其分区的ID号为SWAP类型。

(2).格式化为swap分区：mkswap

```
[root@localhost ~]# mkswap /dev/sdb5
Setting up swapspace version 1, size = 1951096 KiB
no label, UUID=02e5af44-2a16-479d-b689-4e100af6adf5
```

(3).加入swap分区空间(swapon)：

```
[root@localhost ~]# swapon /dev/sdb5  

[root@localhost ~]# free -m
             total       used       free     shared    buffers     cached
Mem:          1861        343       1517          0         16        196
-/+ buffers/cache:        131       1730
Swap:         3953          0       3953
```

(4).取消swap分区空间(swapoff)：

```
[root@xuexi ~]# swapoff /dev/sdb5

[root@xuexi ~]# free -m
             total       used       free     shared    buffers     cached
Mem:          1861        343       1518          0         16        196
-/+ buffers/cache:        130       1731
Swap:            0          0          0
```

(5).开机自动加载swap分区：

修改/etc/fstab，加上一行

```
/dev/sda5    swap    swap    defaults    0    0
```





> 学习于[[骏马金龙](https://www.cnblogs.com/f-ck-need-u/)]