---
title: LC - Tree
date: 2020-11-01 10:54:24
categories: 数据结构与算法
tags: Tree
top:
---
# LC 543. Diameter of Binary Tree 

+ Diameter -- length of the longest path between any two nodes in a tre 
+ Binary Tree 
```
public class Solution {
    int max = 0;
    
    public int diameterOfBinaryTree(TreeNode root) {
        maxDepth(root);
        return max;
    }
    
    private int maxDepth(TreeNode root) {
        if (root == null) return 0;
        
        int left = maxDepth(root.left);
        int right = maxDepth(root.right);
        
        max = Math.max(max, left + right);
        
        return Math.max(left, right) + 1;
    }
}
```

# LC 938. Range Sum of BST 

+ 二叉查找树
    + 左子树的值会小于根节点的值
    + 右子树的值都大于根节点的值

+ 二叉查找树当中找区间，一定是需要用到二叉查找树本身的性质的

## Solution 1: Brute Force

+ 遍历整个树，对于每个节点，对其进行判断，若值在区间内，则累加到一个全局变量上


```
class Solution {
    int sum = 0;
    
    public int rangeSumBST(TreeNode root, int L, int R) {
        
        addToSum(root, L, R);
        
        return sum;
    }
    
    void addToSum(TreeNode node, int L, int R) {
        if (node == null) return;
        
        if (node.val >= L && node.val <= R) {
            sum += node.val;
        }
        addToSum(node.left, L, R);
        addToSum(node.right, L, R);
    }
}

```

## Solution 2: Leverage on BST 

+ 逻辑是如果node.val < L, 那么只有右子树是有可能获取在区间内的值的
```
class Solution {
    int ans;
    public int rangeSumBST(TreeNode root, int L, int R) {
        ans = 0;
        dfs(root, L, R);
        return ans;
    }

    public void dfs(TreeNode node, int L, int R) {
        if (node != null) {
            if (L <= node.val && node.val <= R)
                ans += node.val;
            if (L < node.val)
                dfs(node.left, L, R);
            if (node.val < R)
                dfs(node.right, L, R);
        }
    }
}
```

+ Q: L13 -- L16, 为什么不能写成
    + `node.val > R`
    + `node.val < L`


# LC 199: Binary Tree Right Side View

+ 对树的分层操作
    + 使用BFS算法

## Solution 1: 使用两个队列

+ 两个队列，一个为当前层，一个为下一层

```
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        if (root == null) {
            return new ArrayList<>();
        }
        
        ArrayDeque<TreeNode> currentQueue = new ArrayDeque<>();
        ArrayDeque<TreeNode> nextQueue = new ArrayDeque<>();
        
        nextQueue.add(root);
        
        List<Integer> result = new ArrayList<>();
        
        TreeNode node = null;

        while(!nextQueue.isEmpty()) {
            
            currentQueue = nextQueue.clone();
            nextQueue.clear();
            while (!currentQueue.isEmpty()) {
                node = currentQueue.poll();
                
                if (node.left != null) {
                    nextQueue.add(node.left);
                }
                if (node.right != null) {
                    nextQueue.add(node.right);
                }
            }
            
            if (currentQueue.isEmpty()) {
                result.add(node.val);
            }
        }
        return result;
    }
}

```

## Solution 2: 一个队列 + Sentinel

+ 使用null作为卫兵节点来区分不同层

```
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        if (root == null) {
            return new ArrayList<>();
        }
        
        Queue<TreeNode> queue = new LinkedList() {
            {offer(root);
            offer(null);}
        };
        
        TreeNode prev, cur = root;
        List<Integer> result = new ArrayList();
        
        while (!queue.isEmpty()) {
            prev = cur;
            cur = queue.poll();
            
            while(cur != null) {
                if (cur.left != null) {
                    queue.offer(cur.left);
                }
                if (cur.right != null) {
                    queue.offer(cur.right);
                }
                
                prev = cur;
                cur = queue.poll();
            }
            
            result.add(prev.val);
            
            if (!queue.isEmpty()) queue.offer(null);
        }
        return result;
    }
}

```