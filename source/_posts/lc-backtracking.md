## 回溯

#### Leetcode 46 [全排列](https://leetcode-cn.com/problems/permutations/)

> 给定一个 **没有重复** 数字的序列，返回其所有可能的全排列。

```c++
class Solution {
public:
    void backtrace(vector<vector<int>> &res,vector<int> &output,int first,int len){
        if(first==len)
        {
            res.push_back(output);
            return;
        }
        for(int i=first;i<len;i++){
            swap(output[i],output[first]);
            backtrace(res,output,first+1,len);
            swap(output[i],output[first]);
        }
    }
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> res;
        backtrace(res,nums,0,(int)nums.size());
        return res;
    }
};
```

