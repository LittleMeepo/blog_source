---
title: '[LeetCode] 滑动窗口'
date: 2020-12-6 17:07
tags:
- 滑动窗口
- LeetCode
categories:
- 算法
---

## 滑动窗口

#### Leetcode 3 [无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

> 给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

```c++
class Solution {
public:
	int lengthOfLongestSubstring(string s) {
		char c[256] = { 0 };
		int len = s.size();
		int rk = -1, ans = 0;
		for (int i = 0; i < len; i++)
		{
			if (i != 0)
			{
				c[s[i-1]] = 0;
			}
			while (rk + 1 < len && c[s[rk+1]] == 0) {
				c[s[rk+1]] = 1;
				rk++;
			}
			ans = max(ans, rk - i + 1);
		}
		return ans;
	}
};
```

