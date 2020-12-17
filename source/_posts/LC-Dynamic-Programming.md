---
title: LC-Dynamic Programming
date: 2020-11-24 21:53:50
categories:  数据结构与算法
tags: DP
top:
---

# 动态规划

# 1. 动态规划问题的一些思路

## 1.1 原理

- 一般形式 –> 求最值
    - 核心思路： 穷举
    - 特殊性： 因为其存在重叠子问题
        - 暴力穷举的时间复杂度就会非常高
        - 我们就需要备忘录/ DP table来优化穷举的过程，避免不必要的计算
        - 一般会具备最优子结构，才能通过子问题的最值得到原问题的最值
        - 关于如何穷举所有的可能解，就需要分析得出状态转移方程，以上也就是动态规划的三要素
- 三要素
    - 重叠子问题
        - 子问题之间需要互相独立
        - 重叠子问题 需要做某种记录，使用dp table 或者备忘录
            - 可以尝试通过状态压缩，来缩小DP table的大小，只记录必要的数据
    - 最优子结构
    - 状态转移方程
        - 就是n 和 n-1, n-2之间的关系
        - 如何思考
            - 明确base case
            - 明确状态
            - 明确选择
            - 定义dp数组/ 函数的含义
- 和递归的对比
    - 递归往往是自上而下，分解出子问题，再利用栈的逻辑，反向拿到结果
    - 而动态规划是自下而上的，直接定义出了子问题，然后我们来定义状态是如何转移的

## 1.2 基本方法论

- 逻辑是需要找到一些规律来指导我们解决动态规划问题
    - 寻找子问题
    - 递归求解
    - 重叠子问题
    - 无后效性
    - 状态存储

## 1.3 递归

- 使用贪心算法是有可能失效导致无法生成全局最优解的
- 最优化问题本身指的是一种求极值的方法，即在一组约束为等式或不等式的条件下，使系统的目标函数达到极值，即最大值或最小值的过程
- 我们需要做到的是在满足条件的组合里找出最优解的组合
    - 枚举
        - 求出所有满足条件的组合，然后看看这些组合能否得到最大值或者最小值
        - 关键问题 – 如何得到满足条件的所有组合
            - 使用递归来获得满足约束条件的所有组合
            - –》 递归是一种枚举的手法，满足限定条件下，做递归
            - 在递归的过程中，每一步都对需要求解的问题来进行判断
        - 使用递归的好处
            - 是在使用堆栈的时候实际上自然的就保存了每一层的局部变量们
            - 递归的形式也给了回溯的能力，通过堆栈保存了上一步的状态的
- 直接使用递归的问题
    - 实际上是穷举了所有的组合，导致效率很低
    - 构成的递归树会随着数据规模的增长而指数级别的增长
    - 直接使用递归调试相对比较困难，因为需要思考每一层的表现
- 递归的优化
    - 剪枝
        - 减少搜索的分支数量
        - 重叠子问题，可以通过提前存储，来减少重复的运算

## 1.4 备忘录

- 前面说过我们可以通过枚举来获得满足条件的所有组合，然后再来求需要的极值，但是在这中间是很有可能有重叠子问题的
- 可用的前提条件
    - 无后效性
        - 即在通过 A 阶段的子问题推导 B 阶段的子问题的时候，我们不需要回过头去再根据 B 阶段的子问题重新推导 A 阶段的子问题
        - 即子问题之间的依赖是单向性的
- 以斐波那契数列为例
    - 我们在疯狂的重复计算子函数，大部分的递归树都是重复的
    - 举个例子
        - f(9) = f(8) + f(7)
        - 在计算f(8)的时候，又需要计算一遍f(7)
        - 如此反复
- 因此我们需要使用备忘录来解决重复计算的问题
    - 在每次计算出一个子问题的答案以后，将这个临时的中间结果记录到备忘录当中，然后再返回
- 常见的可用数据结构
    - 数组
    - 哈希表

## 1.5 动归的应用场景

### 1.5.1 0-1背包问题

- 问题描述
    - 总重量为W的背包和N个物品，对每个物品，有重量w和价值v两个属性，即第i个物品重量为w[i],价值为v[i]
- 算法问题分析
    - 首先看是什么类型的问题
        - 求最优解的问题（max / min）
            - 考虑下使用贪心算法的可能性
            - 暴力递归做穷举
            - 动态规划

        - 求可行性的问题
        - 求方案总数的问题
    - 进一步确认是否为动态规划问题
        - 数据需要不可排序
        - 数据不可交换
- 状态转移方程
    - 确定终止条件
        - 背包容量为0了或者物品数量为0要终止执行
        - 背包中的物品数量和背包还能装下的重量是这个问题的状态参数

