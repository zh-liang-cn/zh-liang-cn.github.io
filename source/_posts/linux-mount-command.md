---
layout: post
title: Linux mount命令详解
date: 2014-05-23 07:10:23
category: Linux
excerpt: <i>mount</i>命令在我心目中一直是个高大上的命令：首先，经常看见长度很长的<i>mount</i>命令；其次，<i>mount</i>是直接配置硬件的命令，敲命令的时候总是难免担心把机器搞挂了。因此，我一直以来在Mac OS和Ubuntu中都依靠操作系统自动检查插入的磁盘，就像在Windows里面一样，简直弱爆了。今天就抽空把整理一下这方面的知识。
---

_mount_命令在我心目中一直是个高大上的命令：首先，经常看见长度很长的_mount_命令；其次，是直接配置硬件的命令，敲命令的时候总是难免担心把机器搞挂了。因此，我一直以来在Mac OS和Ubuntu中都依靠操作系统自动检查插入的磁盘，就像在Windows里面一样，简直弱爆了。今天就抽空把整理一下这方面的知识。

## mount命令结构

先举个例子：

```bash
mount -o remount,rw -t yaffs2 /dev/block/mtdblock3 /system
```

这个例子里面基本分为四部分，也是我们最常用的四部分：

* `-o remount,rw`，指定选项
* `-t yaffs2`，指定挂载磁盘上的文件系统的类型，这里的_yaffs2_是_Android_的文件系统类型
* `/dev/block/mtdblock3`，指定磁盘的路径
* `/system`，指定挂载点

也就是这样：

	mount -o <选项> -t <文件系统类型> <磁盘路径> <挂载点>

需要注意的是：

* `-t`一般情况下不用自定，操作系统会自动识别
* 磁盘路径查看方法：Linux: `fdisk -l`；Mac：`diskutil list`

> 关于_fdisk_和_diskutil_，请参考：[What is the equivalent of the Linux command “sudo fdisk -l” in MacOS?][1]

## mount命令的-o选项

这个命令相对比较复杂的地方可能就是这里了：首先，选项可以有很多个，用逗号分隔；其次，针对不同的文件系统，可能有一些特殊的选项，可以参考[man mount][2]，里面用很长的篇幅说明了_mount options for xx_。

我们记住一下几个常用的就好了：

* 修改已经_mount_的文件系统的属性：_remount_
* 控制读写权限的：_rw_与_ro_
* 控制执行权限的：_exec_与_noexec_
* 控制执行权限的：_suid_与_nosuid_。允许和禁止set-user-identifier 或set-group-identifier位起作用
* 控制挂载和卸载权限的：_user_与_nouser_，_users_与_nousers_。前者是允许或禁止先前挂载该文件系统的用户挂载和卸载，后者是允许所有的用户挂载和卸载
* 控制文件系统的读写方式：_sync_与_async_
* 防止用户读写某些系统关键数据：_dev_与_nodev_，如果需要深入了解，请阅读：[Understanding mount option nodev and its use with USB flash drives][4]
* 如果打开了文件，是否更新文件的访问时间：_atime_与_noatime_
* 如果文件的访问时间比当前时间还早，是否更新访问时间：_relatime_与_norelatime_
* _defaults_：默认选项，包括：_rw_，_suid_，_dev_，_exec_，_auto_，_nouser_，_async_以及_relatime_

> 关于_set-user-identifier_详见：[What is SUID and how to set SUID in Linux/Unix?][3]，这篇文章中有几个例子，想必很容易理解了。

## 挂载远程文件系统

Linux和Linux之间文件共享，据我目前所知有两种方法：

1. 一种是基于NFS，参考：[How to Setup NFS (Network File System) on RHEL/CentOS/Fedora and Debian/Ubuntu][5]
2. 另一种是基于SSH来挂在远程文件系统，称为sshfs，参考：[How to Mount a Remote Folder using SSH on Ubuntu][6]

链接中已经给出了具体的方法，此处不赘述，命令格式完全符合本文中提到的结构和参数，除了sshfs是使用_sshfs_命令，而不是_mount_命令。

    mount -t nfs 192.168.0.100:/nfsshare /mnt/nfsshare
    sshfs share@192.168.0.100:/sshshare /mnt/sshshare

## /etc/fstab

另外，_mount_还有一个参数是_mount -a_，这就不得不提一下Linux中的_/etc/fstab_文件和_/etc/mtab_文件。前者是系统启动时会自动挂载的文件系统的信息，后者是系统当前已挂载的文件系统信息。你会发现这个文件里面的配置基本和_mount_命令的参数一致。

	$ cat /etc/fstab
	# <file system> <mount point>   <type>  <options>       <dump>  <pass>
	proc            /proc           proc    nodev,noexec,nosuid 0       0
	...

	$ cat /etc/mtab
	proc /proc proc rw,noexec,nosuid,nodev 0 0
	...

不一样的地方是后面两个字段:

1. _&lt;dump&gt;_告诉_dump_命令是否需要备份这个文件系统，上面例子中因为_proc_是一个[伪文件系统，用于通过内核来访问进程信息][7]，因此不用备份，这个字段是_0_
2. _&lt;pass&gt;_告诉_[fsck][8]_命令是否需要在检查该文件系统，如果需要开机检查该文件系统，可以将这个字段设置为_1_

回到_mount -a_，一句话，此命令就是读取_/etc/fstab_，挂载里面的文件系统，一般Linux系统启动的时候会执行这个命令。

## Linux挂载Windows共享

一般情况下，可以通过下面这个命令来将Windows的共享挂载到Linux的文件系统中：
	
	mount -t cifs -o username=administrator,password=admin //192.168.0.1/share/ /mnt/windows

_[cifs][9]_是微软实现的一种在Windows主机之间进行网络共享的协议。如果你需要在开机的时候自动挂载此文件系统，同样可以相应的配置写到_/etc/fstab_文件中。

## umount

顾名思义，就是卸载文件系统。这个命令更加简单，参数可以是挂载点，也可以是磁盘名称，即下面两种方法均可：

	mount /dev/hda1 /mydoc
	umount /dev/hda1
	umount /mydoc

## 参考

* [mount(8) - Linux man page][2]
* [linux mount挂载设备（u盘,光盘,iso等 ）使用说明][10]

[1]: http://superuser.com/questions/671725/what-is-the-equivalent-of-the-linux-command-sudo-fdisk-l-in-macos
[2]: http://linux.die.net/man/8/mount
[3]: http://www.linuxnix.com/2011/12/suid-set-suid-linuxunix.html
[4]: http://superuser.com/questions/538550/understanding-mount-option-nodev-and-its-use-with-usb-flash-drives
[5]: http://www.tecmint.com/how-to-setup-nfs-server-in-linux/
[6]: http://www.howtogeek.com/howto/ubuntu/how-to-mount-a-remote-folder-using-ssh-on-ubuntu/
[7]: http://baike.baidu.com/view/6096934.htm
[8]: http://baike.baidu.com/view/1757895.htm
[9]: http://baike.baidu.com/view/1034390.htm
[10]: http://www.cnblogs.com/chengmo/archive/2010/10/13/1850515.html
