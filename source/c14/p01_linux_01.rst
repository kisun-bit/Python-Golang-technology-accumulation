[linux]注册系统服务
=====================================


简单而言，系统提供某些功能就需要启动一些服务进程，而这些服务进程又是系统主进程的守护进程，所以也有把\ *service*\ 称之为\ *daemon*\ 的说法。\
对于\ *Linux*\ 系统而言，将所有服务的启动脚本放置于\ */etc/init.d/*\ 目录下， \
作为系统启动后的顶层(inti)执行进程，这些启动脚本基本由\ *bash shell script*\ 语句构成。
服务启动、关闭、重启、观察状态可以通过下述方式来管理：

* 启动：\ */etc/init.d/servicename start*\
* 停止：\ */etc/init.d/servicename stop*\
* 重启：\ */etc/int.d/servicename restart*\
* 状态：\ */etc/init.d/servicename status*\

认识service的unit文件(扩展名为.service的文件)，该类型文件存储于下述路径中:

* /etc/systemd/system/*     供系统管理员和用户使用
* /run/systemd/system/*     运行时配置文件
* /usr/lib/systemd/system/*  安装程序使用（如RPM包安装）

服务类型单元支持管理下述系统资源类型：

========= =========
service   守护进程的启动、停止、重启和重载是此类 unit 中最为明显的几个类型。
socket    此类 unit 封装系统和互联网中的一个socket。当下，systemd支持流式, 数据报和连续包的AF\_INET,AF\_INET6,AF\_UNIXsocket 。也支持传统的 FIFOs 传输模式。每一个 socket unit 都有一个相应的服务 unit 。相应的服务在第一个“连接”进入 socket 或 FIFO 时就会启动(例如：nscd.socket 在有新连接后便启动 nscd.service)。
device    此类 unit 封装一个存在于 Linux设备树中的设备。每一个使用 udev 规则标记的设备都将会在 systemd 中作为一个设备 unit 出现。udev 的属性设置可以作为配置设备 unit 依赖关系的配置源。
mount     此类 unit 封装系统结构层次中的一个挂载点。
automount 此类 unit 封装系统结构层次中的一个自挂载点。每一个自挂载 unit 对应一个已挂载的挂载 unit (需要在自挂载目录可以存取的情况下尽早挂载)。
target    此类 unit 为其他 unit 进行逻辑分组。它们本身实际上并不做什么，只是引用其他 unit 而已。这样便可以对 unit 做一个统一的控制。(例如：multi-user.target 相当于在传统使用 SysV 的系统中运行级别5)；bluetooth.target 只有在蓝牙适配器可用的情况下才调用与蓝牙相关的服务，如：bluetooth 守护进程、obex 守护进程等）
snapshot  与 targetunit 相似，快照本身不做什么，唯一的目的就是引用其他 unit 。
========= =========


Unit文件专门用于systemd下控制资源，\
这些资源包括服务(service)、套接字(socket)、设备(device)、挂载点(mount point)、自动挂载点(automount point)、交换文件或分区(a swap file or partition)… \
所有的unit文件都应该配置[Unit]或者[Install]段.由于通用的信息在[Unit]和[Install]中描述，每一个unit应该有一个指定类型段，例如[Service]来对应后台服务类型unit \
以下是一段service unit文件的例子，属于/usr/lib/systemd/NetworkManager.service文件，它描述的是系统中的网络管理服务。


.. code-block::

    [Unit]
    Description=Network Manager
    Documentation=man:NetworkManager(8)
    Wants=network.target
    After=network-pre.target dbus.service
    Before=network.target network.service

    [Service]
    Type=dbus
    BusName=org.freedesktop.NetworkManager
    #ExecReload=/usr/bin/dbus-send --print-reply --system --type=method_call --dest=org.freedesktop.NetworkManager /org/freedesktop/NetworkManager org.freedesktop.NetworkManager.Reload uint32:0
    ExecReload=/bin/kill -HUP $MAINPID
    ExecStart=/usr/sbin/NetworkManager --no-daemon
    Restart=on-failure
    # NM doesn't want systemd to kill its children for it
    KillMode=process
    #CapabilityBoundingSet=CAP_NET_ADMIN CAP_DAC_OVERRIDE CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SETGID CAP_SETUID CAP_SYS_MODULE CAP_AUDIT_WRITE CAP_KILL CAP_SYS_CHROOT

    # ibft settings plugin calls iscsiadm which needs CAP_SYS_ADMIN (rh#1371201)
    CapabilityBoundingSet=CAP_NET_ADMIN CAP_DAC_OVERRIDE CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SETGID CAP_SETUID CAP_SYS_MODULE CAP_AUDIT_WRITE CAP_KILL CAP_SYS_CHROOT CAP_SYS_ADMIN

    ProtectSystem=true
    ProtectHome=read-only

    [Install]
    WantedBy=multi-user.target
    Alias=dbus-org.freedesktop.NetworkManager.service
    Also=NetworkManager-dispatcher.service

..


从/usr/lib/systemd/NetworkManager.service文件内容中可以发现，一个service文件通常由下面三个语段小节

* [Unit]: 记录unit文件的通用信息
* [Service]：记录服务信息
* [Install]: 记录安装信息

-------------------------------------------------------------


**对于[Unit]小节而言，有如下取值字段：**

1. **Description**：

    * 对本service的描述

2. **Before, After**：

    * 定义启动顺序，Before=xxx.service，代表本服务在xxx.service启动之前启动.After=xxx.service,代表本服务在xxx之后启动。
3. **Requires**:

    * 这个单元启动了，那么它“需要”的单元也会被启动; 它“需要”的单元被停止了，它自己也活不了。但是请注意，这个设定并不能控制某单元与它“需要”的单元的启动顺序（启动顺序是另外控制的），即 Systemd 不是先启动 Requires 再启动本单元，而是在本单元被激活时，并行启动两者。于是会产生争分夺秒的问题，如果 Requires 先启动成功，那么皆大欢喜; 如果 Requires 启动得慢，那本单元就会失败（Systemd 没有自动重试）。所以为了系统的健壮性，不建议使用这个标记，而建议使用 Wants 标记。可以使用多个 Requires。
4. **RequiresOverridable**：

    * 跟 Requires 很像。但是如果这条服务是由用户手动启动的，那么 RequiresOverridable 后面的服务即使启动不成功也不报错。跟 Requires 比增加了一定容错性，但是你要确定你的服务是有等待功能的。另外，如果不由用户手动启动而是随系统开机启动，那么依然会有 Requires 面临的问题。
5. **Requisite**：

    * 强势版本的 Requires。要是这里需要的服务启动不成功，那本单元文件不管能不能检测等不能等待都立刻就会失败。
6. **Wants**：

    * 推荐使用。本单元启动了，它“想要”的单元也会被启动。但是启动不成功，对本单元没有影响。
7. **Conflicts**：

    * 一个单元的启动会停止与它“冲突”的单元，反之亦然。

---------------------------------------------

**对于[Service]小节而言，有如下取值字段:**

1. **Type**：

    * simple 默认，这是最简单的服务类型。意思就是说启动的程序就是主体程序，这个程序要是退出那么一切都退出。
    * forking 标准 Unix Daemon 使用的启动方式。启动程序后会调用 fork() 函数，把必要的通信频道都设置好之后父进程退
    * oneshot(未设置 ExecStart= 时的默认值) 不同之处在于该进程必须在 systemd 启动后继单元之前退出。 此种类型通常需要设置 RemainAfterExit= 选项。
    * notify(与"simple"类似) 不同之处在于该进程将会在启动完成之后通过 sd_notify(3) 之类的接口发送一个通知消息。 systemd 将会在启动后继单元之前，首先确保该进程已经成功的发送了这个消息。 如果设置为此类型，那么 NotifyAccess= 将只能设置为"all"或者"main"(默认) 注意，目前 Type=notify 尚不能在 PrivateNetwork=yes 的情况下正常工作
    * dbus(设置了 ExecStart= 与 BusName= 时的默认值) 与"simple"类似，不同之处在于该进程需要在 D-Bus 上获得一个由 BusName= 指定的名称。 systemd 将会在启动后继单元之前，首先确保该进程已经成功的获取了指定的 D-Bus 名称。设置为此类型相当于隐含的依赖于 dbus.socket 单元
    * idle("simple"类似) 与"simple"类似，不同之处在于该进程将会被延迟到所有的操作都完成之后再执行。 这样可以避免控制台上的状态信息与 shell 脚本的输出混杂在一起。

2. **ExecStart**：

    * 服务启动时执行的命令，通常此命令就是服务的主体
    * 如果你服务的类型不是 oneshot，那么它只可以接受一个命令，参数不限。
    * 同行多个命令要用分号隔开，多行用 \ 跨行。

3. **ExecStartPre, ExecStartPost**：

    * ExecStart执行前后所调用的命令
4. **ExecStop**：

    * 定义停止服务时所执行的命令，定义服务退出前所做的处理。如果没有指定，使用systemctl stop xxx命令时，服务将立即被终结而不做处理
5. **Restart**：

    * 当服务进程正常退出、异常退出、被杀死、超时的时候，是否重新启动该服务。 "服务进程"是指 ExecStart=, ExecStartPre=, ExecStartPost=, ExecStop=, ExecStopPost=, ExecReload= 中设置的进程。 当进程是由于 systemd 的正常操作(例如 systemctl stop|restart)而被停止时，该服务不会被重新启动。 "超时"可以是看门狗的"keep-alive ping"超时，也可以是 systemctl start|reload|stop 操作超时。
    * 该选项可以取下列值之一：no, on-success, on-failure, on-abnormal, on-watchdog, on-abort, always "no"(默认值)表示不会被重启。"always"表示会被无条件的重启。 "on-success"表示仅在服务进程正常退出时重启，所谓"正常退出"是指： 退出码为"0"，或者进程收到 SIGHUP, SIGINT, SIGTERM, SIGPIPE 信号并且退出码符合 SuccessExitStatus= 的设置。 "on-failure"表示仅在服务进程异常退出时重启，所谓"异常退出"是指： 退出码不为"0"，或者进程被强制杀死(包括"core dump"以及收到 SIGHUP, SIGINT, SIGTERM, SIGPIPE 之外的其他信号)， 或者进程由于看门狗或者 systemd 的操作超时而被杀死。 对于 on-failure, on-abnormal, on-abort, on-watchdog 的分别适用于哪种异常退出，见下表：

        .. image:: ../_static/c14-p01-01.png

    * 注意如下两个例外情况： (1) RestartPreventExitStatus= 中列出的退出码或者信号永远不会导致该服务被重启。 (2) RestartForceExitStatus= 中列出的退出码或者信号将会无条件的导致该服务被重启。 对于需要长期持续运行的守护进程，推荐设为"on-failure"以增强可用性。 对于自身可以自主选择何时退出的服务，推荐设为"on-abnormal"。
6. **SuccessExitStatus**：

    * 额外定义附加的进程"正常退出"状态。可以设为一系列以空格分隔的数字退出码或者信号名称，例如： SuccessExitStatus=1 2 8 SIGKILL 表示当进程的退出码是 1, 2, 8 或被 SIGKILL 信号终止时，都可以视为"正常退出"。 注意，退出码"0"以及 SIGHUP, SIGINT, SIGTERM, SIGPIPE 信号是标准的"正常退出"，不需要在此特别定义。 注意，如果进程拥有自定义的信号处理器，并且在收到信号后通过调用 _exit(2) 退出，那么有关信号的信息就会丢失。 在这种情况下，进程必须自己完成清理工作并使用相同的信号自杀。参见 Proper handling of SIGINT/SIGQUIT — How to be a proper program 如果多次使用此选项，那么最终的结果将是多个列表的合集。如果将此项设为空，那么先前设置的列表将被清空。
7. **RestartPreventExitStatus**:

    * 可以设为一系列以空格分隔的数字退出码或者信号名称，当进程的退出码或收到的信号与此处的设置匹配时， 该服务将无条件的禁止重新启动(无视 Restart= 的设置)。例如： RestartPreventExitStatus=1 6 SIGABRT 表示退出码 1, 2, 8 与 SIGKILL 信号将不会导致该服务被重启。 默认值为空，表示完全遵守 Restart= 的设置。
    * 如果多次使用此选项，那么最终的结果将是多个列表的合集。如果将此项设为空，那么先前设置的列表将被清空。 RestartForceExitStatus= 可以设为一系列以空格分隔的数字退出码或者信号名称，当进程的退出码或收到的信号与此处的设置匹配时， 该服务将无条件的被重新启动(无视 Restart= 的设置)。 默认值为空，表示完全遵守 Restart= 的设置。 如果多次使用此选项，那么最终的结果将是多个列表的合集。如果将此项设为空，那么先前设置的列表将被清空。
8. **PermissionsStartOnly**:

    * 设为 true 表示所有与权限相关的执行选项(例如 User= 之类的选项，参见 systemd.exec(5) 手册)仅对 ExecStart= 中的程序有效， 而对 ExecStartPre=, ExecStartPost=, ExecReload=, ExecStop=, ExecStopPost= 中的程序无效。 默认值 false 表示所有与权限相关的执行选项对所有 Exec*= 系列选项中的程序都有效。
9. **RootDirectoryStartOnly**:

    * 设为 true 表示根目录(参见 systemd.exec(5) 中的 RootDirectory= 选项)仅对 ExecStart= 中的程序有效， 而对 ExecStartPre=, ExecStartPost=, ExecReload=, ExecStop=, ExecStopPost= 中的程序无效。 默认值 false 表示根目录对所有 Exec*= 系列选项中的程序都有效。
#. **NonBlocking**:

    * 是否为所有通过socket激活传递的文件描述符设置非阻塞标记(O_NONBLOCK)。默认值为 false 设为 true 表示所有大于2的文件描述符(也就是 stdin, stdout, stderr 之外的文件描述符)都将被设置为非阻塞模式。 该选项仅在与 socket 单元(systemd.socket(5))联用的时候才有意义。
#. **NotifyAccess**:

    * 设置通过sd_notify(3)访问服务状态通知socket的访问模式。 可以设为：none(默认值), main, all 之一。 "none"表示不更新任何守护进程的状态，忽略所有的状态更新消息。 "main"表示仅接受主进程的状态更新消息。 "all"表示接受该服务cgroup内的所有进程的状态更新消息。 当设置了 Type=notify 或 WatchdogSec= 的时候，此选项应该被设为"main"或"all"，如果未设置，那么隐含为"main"。
#. **Sockets**:

    * 设置一个socket单元的名称，表示该服务在启动时应当从它继承socket文件描述符。通常并不需要明确设置此选项， 因为所有与该服务同名(不算后缀)的socket单元的socket文件描述符，都会被自动的传递给派生进程。 注意：(1)同一个socket文件描述符可以被传递给多个不同的进程(服务)。 (2)当socket上有流量进入时，被启动的可能是另一个不同于该服务的其他服务。 换句话说就是：socket单元中的 Service= 所指向的服务单元中的 Sockets= 设置未必要反向指回去。
    * 如果多次使用此选项，那么最终的结果将是多个socket单元的合集。如果将此项设为空，那么先前设置的socket单元的列表将被清空。
#. **StartLimitInterval, StartLimitBurst**:

    * 限制该服务的启动频率。默认值是每10秒内不得超过5次(StartLimitInterval=10s StartLimitBurst=5)。
    * StartLimitInterval= 的默认值等于systemd配置文件中 DefaultStartLimitInterval= 的值，"0"表示取消启动频率限制。
    * StartLimitBurst= 的默认值等于systemd配置文件中 DefaultStartLimitBurst= 的值。
    * 虽然这两个选项经常与 Restart= 一起使用，但是它们不只限制 Restart= 罗辑所导致的重启，而是限制所有类型的启动(包括手动启动)。 注意，当 Restart=逻辑所导致的重启超出了启动频率限制之后，Restart= 逻辑将会被禁用(也就是不会在下一个时间段内再次尝试重启)， 然而，如果该单元随后又被手动重启，那么 Restart= 罗辑将被再次激活。 注意，"systemctl reset-failed ..."命令会清除该服务的重启次数计数器，这通常用于在手动启动之前清除启动限制。
#. **StartLimitAction**:

    * 设置到达启动频率限制后触发什么动作。 可设为 none(默认值), reboot, reboot-force, reboot-immediate, poweroff, poweroff-force, poweroff-immediate 之一。
    * "none"表示除了禁止再次启动之外，不触发任何动作。
    * "reboot"表示触发常规的系统重启的动作，相当于执行"systemctl reboot"命令。
    * "reboot-force"表示触发系统的强制重启动作(强制杀死所有进程但不会造成文件系统不一致)，相当于执行"systemctl reboot -f"命令。
    * "reboot-immediate"表示立即调用内核的reboot(2)函数，可能会造成文件系统的数据丢失。
    * poweroff, poweroff-force, poweroff-immediate 与对应的"reboot*"项含义类似，不同之处仅仅在于是关机而不是重启。
#. **FailureAction**:

    * 设置当该服务进入失败(failed)状态时所触发的动作。取值范围与默认值都与 StartLimitAction= 完全相同。
#. **RebootArgument**:

    * 设置reboot(2)系统调用的可选参数，仅用于 StartLimitAction= 与 FailureAction= 的重启动作。 其作用与"systemctl reboot [arg]"命令中的可选参数[arg]完全相同。
#. **FileDescriptorStoreMax**:

    * 允许在 systemd 中最多为该服务存储多少个使用sd_pid_notify_with_fds(3)的"FDSTORE=1"消息的文件描述符，默认值为"0"(不存储)。 用于实现重启该服务而不会丢失其状态(前提是该服务将各种状态序列化之后保存在 /run 中，同时将文件描述符交给 systemd 暂存)。 所有被 systemd 暂存的文件描述符都将在该服务重启之后交还给该服务的主进程。 所有被 systemd 暂存的文件描述符都将在遇到如下两种情况时被自动关闭： (1)收到 POLLHUP 或 POLLERR 信号；(2)该服务被彻底停止，并且没有任何剩余的任务队列
#. **USBFunctionDescriptors**:

    * 设为一个包含 USB FunctionFS 描述符的文件路径，以实现 USB gadget 支持。 仅与配置了 ListenUSBFunction= 的 socket 单元一起使用。该文件的内容将被写入 ep0 文件。 USBFunctionStrings= 设为一个包含 USB FunctionFS 字符串的文件路径。 其行为与上面的 USBFunctionDescriptors= 类似。 参见 systemd.exec(5) 与 systemd.kill(5) 手册页，以获取更多其他选项。

-------------------------------------------------

**对于[Install]小节而言，有如下取值字段:**

* **WantedBy**：何种情况下，服务被启用。

    eg：WantedBy=multi-user.target（多用户环境下启用）

* **Alias**：别名

--------------------------------------------------


关于自动依赖的说明：

* 设置了 Type=dbus 的服务会自动添加 Requires=dbus.socket 与 After=dbus.socket 依赖
* 基于套接字激活的服务会自动添加对与其相关的 .socket 单元的 After= 依赖。

除非明确设置了 DefaultDependencies=false，否则 service 单元都自动隐含如下依赖：

* Requires=sysinit.target After=sysinit.target After=basic.target Conflicts=shutdown.target Before=shutdown.target 这样可以确保普通的服务单元

    1. 在基础系统启动完毕之后才开始启动，
    2. 在关闭系统之前先被干净的停止。 只有那些需要在系统启动的早期就必须启动的服务，以及那些必须在关机动作的结尾才能停止的服务才需要设置 DefaultDependencies=false 。 systemd.exec(5) 与 systemd.resource-control(5) 中的某些资源限制选项也会自动隐含的添加一些其他的依赖关系。

* 从同一个模版实例化出来的所有服务单元(单元名称中带有 "@" 字符)， 默认全部属于与模版同名的同一个 slice 单元。 该同名 slice 一般在系统关机时，与所有模版实例一起停止。 如果你不希望像上面这样，那么可以在模版单元中明确设置 DefaultDependencies=no ， 并且：要么在该模版文件中明确定义特定的 slice 单元(同样也要明确设置 DefaultDependencies=no)、 要么在该模版文件中明确设置 Slice=system.slice (或其他合适的 slice)。



参考 https://blog.csdn.net/fu_wayne/article/details/38018825

参考 https://blog.csdn.net/yuesichiu/article/details/51485147


