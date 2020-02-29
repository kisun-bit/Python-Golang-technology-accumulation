5.2 两数相加
========================

:来源: 力扣（LeetCode）
:链接: https://leetcode-cn.com/problems/add-two-numbers/


题目
>>>>>>>>>>>>>>>>>>>>>>>>>>

给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。
如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。
您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例
>>>>>>>>>>>>>>>>>>>>>>>>>>

.. code-block:: bash

    输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
    输出：7 -> 0 -> 8
    原因：342 + 465 = 807

..

解法
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. code-block:: python

    # Python 方法-(性能低)
    # 该方法在其数据结构的基础上，
    # 通过循环，分别获取单个节点的数值n1\n2，
    # 并在循环外部暂存进位，此外若出现链表长度不一致，则动态地补0

    # Definition for singly-linked list.
    class ListNode(object):
        def __init__(self, x):
            self.val = x
            self.next = None

    class Solution(object):
        def addTwoNumbers(self, l1, l2):
            """
            :type l1: ListNode
            :type l2: ListNode
            :rtype: ListNode
            """
            l3_head = None
            l3_point = None
            carry_bit = 0

            while 1:
                res = l1.val + l2.val + carry_bit
                cur, carry_bit = res % 10, res // 10

                if l3_point is None:
                    l3_point = ListNode(cur)
                    l3_head = l3_point
                else:
                    l3_point.next = ListNode(cur)
                    l3_point = l3_point.next

                if l1.next is None and l2.next is not None:
                    l1 = ListNode(0)
                    l2 = l2.next
                elif l1.next is not None and l2.next is None:
                    l1 = l1.next
                    l2 = ListNode(0)
                elif l1.next is not None and l2.next is not None:
                    l1 = l1.next
                    l2 = l2.next
                elif l1.next is None and l2.next is None and carry_bit != 0:
                    l3_point.next = ListNode(carry_bit)
                    break
                else:
                    break

            return l3_head
..


.. code-block:: python

    # python 方法二，在方法一的基础上改进，内存消耗小，速度居中
    class Solution(object):
    def addTwoNumbers(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        l3_head = ListNode(0)
        l3_point = l3_head
        carry_bit = 0

        while l1 or l2:
            l1_val = l1.val if l1 else 0
            l2_val = l2.val if l2 else 0
            res = l1_val + l2_val + carry_bit
            l3_point.next, carry_bit = ListNode(res % 10), res // 10
            l3_point = l3_point.next
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None

        if carry_bit != 0:
            l3_point.next = ListNode(1)

        return l3_head.next
..