```jsx
int dp(int[] w, int[] v, int N, int W) {
    // 创建备忘录
    int[][] dp = new int[N+1][W+1];
  
    // 初始化状态
    for (int i = 0; i < N + 1; i++) { dp[i][0] = 0; }
    for (int j = 0; j < W + 1; j++) { dp[0][j] = 0; }
  
    for (int tn = 1; tn < N + 1; tn++) { // 遍历每一件物品
    for (int rw = 1; rw < W + 1; rw++) { // 背包容量有多大就还要计算多少次
        if (rw < w[tn]) {
          // 当背包容量小于第tn件物品重量时，只能放入前tn-1件
          dp[tn][rw] = dp[tn-1][rw];
        } else {
                // 当背包容量还大于第tn件物品重量时，进一步作出决策
          dp[tn][rw] = Math.max(dp[tn-1][rw], dp[tn-1][rw-w[tn]] + v[tn]);
        }
      }
    }
  
  return dp[N][W];
}

int solveDP() {
  int N = 3, W = 5; // 物品的总数，背包能容纳的总重量
  int[] w = {0, 3, 2, 1}; // 物品的重量
  int[] v = {0, 5, 2, 3}; // 物品的价值
  
  return dp(w, v, N, W); // 输出答案
}
```

# 2. 动态规划的写法分析

- 动态规划问题描述
    - 重叠子问题
        - 穷举过程中存在重复计算的现象
    - 无后效性
        - 子问题之间的依赖是单向性的
        - 某阶段状态一旦确定，就不受后续决策的影响
    - 最优子结构
        - 子问题之间必须相互独立，或者说后续的计算可以通过前面的状态推导出来

# 3. 案例分析

## 3.1 硬币找零问题

- 问题描述
    - 给定n种不同面值的硬币 分别记为c[0], c[1], c[2]…c[n]
    - 总数额k
    - 编写一个函数计算出最少需要几枚硬币凑出这个金额K
    - 若无可能的组合，返回-1
- 思路
    - 这是个求最值的问题
    - 求最值问题的核心原理就是穷举，将所有可能的凑硬币的方法做穷举，看看最少需要多少枚硬币
- 贪心算法
    - 每一步计算做出的都是在当前看起来最好的选择
        - 即局部最优解，并不从整体来考虑
    - 基本思路
        - 建立数学模型
        - 将待求解的问题划分为若干子问题，对每个子问题进行求解，得到子问题的局部最优解
        - 将子问题的局部最优解进行合并，最终得到基于局部最优解的一个解
- 用贪心算法来解决上述问题的时候，会遇到“过于贪心”导致最终无解的问题，所以需要引入回溯来解决过于贪心的问题
- 贪心算法的实现

```

int getMinCoinCountHelper(int total, int[] values, int valueCount) {
    int rest = total;
    int count = 0;

    // 从大到小遍历所有面值
    for (int i = 0; i < valueCount; ++ i) {
        int currentCount = rest / values[i]; // 计算当前面值最多能用多少个
        rest -= currentCount * values[i]; // 计算使用完当前面值后的余额
        count += currentCount; // 增加当前面额用量

        if (rest == 0) {
            return count;
        }
    }

    return -1; // 如果到这里说明无法凑出总价，返回-1
}

int getMinCoinCount() {
    int[] values = { 5, 3 }; // 硬币面值
    int total = 11; // 总价
    return getMinCoinCountHelper(total, values, 2); // 输出结果
}
```

- 贪心算法 + 回溯的实现

```

int getMinCoinCountOfValue(int total, int[] values, int valueIndex) {
    int valueCount = values.length;
    if (valueIndex == valueCount) { return Integer.MAX_VALUE; }

    int minResult = Integer.MAX_VALUE;
    int currentValue = values[valueIndex];
    int maxCount = total / currentValue;

    for (int count = maxCount; count >= 0; count --) {
        int rest = total - count * currentValue;

        // 如果rest为0，表示余额已除尽，组合完成
        if (rest == 0) {
            minResult = Math.min(minResult, count);
            break;
        }

        // 否则尝试用剩余面值求当前余额的硬币总数
        int restCount = getMinCoinCountOfValue(rest, values, valueIndex + 1);

        // 如果后续没有可用组合
        if (restCount == Integer.MAX_VALUE) {
            // 如果当前面值已经为0，返回-1表示尝试失败
            if (count == 0) { break; }
            // 否则尝试把当前面值-1
            continue;
        }

        minResult = Math.min(minResult, count + restCount);
    }

    return minResult;
}

int getMinCoinCountLoop(int total, int[] values, int k) {
    int minCount = Integer.MAX_VALUE;
    int valueCount = values.length;
    
    if (k == valueCount) {
        return Math.min(minCount, getMinCoinCountOfValue(total, values, 0));
    }

    for (int i = k; i <= valueCount - 1; i++) {
        // k位置已经排列好
        int t = values[k];
        values[k] = values[i];
        values[i]=t;
        minCount = Math.min(minCount, getMinCoinCountLoop(total, values, k + 1)); // 考虑后一位

        // 回溯
        t = values[k];
        values[k] = values[i];
        values[i]=t;
    }

    return minCount;
}

int getMinCoinCountOfValue() {
    int[] values = { 5, 3 }; // 硬币面值
    int total = 11; // 总价
    int minCoin = getMinCoinCountLoop(total, values, 0);
    
    return (minCoin == Integer.MAX_VALUE) ? -1 : minCoin;  // 输出答案
}
```

