
14.3 Python
=========================

日志记录工具
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

随着项目的单(微)服务化，各个服务间相互通信的场景不断扩大，此时对于主服务而言，日志的管理显得尤为重要。而 *python* \
也提供了 *logging* 库来支持多种日志记录管理方式。笔者最初在做远程 *ssh* 服务时，采用了下述方式做日志对象输出：

.. code-block:: python

    import logging

    def get_logger(logger_name):
        """得到日志对象
        """
        logger = logging.getLogger(logger_name)
        logger.setLevel(logging.DEBUG)
        formatter = logging.Formatter('[ %(asctime)s ] - %(levelname)s - %(message)s')
        if not logger.handlers:
            file_log_handler = logging.FileHandler(settings.VIEW_LOG_PATH, encoding=settings.DEFAULT_CHARSET)
            file_log_handler.setLevel(logging.DEBUG)
            file_log_handler.setFormatter(formatter)
            logger.addHandler(file_log_handler)
        return logger
..


补充:

======== ======
等级     意义
-------- ------
DEBUG    程序运行上下文及关键信息，通常仅仅在DEBUG时使用
INFO     程序正常运行，执行到关键操作时的输出信息
WARNING  程序超出预期运行，但在可控边界内
ERROR    程序发生错误，影响部分功能的继续使用
CRITICAL 程序已几近崩溃，严重错误
======== ======

麻雀虽小，五脏俱全。通过代码可以知道，一个日志对象，其相关关键对象如下：

 * **logger**；对外最上层的日志记录对象，可自定制额外的日志信息，使用看 *logging.info* 源码注释
 * **handler**: 处理程序将日志记录（由记录器创建）发送到适当的目的地（文件或控制台） 
 * **filter**: 筛选器提供了更细粒度的功能，用于确定要输出的日志记录(该对象优先级较低)
 * **formatter**: 格式化程序在最终输出中指定日志记录的格式

因此，对于日志记录配置而言，上述四个对象是紧密相关的。

*logging* 库下的 *config* 提供的诸多配置方式(例如 *Dictionary Schema Details*、*User-defined objects*、*Configuration file format* 等)，也与上述四个关键参数对象紧密相关，这里以项目中常见的 *Configuration file format* 模式说明：

.. code-block:: ini
    
    # log.ini
    # 声明logger，配置logger的handler, 配置handler的formatter
    [loggers]
    keys=root

    [handlers]
    keys=fileHandler

    [formatters]
    keys=simpleFormatter

    [logger_root]
    level=NOTSET
    handlers=fileHandler

    [handler_fileHandler]
    class=logging.handlers.RotatingFileHandler  # 可以进入logging.handlers查找需要的处理器
    args=(r'*.log', 'a', 30*1024*1024, 5)
    level=DEBUG
    formatter=simpleFormatter

    [formatter_simpleFormatter]
    format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
    datefmt=
..


.. code-block:: python
    
    # test.py
    import logging
    import logging.config
    
    current_dir = os.path.abspath(os.path.dirname(__file__))
    log_conf_path = os.path.join(current_dir, 'logging.ini')

    logging.config.fileConfig(log_conf_path)


    def get_logger(name):
        return logging.getLogger(name)

    
    def test_log():
        _l = get_logger(__name__)
        _l.info('attention %s', 'known_issue', exc_info=1)

..

你可能会问，那低优先级的筛选器 *filter* 又是怎样的应用场景呢？

这里我们摘取来自 https://www.programcreek.com/python/example/3364/logging.Filter 上的例子做说明 \ 
*_filter_log* 方法实现了屏蔽来自 *API* 目录运行环境下的日志记录，通过重写 *logging.Filter* 下 *filter* 方法的方式

.. code-block:: python

    import logging

    def _filter_log(self):
        """Disables logging in the discovery API to avoid excessive logging."""

        class _ChildLogFilter(logging.Filter):
        """Filter to eliminate info-level logging when called from this module."""

            def __init__(self, filter_levels=None):
                super(_ChildLogFilter, self).__init__()
                self._filter_levels = filter_levels or set(logging.INFO)
                # Get name without extension to avoid .py vs .pyc issues
                self._my_filename = os.path.splitext(
                    inspect.getmodule(_ChildLogFilter).__file__)[0]

            def filter(self, record):
                if record.levelno not in self._filter_levels:
                    return True
                callerframes = inspect.getouterframes(inspect.currentframe())
                for f in callerframes:
                    if os.path.splitext(f[1])[0] == self._my_filename:
                        return False
                    return True

        googleapiclient.discovery.logger.addFilter(_ChildLogFilter({logging.INFO}))

..




性能分析工具
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

