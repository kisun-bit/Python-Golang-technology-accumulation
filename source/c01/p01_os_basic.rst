
1.1 操作系统-linux
===================================

:特别说明: 该部分内容大多归纳自<<鸟哥的linux私房菜基础学习篇(第四版)>>

基础及重点
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

**基础**：

* 计算机的定义：接收用户的输入指令与数据，经过 \ **CPU** 的处理后，以产生或存储成有用的信息；
* 计算机的五大组成单元：\ **输入单元**、\ **输出单元**、\ **控制单元**、\ **算术逻辑单元**、\ **存储单元**，其中CPU拥有控制、算术逻辑两大单元，而存储单元又包含了主存(内存)和辅存(外存)；
* 数据的流入\流出内存依赖于CPU发出的控制命令，而CPU实际要处理的数据则完全调自主存(内存)；
* \ **RISC** (精简指令集计算机)和 \ **CISC** (复杂指令集计算机)是当前CPU的两种架构，它们的区别在于不同的CPU设计理念和方法；
* 现在的绝大部分计算机已将北桥的内存控制芯片集成到了CPU内，而CPU与主存、显示适配器通信的总线通常称为\ **系统总线**。南桥就是所谓的\ **输入输出(IO)总线**，主要与硬盘(外存)、USB、网卡等接口通信；
* CPU每次能够处理的数据量称为\ **字组大小**，字组大小依据CPU的设计有32位、64位。

**主机与磁盘**

* 在 Linux 系统中，每个装置都被当成一个文件来对待，每个装置都会有装置文件名。
* 磁盘装置文件名通常分为两种，实际 SATA/USB 装置文件名为/dev/sd[a-p]，而虚拟机的装置可能为 /dev/vd[a-p]
* 磁盘的第一个扇区主要记录了两个重要的信息，分别是：(1)主要启动记录区(Master Boot Record, MBR)：可以安装开机管理程序的地方，有 446 bytes；(2)分区表(partition table)：记录整颗硬盘分区的状态，有 64 bytes；磁盘的 MBR 分区方式中，主要与延伸分区最多可以有四个，逻辑分区的装置文件名号码，一定由 5 号开始；
* 使用MBR分区表分区时，物理分区数(主分区)+扩展分区数最多不能超过四个；
* 扩展分区最多是1个，扩展分区不能够直接使用，必须在其上划分逻辑分区后才能够使用，逻辑分区的数量可以是0个或多个；
* linux规定：逻辑分区只能建立在扩展分区上，不能建立在主分区上。主分区编号为1-4，逻辑分区为5，6......
* 如果磁盘容量大于 2TB 以上时，系统会自动使用 GPT 分区方式来处理磁盘分区。
* GPT 分区已经没有延伸与逻辑分区槽的概念，你可以想象成所有的分区都是主分区！ 某些操作系统要使用 GPT 分区时，必须要搭配 UEFI 的新型 BIOS 格式才可安装使用。
* 开机的流程由：BIOS-->MBR-->boot loader-->核心文件； boot loader 的功能主要有：提供选单、加载核心、转交控制权给其他 loader
* boot loader 可以安装的地点有两个，分别是 MBR 与 boot sector
* Linux 操作系统的文件使用目录树系统，与磁盘的对应需要有『挂载』的动作才行；

