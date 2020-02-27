
14.1 Linux
=======================




Linux注册系统服务
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

系统服务
::::::::::::::::::

简单而言，系统提供某些功能就需要启动一些服务进程，而这些服务进程又是系统主进程的守护进程，所以也有把\ *service*\ 称之为\ *daemon*\ 的说法。\ 
对于\ *Linux*\ 系统而言，将所有服务的启动脚本放置于\ */etc/init.d/*\ 目录下，这些启动脚本基本由\ *bash shell script*\ 语句构成。服务启动、关闭、重启、观察状态可以通过下述方式来处理：

 * 启动：\ */etc/init.d/servicename start*\
 * 停止：\ */etc/init.d/servicename stop*\
 * 重启：\ */etc/int.d/servicename restart*\
 * 状态：\ */etc/init.d/servicename status*\