在的算法分析，时间和空间损耗是衡量一个算法的优劣(性能)主要的指标，下面我们分别从这两个方向来分析Python程序性能。

**时间损耗**

cProfile_ 自Python2.5以来的Python解释器(限于cPython)默认的性能分析器

..  _cProfile:  https://docs.python.org/zh-cn/3.6/library/profile.html?highlight=cprofile#module-cProfile

分析报告参数解释：


========= ========
参数      含义
--------- --------
ncalls    表示函数调用的次数；
tottime   表示指定函数的总的运行时间，除开函数中调用子函数的运行时间；
percall   等于 tottime/ncalls；
cumtime   表示该函数及其所有子函数的调用运行的时间，即函数开始调用到返回的时间；
percall   即函数运行一次的平均时间，等于 cumtime/ncalls；
filename  lineno(function)：每个函数调用的具体信息；
========= ========


用法参考：

**1.使用profile.run(command, filename=None, sort=-1)分析**

.. code-block:: python
  
	# 假设我们要分析 foo 函数的性能，代码如下
	import cProfile
	cProfile.run('foo()')
	  
	# 执行完成之后，就会输出如下格式的统计报告
	#        5 function calls in 0.143 CPU seconds
	#
	# Ordered by: standard name
	#
	# ncalls tottime percall cumtime percall filename:lineno(function)
	#  1    0.000    0.000    0.000    0.000 :0(range)
	#  1    0.143    0.143    0.143    0.143 :0(setprofile)
	#  1    0.000    0.000    0.000    0.000 <string>:1(?)
	#  1    0.000    0.000    0.000    0.000 prof1.py:1(foo)
	#  1    0.000    0.000    0.143    0.143 profile:0(foo())
	#  0    0.000             0.000          profile:0(profiler)

..


**2.使用命令行方式分析模块**

/home/xxxxx/demo.py

.. code-block:: python

	import time


	def iter_val(num: int):
        for i in range(num):
			if i == 5:
				time.sleep(5)
			yield i


	def main():
		for val in iter_val(10):
			# print(val, '\n')
			pass


	if __name__ == '__main__':
		main()

..

此时，如果我要对该例程(demo.py)进行时间损耗分析，运行下面的命令

.. code-block:: 
	
	# 直接把分析结果打印到控制台
	# python -m cProfile demo.py
	# 把分析结果保存到文件中
	# python -m cProfile -o result.out demo.py
	# 增加排序方式
	# python -m cProfile -o result.out -s cumulative demo.py
	[root@node3 ~]# python3 -m cProfile -s cumulative demo.py                                                                        
	26 function calls in 5.005 seconds                                                   
																								  
	Ordered by: cumulative time                                                                
																								  
	ncalls  tottime  percall  cumtime  percall filename:lineno(function)                       
		1    0.000    0.000    5.005    5.005 {built-in method exec}                          
		1    0.000    0.000    5.005    5.005 demo.py:1(<module>)                             
		1    0.000    0.000    5.005    5.005 demo.py:11(main)                                
	   11    0.000    0.000    5.005    0.455 demo.py:4(iter_val)                             
		1    5.005    5.005    5.005    5.005 {built-in method sleep}                         
	   10    0.000    0.000    0.000    0.000 {built-in method print}                         
		1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
			
..

所以，通过上述的分析输出，可以看出，该例程的主要时间损耗在 *iter_val* 方法下的time模块内建方法 *sleep* 中

**3.分析某一代码块的性能**

有时项目的代码量十分巨大，针对某一块功能接口的性能分析报告可能出现刷屏的情况，此时cProfile支持将分析报告输出到文件中，实现筛选过滤

.. code-block:: python
	
	import time
	import cProfile


	def main():
		for i in range(3):
			time.sleep(1)

		pr = cProfile.Profile()
		pr.enable()
		time.sleep(2)
		pr.disable()
		pr.dump_stats('./demo.stats')

	if __name__ == '__main__':
		main()
	
..

交互式命令查看

.. code-block:: 

    [root@node3 ~]# python3 -m pstats demo.stats

	Welcome to the profile statistics browser.
	demo.stats%
	demo.stats% strip
	demo.stats% sort time  
	demo.stats% stats 10
	Thu Apr 16 14:20:49 2020    demo.stats

			 2 function calls in 2.002 seconds

	   Ordered by: internal time

	   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
			1    2.002    2.002    2.002    2.002 {built-in method sleep}
			1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}


	##### 在交互式pstats环境中可以使用sort来查看帮助(使用哪一个参数项排序)
..

脚本查看方式::

import pstats
p=pstats.Stats("result")
p.strip_dirs().sort_stats(-1).print_stats()      
p.strip_dirs().sort_stats("name").print_stats()
p.strip_dirs().sort_stats("cumulative").print_stats(3)



