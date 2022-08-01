---
title: "算法 - Leetcode - 剑指offer(2nd Edition)（持续更新）"
date: 2022-08-01T10:56:31+08:00
categories:
    - Java
tags:
    - Leetcode
draft: false
---

&emsp;&emsp;上次刷算法题都是大一下学期的事情了，想起来整个大一都在用C语言写各种简单算法题的日子，那个时候写个栈都要自己手动写，太煎熬了。\
&emsp;&emsp;现在为了一年后校招（或半年后准备投暑期实习）的事情开始用Java重捡算法题，从Leetcode的剑指offer（第2版）开始记录一下每天的题解和笔记，不过因为最近还在忙着学JavaWeb，后面还有SSM等各种框架还有中间件的学习，所以算法题争取一天能写个1-2道吧。
<div align = right>2022.8.1</div>

### 03.数组中重复的数字
&emsp;&emsp;[03.数组中重复的数字](https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)&emsp;难度：easy\
&emsp;&emsp;看到题目最先想到的用哈希表，可以直接AC：
```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        HashSet<Integer> set = new HashSet<>();
        for(int i : nums)
        {
            if(set.contains(i))
            {
                return i;
            }
            else
            {
                set.add(i);
            }
        }
        return -1;
    }
}
```
&emsp;&emsp;如果允许修改原数组，那可以将其排序后遍历，时间复杂度和空间复杂度都可以进一步下降：
```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        Arrays.sort(nums);
        for(int i = 0;i < nums.length-1;i++)
        {
            if(nums[i] == nums[i+1])
            {
                return nums[i];
            }
        }
        return -1;
    }
}
```

### 04.二维数组中的查找
&emsp;&emsp;[04.二维数组中查找](https://leetcode.cn/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)&emsp;难度：medium\
&emsp;&emsp;一开始除了暴力以外没有头绪，后来看了评论以后发现从矩阵右上角看是颗二叉搜索树，难度瞬间变成了easy，说起来这算脑筋急转弯级别的了吧...
```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if(matrix.length == 0 || matrix[0].length == 0)
        {
            return false;
        }//一开始没想到有空数组的测试用例，看到卡在[[]]的用例时快脑淤血了
        int x = 0,y = matrix[0].length-1;
        while(y>=0 && x<matrix.length)
        {
            if(target>matrix[x][y])
            {
                x++;
            }
            else if(target<matrix[x][y])
            {
                y--;
            }
            else
            {
                return true;
            }
        }
        return false;
    }
}
```

### 05.替换空格
&emsp;&emsp;[05.替换空格](https://leetcode.cn/problems/ti-huan-kong-ge-lcof/)&emsp;难度：easy\
&emsp;&emsp;思路挺简单的，只需要注意String对象不可变，所以用上StringBuilder就可以了。
```java
class Solution {
    public String replaceSpace(String s) {
        StringBuilder sb = new StringBuilder();
        for(int i=0;i<s.length();i++)
        {
            char c = s.charAt(i);
            if(c == ' ')
            {
                sb.append("%20");
            }
            else
            {
                sb.append(c);
            }
        }
        return sb.toString();
    }
}
```
&emsp;&emsp;当然可以用replace()方法一行解决，不过这题就没啥意义了（虽然本身也没啥意义）。

### 06.从尾到头打印链表
&emsp;&emsp;[06.从尾到头打印链表](https://leetcode.cn/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)&emsp;难度：easy\
&emsp;&emsp;以前好像写过反转链表的题目，不过这题是要返回数组，和直接改指针不太一样。\
&emsp;&emsp;第一反应是用栈，然后装填进数组中：
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] reversePrint(ListNode head) {
        Stack<Integer> stack = new Stack();
        ListNode node = head;
        int cnt = 0;
        while(node != null)
        {
            stack.push(node.val);
            cnt++;
            node = node.next;
        }
        int[] ans = new int[cnt];
        for(int i=0;i<cnt;i++)
        {
            ans[i] = stack.pop();
        }
        return ans;
    }
}
```
&emsp;&emsp;但这样的复杂度太高了，不如直接将数组倒过来装填：
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution{
    public int[] reversePrint(ListNode head) {
        ListNode node = head;
        int cnt = 0;
        while(node != null) //先遍历到链表尾来统计链表的长度，确立数组的大小
        {
            cnt++;
            node = node.next;
        }
        node = head;
        int[] ans = new int[cnt];
        for(int i=cnt-1;i>=0;i--)
        {
            ans[i] = node.val;
            node = node.next;
        }
        return ans;
    }
}
```