- 使用动态规划来求解
    - 初始化状态
        - 终止条件
    - 状态参数
        - 子问题与原问题之间会发生变化的变量
        - 变量是目标兑换金额K
    - 整体思路
        - 确定初始化状态
        - 确定状态参数
        - 设计决策
- 递归与动态规划的对比
    - 递归是自上而下的，从目标问题开始，不断将大问题拆解成子问题，直到子问题不可拆借为止
    - 如果要自底向上，那就应该首先求出所有的子问题，然后通过底层的子问题向上求解更大的问题

    ```
    // 伪代码
    DP(values, k) {
    res = MAX
    for c in values
      // 作出决策，找到需要硬币最少的那个结果
      res = min(res, 1 + DP(values, k-c)) // 递归调用

    if res == MAX
      return -1

    return res
    }
    ```

```

int getMinCounts(int k, int[] values) {
   int[] memo = new int[k + 1]; // 创建备忘录
   memo[0] = 0; // 初始化状态
   for (int i = 1; i < k + 1; i++) { memo[i] = k + 1; }
   
   for (int i = 1; i < k + 1; i++) {
       for (int coin : values) {
           if (i - coin < 0) { continue; }
           memo[i] = Math.min(memo[i], memo[i - coin] + 1); // 作出决策
       }
   }

   return memo[k] == k + 1 ? -1 : memo[k];
}

int getMinCountsDPSolAdvance() {
   int[] values = { 3, 5 }; // 硬币面值
   int total = 22; // 总值

   return getMinCounts(total, values); // 输出答案
}
```

# 62. Unique Paths

## Solution 1: Recursion

```
class Solution {
    int result = 0;
    
    public int uniquePaths(int m, int n) {
        // direction is either right or down 
        helper(1, 1, m, n);
        return result;
    }
    
    private void helper(int x, int y, int m, int n) {
        if (x < 1 || x > m || y < 1 || y > n) {
            return;
        }
        
        if (x == m && y == n) {
            result ++;
        }
        helper(x + 1, y, m, n);
        helper(x, y + 1, m, n);
    }
}
```

## Solution 2: DP

- 注意的点
    - 状态转移方程
        - `dp[m][n] = dp[m-1][n] + dp[m][n-1];`
    - 因为只有两个方向，实际上第一行第一列能到达的方法都只有一种，所以我们在做初始化的时候可以都初始化为1，具体的迭代从col = 1, row = 1开始，即第二行和第二列 这样来做就okay了

```
class Solution { 
    public int uniquePaths(int m, int n) {
        // 二维DP   
        // dp[m][n] = dp[m-1][n] + dp[m][n-1];
        int[][] dp = new int[m][n];
        
        for (int[] arr: dp) {
            Arrays.fill(arr, 1);
        }
        
        for (int col = 1; col < n; col ++) {
            for(int row = 1; row < m; row ++) {
                dp[row][col] = dp[row-1][col] + dp[row][col-1];
            }
        }
        
        return dp[m-1][n-1];
    }
}

```

# 70. Climbing Stairs

## Solution 1: DP

- `dp[n] = dp[n-1] + dp[n-2]`

```
class Solution {
    public int climbStairs(int n) {
        // dp[n] = dp[n-1] + dp[n-2]
        if (n <= 0) {
            return 0;
        }
        
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n];
    }
}

```

# 91. Decode Ways

## Solution 1: DP

- 每次可以选择往前走一步或者两步

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

- 每次都可以往前走两步或者一步

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

- 记录当前的最小值，碰到比最小值大的值，就更新最大利润

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

- 字典里的词可以选用
- 可以重复使用
- 目的是需要拼起string
- DFS 深度优先遍历

## Solution 1 Brute Force

- Kind of DFS
- Go through one way to the very bottom, and we need to go through all possible result if needed

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

- 解法1 包含太多的重复，通过记录在某个位置以后能不能成功来保证不会重复运行

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