.. code-block:: bash

	# 使用fdisk管理分区

	[root@localhost dev]# ls -l /dev/sd*  # 查看磁盘
	brw-rw---- 1 root disk 8,  0 Mar 30 17:42 /dev/sda
	brw-rw---- 1 root disk 8,  1 Mar 30 17:42 /dev/sda1
	brw-rw---- 1 root disk 8,  2 Mar 30 17:42 /dev/sda2
	brw-rw---- 1 root disk 8,  3 Mar 30 17:42 /dev/sda3
	brw-rw---- 1 root disk 8, 16 Mar 30 17:42 /dev/sdb
	brw-rw---- 1 root disk 8, 17 Mar 30 17:42 /dev/sdb1
	brw-rw---- 1 root disk 8, 32 Mar 30 17:42 /dev/sdc
	brw-rw---- 1 root disk 8, 33 Mar 30 17:42 /dev/sdc1
	brw-rw---- 1 root disk 8, 48 Mar 30 17:42 /dev/sdd
	brw-rw---- 1 root disk 8, 49 Mar 30 17:42 /dev/sdd1
	brw-rw---- 1 root disk 8, 64 Mar 30 17:42 /dev/sde
	brw-rw---- 1 root disk 8, 65 Mar 30 17:42 /dev/sde1
	brw-rw---- 1 root disk 8, 80 Mar 30 17:42 /dev/sdf
	[root@localhost dev]# fdisk /dev/sdf   # 管理 /dev/sdf 磁盘分区
	Welcome to fdisk (util-linux 2.23.2).

	Changes will remain in memory only, until you decide to write them.
	Be careful before using the write command.

	Device does not contain a recognized partition table
	Building a new DOS disklabel with disk identifier 0xb3b9500d.

	Command (m for help): m
	Command action
	   a   toggle a bootable flag
	   b   edit bsd disklabel
	   c   toggle the dos compatibility flag
	   d   delete a partition
	   g   create a new empty GPT partition table
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

	Command (m for help): n
	Partition type:
	   p   primary (0 primary, 0 extended, 4 free)
	   e   extended
	Select (default p): p
	Partition number (1-4, default 1): 
	First sector (2048-2147483647, default 2048): 
	Using default value 2048
	Last sector, +sectors or +size{K,M,G} (2048-2147483647, default 2147483647): 
	Using default value 2147483647
	Partition 1 of type Linux and of size 1024 GiB is set

	Command (m for help): p

	Disk /dev/sdf: 1099.5 GB, 1099511627776 bytes, 2147483648 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk label type: dos
	Disk identifier: 0xb3b9500d

	   Device Boot      Start         End      Blocks   Id  System
	/dev/sdf1            2048  2147483647  1073740800   83  Linux

	Command (m for help): w
	The partition table has been altered!

	Calling ioctl() to re-read partition table.
	Syncing disks.
	[root@localhost dev]# ls -l /dev/sd*
	brw-rw---- 1 root disk 8,  0 Mar 30 17:42 /dev/sda
	brw-rw---- 1 root disk 8,  1 Mar 30 17:42 /dev/sda1
	brw-rw---- 1 root disk 8,  2 Mar 30 17:42 /dev/sda2
	brw-rw---- 1 root disk 8,  3 Mar 30 17:42 /dev/sda3
	brw-rw---- 1 root disk 8, 16 Mar 30 17:42 /dev/sdb
	brw-rw---- 1 root disk 8, 17 Mar 30 17:42 /dev/sdb1
	brw-rw---- 1 root disk 8, 32 Mar 30 17:42 /dev/sdc
	brw-rw---- 1 root disk 8, 33 Mar 30 17:42 /dev/sdc1
	brw-rw---- 1 root disk 8, 48 Mar 30 17:42 /dev/sdd
	brw-rw---- 1 root disk 8, 49 Mar 30 17:42 /dev/sdd1
	brw-rw---- 1 root disk 8, 64 Mar 30 17:42 /dev/sde
	brw-rw---- 1 root disk 8, 65 Mar 30 17:42 /dev/sde1
	brw-rw---- 1 root disk 8, 80 Apr  8 10:47 /dev/sdf
	brw-rw---- 1 root disk 8, 81 Apr  8 10:47 /dev/sdf1
	[root@localhost dev]# mkfs.xfs /dev/sdf1    # 现在的 CentOS 7 默认使用大容量效能较佳的 xfs 当预设文件系统了
	meta-data=/dev/sdf1              isize=512    agcount=4, agsize=67108800 blks
			 =                       sectsz=512   attr=2, projid32bit=1
			 =                       crc=1        finobt=0, sparse=0
	data     =                       bsize=4096   blocks=268435200, imaxpct=25
			 =                       sunit=0      swidth=0 blks
	naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
	log      =internal log           bsize=4096   blocks=131071, version=2
			 =                       sectsz=512   sunit=0 blks, lazy-count=1
	realtime =none                   extsz=4096   blocks=0, rtextents=0
	[root@localhost dev]# mkdir /xfs_kisun
	[root@localhost dev]# mount /dev/sdf1 /xfs_kisun/
	[root@localhost dev]# echo "/dev/sdf1/xfs_kisun xfs defaults 0 0" >> /etc/fstab
	
..

----------------

linux 文件、目录、磁盘
>>>>>>>>>>>>>>>>>>>>>>>>>>

