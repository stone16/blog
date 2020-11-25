---
title: LC-Dynamic Programming
date: 2020-11-24 21:53:50
categories:  数据结构与算法
tags: DP
top:
---

# 1. 动态规划问题的一些思路
## 1.1 原理
+ 一般形式 --> 求最值
    + 核心思路： 穷举
    + 特殊性： 因为其存在重叠子问题
        + 暴力穷举的时间复杂度就会非常高
        + 我们就需要备忘录/ DP table来优化穷举的过程，避免不必要的计算
        + 一般会具备最优子结构，才能通过子问题的最值得到原问题的最值
        + 关于如何穷举所有的可能解，就需要分析得出状态转移方程，以上也就是动态规划的三要素


+ 三要素
    + 重叠子问题
        + 子问题之间需要互相独立
        + 重叠子问题 需要做某种记录，使用dp table 或者备忘录
            + 可以尝试通过状态压缩，来缩小DP table的大小，只记录必要的数据
    + 最优子结构
    + 状态转移方程
        + 就是n 和 n-1, n-2之间的关系
        + 如何思考
            + 明确base case
            + 明确状态
            + 明确选择
            + 定义dp数组/ 函数的含义

+ 和递归的对比
    + 递归往往是自上而下，分解出子问题，再利用栈的逻辑，反向拿到结果
    + 而动态规划是自下而上的，直接定义出了子问题，然后我们来定义状态是如何转移的

## 1.2 基本方法论

+ 逻辑是需要找到一些规律来指导我们解决动态规划问题
    + 寻找子问题
    + 递归求解
    + 重叠子问题
    + 无后效性
    + 状态存储


# 2. 案例分析

## 2.1 硬币找零问题

# 91. Decode Ways 

## Solution 1: DP 

+ 每次可以选择往前走一步或者两步

```
class Solution {
    public int numDecodings(String s) {
        if (s == null || s.length() == 0) {
            return 0;
        }
        
        int[] dp = new int[s.length() + 1];
        // could go by 1 step or 2 step 
        // dp[n] = dp[n-1] + dp[n-2]
        //     s.charAt(n) 1 --9
        //     s.charAt(n-1) 1 -- 2  if n-1 == 1  n 0 - 9  if n-1 == 2 n 0 - 6 
            
        dp[0] = 1;
        dp[1] = s.charAt(0) == '0' ? 0 : 1;
        for (int i = 2; i <= s.length(); i ++) {
            if (s.charAt(i - 1) != '0') {
                dp[i] += dp[i-1];
            }
            
            if (s.charAt(i-2) == '1' || (s.charAt(i-2) == '2' && 
                                         s.charAt(i - 1) >= '0' && s.charAt(i-1) <= '6' )){
                dp[i] += dp[i-2];
            }
        }
        return dp[s.length()];
    }
}

```

## Solution 2: Recursive 

+ 每次都可以往前走两步或者一步

```
class Solution {
    
    HashMap<Integer, Integer> memo = new HashMap<>();
    
    public int numDecodings(String s) {
        if (s == null || s.length() == 0) {
            return 0;
        }
        return helper(s, 0);
    }
    
    private int helper(String s, int pos) {
        if (pos == s.length()) {
            return 1;
        } 
        
        if (s.charAt(pos) == '0') {
            return 0;
        }
        
        if (pos == s.length() - 1) {
            return 1;
        }
        
        if (memo.containsKey(pos)) {
            return memo.get(pos);
        }
        
        int ans = helper(s, pos + 1);
        
        if (Integer.parseInt(s.substring(pos, pos+2)) <= 26) {
             ans += helper(s, pos + 2);
        }
        
        memo.put(pos, ans);
        
        return ans;
    }
}

```
# 121. Best Time to Buy and Sell Stock 

## Solution 1: Brute Force 

```
class Solution {
    public int maxProfit(int[] prices) {
        int result = 0;
        
        for (int i = 0; i < prices.length; i++) {
            for (int j = i + 1; j < prices.length; j++) {
                result = Math.max(result, prices[j] - prices[i]);
            }
        }
        
        return result;
    }
}
```

## Solution 2: One Pass 

+ 记录当前的最小值，碰到比最小值大的值，就更新最大利润

```
class Solution {
    public int maxProfit(int prices[]) {
        int minprice = Integer.MAX_VALUE;
        int maxprofit = 0;
        for (int i = 0; i < prices.length; i++) {
            if (prices[i] < minprice)
                minprice = prices[i];
            else if (prices[i] - minprice > maxprofit)
                maxprofit = prices[i] - minprice;
        }
        return maxprofit;
    }
}
```


# LC 139 Word Break

+ 字典里的词可以选用
+ 可以重复使用
+ 目的是需要拼起string
+ DFS  深度优先遍历

## Solution 1 Brute Force 
+ Kind of DFS 
+ Go through one way to the very bottom, and we need to go through all possible result if needed 


```
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        
        if (s == null || s.length() == 0 || wordDict == null || wordDict.size() == 0) {
            return false;
        }
        
        return subString(s, new HashSet(wordDict), 0);
    }
    
    private boolean subString(String testStr, Set<String> wordDict, int position) {
        
        if (position == testStr.length()) {
            return true;
        }
        
        // 注意这里的<=  因为substring的第二个参数是不包括在第二个参数里面的
        for (int n = position + 1; n <= testStr.length(); n ++) {
            if (wordDict.contains(testStr.substring(position, n)) && subString(testStr, wordDict, n)) {
                return true;
            }
        }
        return false;
    }
}

```

## Solution 1.1 Brute Force with array record 

+ 解法1 包含太多的重复，通过记录在某个位置以后能不能成功来保证不会重复运行

```
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        
        if (s == null || s.length() == 0 || wordDict == null || wordDict.size() == 0) {
            return false;
        }
        
        return subString(s, new HashSet(wordDict), 0, new Boolean[s.length()]);
    }
    
    private boolean subString(String testStr, Set<String> wordDict, int position, Boolean[] memo) {
        
        if (position == testStr.length()) {
            return true;
        }
        
        if (memo[position] != null) {
            return memo[position];
        }
        
        // 注意这里的<=  因为substring的第二个参数是不包括在第二个参数里面的
        for (int n = position + 1; n <= testStr.length(); n ++) {
            if (wordDict.contains(testStr.substring(position, n)) && subString(testStr, wordDict, n, memo)) {
                return true;
            }
        }
        memo[position] = false;
        
        return false;
    }
}
```

## Solution 2 Dynamic Programming 

```
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        
        Set<String> wordDictSet = new HashSet(wordDict);
        boolean[] dp = new boolean[s.length() + 1];
        dp[0] = true;
        for (int i = 1; i <= s.length(); i ++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && wordDictSet.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.length()];
    }
}
```

# Reference
1. https://labuladong.gitbook.io/algo/dong-tai-gui-hua-xi-lie/1.1-dong-tai-gui-hua-ji-ben-ji-qiao/dong-tai-gui-hua-xiang-jie-jin-jie
