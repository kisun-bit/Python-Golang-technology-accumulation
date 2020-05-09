4.6 串
=======================


.. code-block:: python

	def substring_index(parent_string: str, child_string: str) -> int:
		"""查找master主串中第一次出现子串child的位置

		:param parent_string:  主串
		:param child_string: 子串
		:return: 位置索引（若为-1代表匹配失败）
		"""
		p_start, c_start, pos, idx = 0, 0, 0, -1
		p_max, c_max = len(parent_string) - 1, len(child_string) - 1
		if p_max < c_max:
			return idx

		while p_start <= p_max + 1:
			if c_start == c_max + 1:
				idx = p_start - c_start
				break
			if p_start <= p_max and parent_string[p_start] == child_string[c_start]:
				c_start += 1
			else:
				p_start -= c_start
				c_start = 0
			p_start += 1

		return idx



	if __name__ == '__main__':
		index = substring_index("3123123101234", "20123")
		print(f'位置：{index}')

	
..