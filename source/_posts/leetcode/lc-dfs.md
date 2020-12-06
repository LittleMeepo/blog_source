---
title: '[LeetCode] DFS'
date: 2020-12-6 17:05
tags:
- DFS
- LeetCode
categories:
- 算法
---

## 深度优先遍历

#### Leetcode 22 [括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

> 数字 *n* 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。
>
> ```bash
> 输入：n = 3
> 输出：[
>        "((()))",
>        "(()())",
>        "(())()",
>        "()(())",
>        "()()()"
>      ]
> ```

```c++
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        dfs("",n,0);
        return result;
    }
private:
    vector<string> result;
    void dfs(string cur,int left,int right){
        if(left==0&&right==0){
            result.push_back(cur);
            return;
        }
        if(left>0)dfs(cur+"(",left-1,right+1);
        if(right>0)dfs(cur+")",left,right-1);
    }
};
```

