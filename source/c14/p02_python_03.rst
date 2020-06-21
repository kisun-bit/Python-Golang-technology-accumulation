
[python]动态加载目标模块
===========================

**需求描述**

在Go语言中， 通过 * go get http://githubXXXXXXXXX/xxxxxx/xxxx * 下载远端项目至本地，本地即能够 * import xxxx * 来完成项目或工程的导入。
那么在Python中， 如何自己实现一个类似的导包器呢？并支持远程导包和本地(跨环境)导包

**构造包环境**

这里我们创建了一个名为pkg_test的Python包，包下一级目录中包含了wrapper_1.py、wrapper_2.py和inner_pkg，其中inner_pkg为pkg_test下的一个子包，并包含了inner.py模块

:: 

    pkg_test
	  │
	  │ wrapper_1.py
	  │ wrapper_2.py
	  │ __init__.py
	  │
	  └─inner_pkg
		  inner.py
		  __init__.py

pkg_test包下各个模块的内容

pkg_test/wrapper_1.py

.. code-block:: python 

    # !/user/sbin/python3                                                                               
    # -*- coding:utf-8 -*-                                                                              
                                                                                                        
    import sys                                                                                          
    import inspect                                                                                      
                                                                                                        
                                                                                                        
    def wrapper_1():                                                                                    
        func_name = inspect.stack()[0].function if hasattr(inspect.stack()[0], 'function') else '未定义'   
        print("I'm `{}`. full_path:{}".format(func_name, sys.modules[__name__]))                        
                                                                                                        
                                                                                                        
    if __name__ == '__main__':                                                                          
        wrapper_1()  
..

pkg_test/wrapper_2.py

.. code-block:: python
    
    # !/user/sbin/python3                                                                               
    # -*- coding:utf-8 -*-                                                                              
                                                                                                        
    import sys                                                                                          
    import inspect                                                                                      
                                                                                                        
                                                                                                        
    def wrapper_2():                                                                                    
        func_name = inspect.stack()[0].function if hasattr(inspect.stack()[0], 'function') else '未定义'   
        print("I'm `{}`. full_path:{}".format(func_name, sys.modules[__name__]))                        
                                                                                                        
                                                                                                        
    if __name__ == '__main__':                                                                          
        wrapper_2()  
..		

pkg_test/inner_pkg/inner.py

.. code-block:: python
    
    # !/user/sbin/python3
    # -*- coding:utf-8 -*-

    import sys
    import inspect


    def inner_func():
        func_name = inspect.stack()[0].function if hasattr(inspect.stack()[0], 'function') else '未定义'
        print("I'm `{}`. full_path:{}".format(func_name, sys.modules[__name__]))


    if __name__ == '__main__':
        inner_func()
..		
		