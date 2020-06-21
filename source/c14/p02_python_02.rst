[Python]性能分析工具
=============================

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


脚本查看方式

>>> import pstats
>>> p=pstats.Stats("result")
>>> p.strip_dirs().sort_stats(-1).print_stats()      
>>> p.strip_dirs().sort_stats("name").print_stats()
>>> p.strip_dirs().sort_stats("cumulative").print_stats(3)
