## 动态规划

用一个dp数组来保存以$a_i$为结尾的一个子序列的某种性质，再用一个全局变量记录其中的最值。

#### Leetcode 53 [最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

> 给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int pre = 0, maxAns = nums[0];
        for (const auto &x: nums) {
            pre = max(pre + x, x);
            maxAns = max(maxAns, pre);
        }
        return maxAns;
    }
}
```

#### Leetcode 91 [解码方法](https://leetcode-cn.com/problems/decode-ways/)

> 一条包含字母 A-Z 的消息通过以下方式进行了编码：
>
> ```bash
> 'A' -> 1
> 'B' -> 2
> ...
> 'Z' -> 26
> ```
>
>
> 给定一个只包含数字的非空字符串，请计算解码方法的总数。
>
> 题目数据保证答案肯定是一个 32 位的整数。
>

```c++
class Solution {
public:
    int check(char a){
        return a!='0';
    }
    int func(char a,char b){
        return a=='1'||a=='2'&&b<='6';
    }
    int numDecodings(string s){
        int len=s.length();
        vector<int> dp(len,0);
        if(len==0||s[0]=='0')
            return 0;
        if(len==1)
            return check(s[0]);
        dp[0]=1;
        dp[1]=check(s[1])+func(s[0],s[1]);
        for(int i=2;i<len;i++){
            if(check(s[i]))
                dp[i]=dp[i-1];
            if(func(s[i-1],s[i]))
                dp[i]+=dp[i-2];
        }
        return dp[len-1];
    }
};
```

#### Leetcode 121 [买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

> 给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
>
> 如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。
>
> 注意：你不能在买入股票前卖出股票。
>

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int len=prices.size();
        if(len==0||len==1) return 0;
        if(len==2){
            if(prices[1]>prices[0])return prices[1]-prices[0];
            else return 0;
        }
        for(int i=0;i<len-1;i++){
            prices[i]=prices[i+1]-prices[i];
        }
        prices.pop_back();
        int pre=0;
        int max_sum=prices[0];
        for(const int& i:prices){
            pre=max(i,pre+i);
            max_sum=max(pre,max_sum);
        }
        if(max_sum>0)return max_sum;
        else return 0;
    }
};
```

#### Leetcode 139 [单词拆分](https://leetcode-cn.com/problems/word-break/)

> 给定一个非空字符串 s 和一个包含非空单词的列表 wordDict，判定 s 是否可以被空格拆分为一个或多个在字典中出现的单词。
>
> 说明：
>
> 拆分时可以重复使用字典中的单词。
> 你可以假设字典中没有重复的单词。

```c++
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        unordered_set<string> set;
        for(auto word:wordDict)
        {
            set.insert(word);
        }

        vector<bool> dp(s.size()+1);
        dp[0]=true;//空字符串合法
        for(int i=1;i<=s.size();i++)
        {
            for(int j=0;j<i;j++)
            {
                if(dp[j]&&set.find(s.substr(j,i-j))!=set.end())
                {
                    dp[i]=true;
                    break;
                }
            }
        }
        return dp[s.size()];
    }
};
```

#### Leetcode 152 [乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)

> 给你一个整数数组 `nums` ，请你找出数组中乘积最大的连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

```c++
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        if(nums.size()==0) return 0;
        int result=nums[0],maxValue=nums[0],minValue=nums[0];
        for(int i=1;i<nums.size();i++){
            int tempMax=max(nums[i],maxValue*nums[i]);
            int tempMin=min(nums[i],maxValue*nums[i]);
            maxValue=max(tempMax,minValue*nums[i]);
            minValue=min(tempMin,minValue*nums[i]);
            result=max(maxValue,result);
        }
        return result;
    }
};//因为是乘法，和最大最小值有关，所以用两个dp数组
```

#### 198 [打家劫舍](https://leetcode-cn.com/problems/house-robber/)

> 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。
>
>  ```bash
> 示例 1：
> 
> 输入：[1,2,3,1]
> 输出：4
> 解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
>      偷窃到的最高金额 = 1 + 3 = 4 。
>  ```

```c++
class Solution {
public:
    int rob(vector<int>& nums) {
        if(nums.empty())
        {
            return 0;
        }
        int size=nums.size();
        if(size==1)
        {
            return nums[0];
        }
        vector<int> dp(size,0);
        dp[0]=nums[0];
        dp[1]=max(nums[0],nums[1]);
        for(int i=2;i<size;i++)
        {
            dp[i]=max(dp[i-2]+nums[i],dp[i-1]);
        }
        return dp[size-1];
    }
};
```

#### Leetcode 264 [丑数 II](https://leetcode-cn.com/problems/ugly-number-ii/)

> 编写一个程序，找出第 n 个丑数。
>
> 丑数就是质因数只包含 2, 3, 5 的正整数。
>
> ```bash
> 示例:
> 
> 输入: n = 10
> 输出: 12
> 解释: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 是前 10 个丑数。
> ```

```c++
class Solution {
public:
    int nthUglyNumber(int n) {
        if(n<=0)return 0;
        if(n==1)return 1;
        int p2=0,p3=0,p5=0;
        int nums[1691];
        nums[0]=1;
        for(int i=1;i<n;i++)
        {
             int ugly=min(nums[p2]*2,min(nums[p3]*3,nums[p5]*5));
             nums[i]=ugly;
             if(ugly==nums[p2]*2) p2++;
             if(ugly==nums[p3]*3) p3++;
             if(ugly==nums[p5]*5) p5++;
        }
        return nums[n-1];
    }
};
```

#### Leetcode 300 [最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

> 给定一个无序的整数数组，找到其中最长上升子序列的长度。

```c++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        if(nums.size()==0)
        {
            return 0;
        }
        int length=nums.size();
        int maxAns=0;
        vector<int> dp(length,0);
        for(int i=0;i<length;i++)
        {
            dp[i]=1;
            for(int j=0;j<i;j++)
            {
                if(nums[j]<nums[i])
                {
                    dp[i]=max(dp[i],dp[j]+1); 
                }
            }
            maxAns=max(maxAns,dp[i]);
        }
        return maxAns;
    }
};
```

#### Leetcode 1143 [最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/) (二维DP)

> 给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列的长度。
>
> 一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
> 例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。两个字符串的「公共子序列」是这两个字符串所共同拥有的子序列。
>
> 若这两个字符串没有公共子序列，则返回 0。
>

```c++
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int len1=text1.size();
        int len2=text2.size();
        if(len1==0||len2==0) return 0;
        int dp[len1+1][len2+1];
        memset(dp,0,sizeof(dp));

        for(int i=1;i<=len1;i++)
        {
            for(int j=1;j<=len2;j++)
            {
                if(text1[i-1]==text2[j-1])
                {
                    dp[i][j]=dp[i-1][j-1]+1;
                }
                else{
                    dp[i][j]=max(dp[i][j-1],dp[i-1][j]);
                }
            }
        }    
        return dp[len1][len2];
    }
};
```

