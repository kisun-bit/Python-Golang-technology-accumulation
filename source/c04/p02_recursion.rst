
4.2 递归
=========================

在计算机程序中，描述迭代的一种方法是使用循环，而另外一种完全不同的迭代方法就是递归
函数调用是以栈的方式展开和结束的，这也就意味着，过多层级的递归调用会造成内存泄漏的情况发生。
但从另外一种角度而言，递归是一种迭代的设计思想，是非常值得我们学习的一种算法。

下面我们分别以下述4个例子举例说明递归的用法：
	* 阶乘的递归实现
	* 斐波拉契数列的递归实现
	* 二分查找的递归实现
	
Python实现
	
.. code-block:: python

	#!/usr/bin/env python
	# -*- coding:utf-8 -*-

	# Copyright: Copyright (c) 2020
	# Created on 2020-6-8
	# Author: Kisun
	# Version 1.0
	# Title: Python地递归调用


	def factorial(n: int):
		"""阶乘函数的递归实现

		remark:
								| 1  (n = 0)
			  n的阶乘描述为 n! = <
								| n * (n -1) (n >= 1)
		"""

		return 1 if n == 0 else n * factorial(n - 1)


	def fib_bad(n: int):
		"""计算第n个斐波拉契数列值（低效方式）
		"""

		return 1 if n < 2 else (fib_bad(n - 1) + fib_bad(n - 2))


	def fib_good(n: int) -> tuple:
		"""计算第n个斐波拉契数列值（低效方式）
		"""

		if n == 1:
			return (n, 0)
		else:
			(a, b) = fib_good(n - 1)
			return (a + b, a)


	def binary_search(target: (int, float), data: (list, tuple), low: int, high: int):
		"""二分查找的递归实现

		:param target: 待查找的数据值
		:param data: 目标序列
		:param low: 子序列起始值
		:param high: 子序列结束值
		"""
		if low > high:
			return -1

		mid = (low + high) // 2
		if target > data[mid]:
			return binary_search(target, data, mid + 1, high)
		elif target < data[mid]:
			return binary_search(target, data, low, mid - 1)
		else:
			return mid
		
..


Go实现

.. code-block:: go

	package main

	import (
		"fmt"
		"os"
	)

	// n的阶乘
	func Factorial(n int) int {
		if n < 0 {
			os.Exit(-1)
		}
		if n == 0 {
			return 1
		} else {
			return n * Factorial(n-1)
		}
	}

	// 二分查找的递归实现
	func BinarySearch(data []int, target, low, high int) int {
		if low > high {
			return -1
		}
		mid := (high + low) >> 1
		if data[mid] == target{
			return mid
		}else if data[mid] > target{
			return BinarySearch(data, target, low, mid -1)
		}else{
			return BinarySearch(data, target, mid + 1, high)
		}
	}

..


