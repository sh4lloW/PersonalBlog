---
title: "算法 - Leetcode - 剑指offer(2nd Edition)（持续更新）"
date: 2022-08-01T10:56:31+08:00
categories:
    - 算法
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

### 07.重建二叉树
&emsp;&emsp;[07.重建二叉树](https://leetcode.cn/problems/zhong-jian-er-cha-shu-lcof/)&emsp;难度：medium\
&emsp;&emsp;算是把当时学数据结构时的算法实现了一遍，其实这种题作为教学来说手写出来是很快的，但代码实现的时候写的确实不算流畅，磕磕绊绊写了个简单但是复杂度较高的算法：
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        if(preorder.length == 0)
        {
            return null;
        }
        int rootVal = preorder[0];  //先序遍历开头就是根节点
        int rootIndex = 0;
        for(int i=0;i<inorder.length;i++)   //定位中序遍历中根节点的位置
        {
            if(inorder[i] == rootVal)
            {
                rootIndex = i;
                break;
            }
        }
        TreeNode root = new TreeNode(rootVal);
        //直接开始递归建树
        root.left = buildTree(Arrays.copyOfRange(preorder,1,rootIndex+1),Arrays.copyOfRange(inorder,0,rootIndex));
        root.right = buildTree(Arrays.copyOfRange(preorder,rootIndex+1,preorder.length),Arrays.copyOfRange(inorder,rootIndex+1,inorder.length));
        return root;
    }
}
```
&emsp;&emsp;后来在评论区看到了@Krahets的解法，利用哈希表可以达到O(N)的时空复杂度：
```java
class Solution {
    int[] preorder;     //保留的先序遍历
    HashMap<Integer, Integer> dic = new HashMap<>();    //哈希表用于存储中序遍历的值与索引
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        this.preorder = preorder;
        for(int i = 0; i < inorder.length; i++)     //放入哈希表中
            dic.put(inorder[i], i);
        return recur(0, 0, inorder.length - 1);     //递归解法
    }
    TreeNode recur(int root, int left, int right) {
        if(left > right)    //递归的终止条件，即回到了根节点
        {
            return null;
        }
        TreeNode node = new TreeNode(preorder[root]);   //建立根节点
        int i = dic.get(preorder[root]);    //定位中序遍历中根节点的位置
        node.left = recur(root + 1, left, i - 1);
        //左子树的递归，左子树的根节点为先序根节点+1，左边界为left，右边界为中序根节点-1
        node.right = recur(root + i - left + 1, i + 1, right);
        //右子树的递归，右子树的根节点为（先序根节点+左子树数量+1），其中(i-left)为左子树的数量
        //左边界为中序根节点+1，右边界为right
        return node;    //返回根节点
    }
}
```
&emsp;&emsp;在这个算法中，最花费的时间的操作是哈希表的初始化，为O(N)，而递归建立结点与定位的时间仅为O(1)，故这是一个O(N)的算法。

### 09.用两个栈实现队列
&emsp;&emsp;[09.用两个栈实现队列](https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)&emsp;难度：easy\
\
![图片](https://s1.328888.xyz/2022/08/04/jJqN5.png)\
```java
class CQueue {
    Stack<Integer> s1 = new Stack<>();  //栈建在外面，默认构造方法
    Stack<Integer> s2 = new Stack<>();
    public CQueue() {
        
    }
    
    public void appendTail(int value) {
        s1.push(value);     //直接推入栈1
    }
    
    public int deleteHead() {
        if(s1.empty() && s2.empty())    //当两个栈都空时说明队列已空
        {
            return -1;
        }
        if(s2.empty())  //栈1非空但栈2空，就将栈1的全放到栈2中
        {
            while(!s1.empty())
            {
                s2.push(s1.pop());
            }
        }
        return s2.pop();    //从栈2弹出
    }
}

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
```

### 10.I.斐波那契数列
&emsp;&emsp;[10.I.斐波那契数列](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/)&emsp;难度：easy\
&emsp;&emsp;看到通过率这么低就感觉有些不对劲，十秒钟写了个递归交了上去：
```java
class Solution {
    public int fib(int n) {
        if(n == 0 || n ==1)
        {
            return n;
        }
        return fib(n-1) + fib(n-2)%1000000007;
    }
}
```
&emsp;&emsp;不出意外TLE了，于是改换DP：
```java
class Solution {
    public int fib(int n) {
        if(n == 0 || n == 1)
        {
            return n;
        }
        int[] dp = new int[n+1];
        dp[0] = 0;
        dp[1] = 1;
        for(int i=2;i<=n;i++)
        {
            dp[i] = dp[i-1] + dp[i-2];
            dp[i] %= 1000000007;
        }
        return dp[n];
    }
}
```
&emsp;&emsp;原本在返回值时进行模运算，结果在某个数据那溢出了，评论区说是要在每次求和时就进行模运算，不得不说Leetcode的题面有时候是真的坑人...\
&emsp;&emsp;其实在运算过程中只用到了dp[i-1]和dp[i-2]两个值，这里可以用两个整型变量取代，不必建立长度n+1的数组，取代以后空间复杂度可以从O(N)降为O(1)。

### 10.II.青蛙跳台阶问题
&emsp;&emsp;[10.II.青蛙跳台阶问题](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)&emsp;难度：easy\
&emsp;&emsp;经典的DP问题，实际也就是斐波那契数列问题，解法与上题一致。\
&emsp;&emsp;但值得注意的是这题默认fib(0)没有意义，在0阶台阶时也会有一种解法，所以dp[0]=1。