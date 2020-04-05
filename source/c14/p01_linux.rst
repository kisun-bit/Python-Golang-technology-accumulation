14.1 Linux
=======================




Linux注册系统服务
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

**系统服务**

简单而言，系统提供某些功能就需要启动一些服务进程，而这些服务进程又是系统主进程的守护进程，所以也有把\ *service*\ 称之为\ *daemon*\ 的说法。\ 
对于\ *Linux*\ 系统而言，将所有服务的启动脚本放置于\ */etc/init.d/*\ 目录下，作为系统启动后的顶层(inti)执行进程，这些启动脚本基本由\ *bash shell script*\ 语句构成。服务启动、关闭、重启、观察状态可以通过下述方式来处理：

 * 启动：\ */etc/init.d/servicename start*\
 * 停止：\ */etc/init.d/servicename stop*\
 * 重启：\ */etc/int.d/servicename restart*\
 * 状态：\ */etc/init.d/servicename status*\



监控信息接口- *linux* 查询网卡控制器名称
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

**对于非链路聚合的网络设备(enXXX)而言：**

.. code-block:: bash

    # -------------------------------------------------------------------------------------- 
    # 1. 例如我们先通过 【ip addr】查询ip地址信息
    [root@localhost ~]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host 
        valid_lft forever preferred_lft forever
    2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
        link/ether 00:50:56:95:ff:8b brd ff:ff:ff:ff:ff:ff
        inet 172.16.4.126/16 brd 172.16.255.255 scope global ens160
        valid_lft forever preferred_lft forever
        inet6 fe80::250:56ff:fe95:ff8b/64 scope link 
        valid_lft forever preferred_lft forever
    3: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:50:56:95:b0:fe brd ff:ff:ff:ff:ff:ff
        inet 192.168.4.126/16 brd 192.168.255.255 scope global ens32
        valid_lft forever preferred_lft forever
        inet6 fe80::250:56ff:fe95:b0fe/64 scope link 
        valid_lft forever preferred_lft forever
    4: takeoverbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN qlen 1000
        link/ether 6e:62:25:73:1b:04 brd ff:ff:ff:ff:ff:ff
        inet 172.29.16.2/16 brd 172.29.255.255 scope global takeoverbr0
        valid_lft forever preferred_lft forever
        inet6 fe80::6c62:25ff:fe73:1b04/64 scope link 
        valid_lft forever preferred_lft forever
    5: bond0: <BROADCAST,MULTICAST,MASTER> mtu 1500 qdisc noop state DOWN qlen 1000
        link/ether 72:fc:63:f1:e7:6a brd ff:ff:ff:ff:f

    # -------------------------------------------------------------------------------------- 
    # 2. 【ip route | grep 172.16.4.126 | awk -F '[ \t*]' '{print $3}'】 查找到ip为172.16.4.126网络设备名
    [root@localhost ~]# ip route | grep 172.16.4.126 | awk -F '[ \t*]' '{print $3}'
    ens160

    # -------------------------------------------------------------------------------------- 
    # 3. 【ethtool -i ens160 | grep 'bus-info'】 查找到ens160网络设备的PCI插口号
    [root@localhost ~]# ethtool -i  ens160 | grep 'bus-info'
    bus-info: 0000:03:00.0

    # --------------------------------------------------------------------------------------
    # 4. 【lspci -v | grep '03:00.0'】
    [root@localhost ~]# lspci -v | grep 03:00.0
    03:00.0 Ethernet controller: VMware VMXNET3 Ethernet Controller (rev 01)  # 控制器名称

..


**bond链路聚合的网络设备(bondXX)而言：**

.. code-block:: bash

    # -------------------------------------------------------------------------------------- 
    # 1. 【ip a】 查找聚合性网络设备
    [root@node1744 ~]# ip a
    1: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether 04:7d:7b:3a:ea:62 brd ff:ff:ff:ff:ff:ff
    inet 172.16.17.44/16 brd 172.16.255.255 scope global bond0
       valid_lft forever preferred_lft forever
    inet 172.16.15.44/16 brd 172.16.255.255 scope global secondary bond0:0
       valid_lft forever preferred_lft forever
    inet6 fe80::67d:7bff:fe3a:ea62/64 scope link 
       valid_lft forever preferred_lft forever

    # -------------------------------------------------------------------------------------- 
    # 2. 【cat /proc/net/bonding/bond0 | grep 'Slave Interface'】  以bond0为例
    [root@node1744 ~]# cat /proc/net/bonding/bond0 | grep 'Slave Interface'
    Slave Interface: enp6s0f0
    Slave Interface: enp6s0f1

    # 后续同上(非聚合性网络设备)

..


Linux链路聚合 bond，team 及网桥的搭建 
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>