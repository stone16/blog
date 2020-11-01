---
title: LC - Linked List
date: 2020-10-24 20:33:44
categories: 数据结构与算法
tags: LinkedList
top:
---
# LC-23 Merge K Sorted Lists
+ 给定k个链表，每个链表都按照升序排列，合并链表使得最终的链表也按照升序排列
+ 最直观的想法
+ 以一个链表为基准，拿出一个链表来做比较，各保留一个指针，然后来做合并操作


## Solution1: Brute Force 
+ 都放到一个arraylist当中
+ 使用Collection的排序算法，对其进行排序
+ 然后创建一个新的链表，来返回结果
```
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        
        if (lists == null || lists.length == 0) {
            return null;
        }
        
        List<Integer> list = new ArrayList();
        
        for (ListNode ln : lists) {
            while (ln != null) {
                list.add(ln.val);
                ln = ln.next;
            }
        }
        
        Collections.sort(list);
        
        ListNode dummy = new ListNode(0);
        ListNode result = dummy;
        
        for (int i = 0; i < list.size(); i++) {
            result.next = new ListNode(list.get(i));
            result = result.next;
        }
        return dummy.next;
        
    }
}
```

## Solution 2
+ 比较每个链表头，将最小的放到result中，一个一个这样比较，相当于只遍历了以便就生成了结果
+ 注意终止循环的判断，应该是几个链表都到头了才退出

```
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        
        if (lists == null || lists.length == 0) {
            return null;
        }
        
        ListNode result = new ListNode(0);
        ListNode head = result;
        
        while (true) {
            
            int min = Integer.MAX_VALUE;
            int min_index = -1;
            boolean shouldBreak = true;
            
            for (int i = 0; i < lists.length; i++) {
                if (lists[i] != null && lists[i].val < min) {
                    min = lists[i].val;
                    min_index = i;
                    shouldBreak = false;
                }
            }
        
            if (shouldBreak) break;
            
            // get one value min 
            head.next = lists[min_index];
            head = head.next;
            lists[min_index] = lists[min_index].next;
        }
        
        
        return result.next;
        
        
    }
}

```

## Solution 3: 使用Priority Queue 

```
public ListNode mergeKLists(ListNode[] lists) { 
        PriorityQueue q = new PriorityQueue<>((o1, o2) -> o1.val - o2.val);
        for(ListNode l : lists){
            if(l!=null){
                // 这里加进去的不是value 而是几个ListNode
                q.add(l);
            }        
        }
        ListNode head = new ListNode(0);
        ListNode point = head;
        while(!q.isEmpty()){ 
            point.next = q.poll();
            point = point.next; 
            ListNode next = point.next;
            if(next!=null){
                // 顺次下移以后比较下一个节点
                q.add(next);
            }
        }
        return head.next;
    }
```

# LC-24 Swap Nodes in Pairs
+ 交换相邻节点
+ 临界条件
    + 单个节点
    + 没有节点
+ 因为会涉及到首节点的替换变动，所以为了简化问题，需要设置一个 dummy node，使得`dummy.next = head`
+ 画一个基本的swap示意图可以发现整个替换会涉及四个节点
    + 首节点的上个节点原先指向首节点的指针
    + 首节点本身
    + 次节点本身
    + 次节点原先指向次节点下一个节点的指针

+ 因为是单向链表，我们没法用prev指针来找到上一个节点，所以需要在过程中做好prev的记录，而对于次节点的下一个节点，完全可以用next指针来表示
+ 另外要注意做swap的时候的顺序问题

```
// Iteration的版本
class Solution {
    public ListNode swapPairs(ListNode head) {
        
        ListNode dummy = new ListNode(0);
        
        dummy.next = head;
        
        ListNode prev = dummy;
        
        while (head != null && head.next != null) {
            
            
            ListNode cur = head;
            ListNode next = head.next;
          
            // Swapping 
            prev.next = next;
            cur.next = next.next;
            next.next = cur;
            
            prev = cur;
            head = cur.next;
        }
        return dummy.next;
    }
}
```

# LC-234 Palindrome Linked List 

## Solution 1: 递归

+ 反向输出链表的递归逻辑
```
function print_values_in_reverse(ListNode head)
    if head is NOT null
        print_values_in_reverse(head.next)
        print head.val
```

+ 题解

```
class Solution {
    
    private ListNode front;
    
    private boolean recursiveCheck(ListNode cur) {
        
        if (cur != null) {
            if (!recursiveCheck(cur.next)) return false;
            if (cur.val != front.val) return false;
            front = front.next;
        }
        return true;
    }
    
    public boolean isPalindrome(ListNode head) {
        front = head;
        return recursiveCheck(head);
    }
}

```

## Solution 2: 反向后半链表来做比较

```
class Solution {

    public boolean isPalindrome(ListNode head) {

        if (head == null) return true;

        // Find the end of first half and reverse second half.
        ListNode firstHalfEnd = endOfFirstHalf(head);
        ListNode secondHalfStart = reverseList(firstHalfEnd.next);

        // Check whether or not there is a palindrome.
        ListNode p1 = head;
        ListNode p2 = secondHalfStart;
        boolean result = true;
        while (result && p2 != null) {
            if (p1.val != p2.val) result = false;
            p1 = p1.next;
            p2 = p2.next;
        }        

        // Restore the list and return the result.
        firstHalfEnd.next = reverseList(secondHalfStart);
        return result;
    }

    // Taken from https://leetcode.com/problems/reverse-linked-list/solution/
    private ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode nextTemp = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextTemp;
        }
        return prev;
    }

    private ListNode endOfFirstHalf(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;
        while (fast.next != null && fast.next.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

# LC-1474 Delete N nodes after M nodes of a linked list 

+ 链表热身题
+ 注意边界条件的判定，题中已给m和n的限定范围，所以不需要额外做判断了
+ 对于n的周期性去除的node，注意可以不用额外空间，通过改变指针的指向即可
```
class Solution {
    public ListNode deleteNodes(ListNode head, int m, int n) {
        
        ListNode cur = head;
        ListNode last = head;
        
        while (cur != null) {
            
            int mCount = m;
            int nCount = n;
            
            while (cur != null && mCount > 0 ) {
                last = cur;
                cur = cur.next;
                mCount --;
            }
            
            while (cur != null && nCount > 0) {
                cur = cur.next;
                nCount --;
            }
            
            last.next = cur;
        }
        return head;
    }
}
```