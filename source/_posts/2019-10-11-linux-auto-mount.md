---
layout: post
title: Linux中设备挂载命令
categories: Linux
description: Linux中设备挂载命令
index_img: 
date: 2019-10-11 09:09:09
tags: [Linux]
---
#### 1、查询与自动挂载

#查询系统中已经挂载的设备
```
[root@localhost ~]# mount

```

#依据配置文件/etc/fstab的内容，自动挂载

```
[root@localhost ~]# mount -a

```
#我们打开fstab这个文件看看吧，执行如下命令：

```
[root@localhost ~]# vi /etc/fstab
UUID=c2ca6f57-b15c-43ea-bca0-f239083d8bd2 / ext4 defaults 1 1
UUID=0b23d315-33a7-48a4-bd37-9248e5c44345 /boot ext4 defaults 1 2
UUID=4021be19-2751-4dd2-98cc-383368c39edb swap swap defaults 0 0
#只有这三个是真正的硬盘分区，下面的都是虚拟文件系统或交换分区
tmpfs /dev/shm tmpfs defaults 0 0
devpts /dev/pts devpts gid=5, mode=620 0 0
sysfs /sys sysfe defaults 0 0
proc /proc proc defaults 0 0
```
tmpfs、devpts、sysfs 和 proc 这几行，它们分别是与共享内存、终端窗口、设备信息和内核参数相关联的特殊设备。

可以看到，在 fstab 文件中，每行数据都分为了 6 个字段，它们的含义分别是：

1.  用来挂载每个文件系统的分区设备文件名或 UUID（用于指代设备名）；
2.  挂载点；
3.  文件系统的类型；
4.  各种挂载参数；
5.  指定分区是否被 dump 备份；
6.  指定分区是否被 fsck 检测


#### 2、挂载命令格式

```
[root@localhost ~]# mount [-t 文件系统] [-o 特殊选项] 设备文件名 挂载点

```

**选项：**

\-t 文件系统：加入文件系统类型来指定挂载的类型，可以ext3、ext4、iso9660等文件系统
\-o 特殊选项：可以指定挂载的额外选项
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190603095712319.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDU3NDY5,size_16,color_FFFFFF,t_70)

```
[root@localhost ~]# mount -o remount,noexec/home

```

#重新挂载/boot分区，并使用noexec权限

```
[root@localhost sh]# cd /home
[root@localhost boot]# vi hello.sh
[root@localhost boot]# chmod 755 hello.sh
[root@localhsot boot]# ./hello.sh
[root@localhsot boot]# mount -o remount,exec/home

```

＃记着改回来啊，要不会影响系统启动的

#### 3、挂载光盘

```
[root@localhost ~]# mkdir /mnt/cdrom/

```

#建立挂载点

```
[root@localhost ~]# mount -t iso9660/dev/cdrom /mnt/cdrom/

```

#挂载光盘

```
[root@localhsot ~]# mount /dev/sr0/mnt/cdrom/

```

#### 4、卸载命令

```
[root@localhost ~]# umount 设备文件名或挂载点
[root@localhost ~]# umount/mnt/cdrom

```

#### 5、挂载U盘

```
[root@localhost ~]# fdisk -l

```

#查看U盘设备文件名

```
[root@localhost ~]# mount -t vfat/dev/sdb1/mnt/usb/

```

注意：Linux默认是不知此NTFS文件系统的

# mmcblk0p0和sda1等等的含义

1. mmcblk0p0

     1.1. mmc含义
     
        SD/MMC 卡的设备构造差不多，MMC 应该是 SD 的前身，不过 MMC 当时的设计比 SD 小一半。所以，SD/MMC 的驱动通用，进一步的，Linux 的设备节点就延续了 MMC 的这个名字
     
     1.2. blk含义
     
         blk 是块设备，后面的数字是设备的顺序编号
     
     1.3 p含义
     
         p表示分区，p1 就是第一个分区

2. sda1

    2.1 sdxx
    
        sd是scsi,SATA硬盘设备。
        
    2.2. sda1
    
        a代表是第一块硬盘 1代表是第一个主分区
        
    2.3. sdb1
    
        b代表是第二块硬盘 1代表是第一个主分区