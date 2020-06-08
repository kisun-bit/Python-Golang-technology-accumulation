3.2 《Go程序设计语言-读书笔记》-命名规则
===============================================

在 *Go* 语言中，函数名、变量名、类型名、语句标号和包名等所有的命名，
与 *python* 一样，都遵循一个简单的命名规则：其必须以字母、下划线、数字组成，但起首必须为
字母或下划线开头，且名称区分大小写

在习惯上，*Go* 语言推崇驼峰式命名(缩略词避免使用大小写混合的写法)，而非 *Python* 中的蛇式命名
例如 *HTMLParser* 、*htmlParser*、*ParserHTML* 是合适的，但不会是 *parserHtml*。此外，
通常情况下，我们对一个变量命名应该尽量简短，视作用域的差别选择性的命名。例如在一个循环体的作用域中，经常可见 *i* 类似的命名，
而非类似 *theBooksLoopIndex* 来表示，对于全局作用域的变量名，其命名规则除了简短之外，清晰明了也是尤为重要的。

说完上述规则，我们来看看命名的雷区，即命名时不要和 *Go* 关键字起冲突哦

语法级别的关键字
>>>>>>>>>>>>>>>>>>>

.. code-block:: bash

	break 		//退出当前循环或者switch语句等
	continue 	//跳过本次循环
	return 		//返回
	default 	//选择结构默认项（switch、select）
	switch		//选择结构
	case 		//选择结构标签
	fallthrough //用于标明执行完当前 case 语句之后按顺序执行下一个case语句
	if 		    //选择结构
	else 		//选择结构
	goto 		//跳转语句
	select		//channel
	struct		//定义结构体
	var		    //定义变量
	type		//定义类型
	map 		//map类型
	chan 		//定义channel
	const 		//常量
	for 		//循环
	range 		//从引用类型中遍历元素
	func 		//定义函数
	interface	//定义接口
	defer 		//延迟执行内容，可用于最后清理资源等
	go 		    //并发执行
	package 	//包
	import 		//导入包
	
..

其他关键字(类型相关)
>>>>>>>>>>>>>>>>>>>>>>>>>>

.. code-block:: bash
	
	内建常量: true false iota nil

	内建类型: int int8 int16 int32 int64
			  uint uint8 uint16 uint32 uint64 uintptr
			  float32 float64 complex128 complex64
			  bool byte rune string error

	内建函数: make len cap new append copy close delete
			  complex real imag
			  panic recover
			  
..

这里我们可以先对其关键字集合有一个初步的印象，后面的教程中会慢慢提及到它们的具体应用场景，Let's Go!
