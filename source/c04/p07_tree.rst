4.7 树
===============================

**树的定义及属性**

树是一种将元素分层次存储的抽象数据结构，每一个元素在树中(除根元素外)都有唯一一个双亲节点和0哥或复数个孩子结点。

**一个例子**

在一个python程序中，当使用单继承时，类与类之间的关系便是一个多路树结构，如下图所示：

.. raw:: html

   <center>
    <img src="../_static/c04_p07_01.png" style="padding:3px;" alt="Python异常层次结构示意图"></img>
   </center>
   
..

树结构在生活中随意可见，离我们最近的便要数操作系统的文件系统中文件与目录的分层关系便是一颗标准无序树，除此之外，IP子网划分、公司结构组成等等，均是以树来描述的。
所以，树提供了一种更加真实、非线性的组织形式，并由此在数据库系统（DBMS）、文件系统、图形用户界面、网站和其他计算机系统中得以广泛应用。那么，我们又如何来实现一棵树呢？

**Tree ADT**

如果要构造一棵树，第一步便是“搭架子”，也即是说定义树的抽象数据类型
一颗树(Tree)的抽象数据类型应该满足如下方法：

* Tree.root(): 返回树Tree的根节点
* Tree.is_root(p): 如果位置P为根节点则返回True
* Tree.parent(p): 返回位置p的双亲节点
* Tree.children(p): 产生位置p的子节点的迭代
* Tree: 树Tree满足迭代协议，即从根节点开始迭代
* Tree.is_leaf(p): 如果位置p为叶子节点，则返回真
* Tree.is_empty(): 如果树Tree为空树，则返回真

通过对树的抽象数据类型的分析，我们得到了描述一颗树的基本功能方法

**树的抽象基类**

.. code-block:: python

	import abc
	import inspect


	# classes


	class Position(object):
		"""An abstraction representing the location of a single node in a tree structure
		"""

		def element(self):
			"""Return the element stored at this position

			:return: object
			"""
			raise NotImplementedError('`{}` must be implemented by subclass'.format(inspect.stack()[1][3]))

		def __eq__(self, other):
			"""Return True if other position represents the same location.

			:param other: instance of Position
			:return: bool
			"""
			raise NotImplementedError('`{}` must be implemented by subclass'.format(inspect.stack()[1][3]))

		def __ne__(self, other):
			"""Return True if other position don't represents the same location.

			:param other: instance of Position
			:return: bool
			"""
			raise NotImplementedError('`{}` must be implemented by subclass'.format(inspect.stack()[1][3]))


	class Tree(abc.ABC):
		"""Tree ADT
		"""

		@abc.abstractmethod
		def root(self):
			"""Return this tree's root node(None if  empty).

			:return: bool
			"""
			raise NotImplementedError('`{}` must be implemented by subclass'.format(inspect.stack()[1][3]))

		@abc.abstractmethod
		def is_root(self, pos: Position):
			"""Return True if `pos` represents the root node of this tree.

			:param pos: instance of Position
			:return: bool
			"""
			return self.root() == pos

		@abc.abstractmethod
		def is_leaf(self, pos: Position):
			"""Return True if `pos` represents the leaf node

			:param pos: instance of Position
			:return: bool
			"""
			return sum(1 for _ in self.children(pos)) == 0

		@abc.abstractmethod
		def parent(self, pos: Position):
			"""Return parent node of node `pos`.

			:param pos: instance of Position
			:return: instance of Position(or None)
			"""
			raise NotImplementedError('`{}` must be implemented by subclass'.format(inspect.stack()[1][3]))

		@abc.abstractmethod
		def children(self, pos: Position):
			"""Generate iteration of the children node of `pos`

			:param pos:instance of Position
			:return:iteration
			"""
			raise NotImplementedError('`{}` must be implemented by subclass'.format(inspect.stack()[1][3]))

		@abc.abstractmethod
		def nodes_count(self):
			"""Return the number of this tree's node

			:return: int
			"""
			raise NotImplementedError('`{}` must be implemented by subclass'.format(inspect.stack()[1][3]))

		def __iter__(self):
			"""Generate iteration of the tree
			"""
			raise NotImplementedError('`{}` must be implemented by subclass'.format(inspect.stack()[1][3]))

		def depth(self, pos: Position) -> int:
			"""Return the depth of node pos in this tree
			"""
			if self.is_root(pos):
				return 0
			else:
				return 1 + self.depth(self.parent(pos))

..
