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

## 03.数组中重复的数字
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

## 04.二维数组中的查找
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

## 05.替换空格
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

## 06.从尾到头打印链表
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

## 07.重建二叉树
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
&emsp;&emsp;后来在评论区看到了@Krahets的解法，利用哈希表可以达到$O(N)$时空复杂度：
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
&emsp;&emsp;在这个算法中，最花费的时间的操作是哈希表的初始化，为$O(N)$，而递归建立结点与定位的时间仅为$O(1)$，故这是一个$O(N)$的算法。

## 09.用两个栈实现队列
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

## 10.I.斐波那契数列
&emsp;&emsp;[10.I.斐波那契数列](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/)&emsp;难度：easy\
&emsp;&emsp;看到通过率这么低就感觉有些不对劲，十秒钟写了个递归交了上去：
```java
class Solution {
    public int fib(int n) {
        if(n == 0 || n == 1)
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
&emsp;&emsp;其实在运算过程中只用到了dp[i-1]和dp[i-2]两个值，这里可以用两个整型变量取代，不必建立长度n+1的数组，取代以后空间复杂度可以从$O(N)$降为$O(1)$。

## 10.II.青蛙跳台阶问题
&emsp;&emsp;[10.II.青蛙跳台阶问题](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)&emsp;难度：easy\
&emsp;&emsp;经典的DP问题，实际也就是斐波那契数列问题，解法与上题一致。\
&emsp;&emsp;但值得注意的是这题默认fib(0)没有意义，在0阶台阶时也会有一种解法，所以dp[0]=1。

## 11.旋转数组的最小数字
&emsp;&emsp;[11.旋转数组的最小数字](https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)&emsp;难度：easy\
&emsp;&emsp;**部分有序的数组考察的就是二分查找。**\
&emsp;&emsp;可以把原先的数组看成两个递增数组组合而成，如果中间点小于最右边，说明此时位于右边递增数组，就搜索中间点左边；如果中间点大于最右边，说明此时位于左边递增数组，就搜索中间点右边；如果中间点与右边相等，无法判断，可以直接进行遍历。
```java
class Solution {
    public int minArray(int[] numbers) {
        int left = 0;
        int right = numbers.length-1;
        while (left < right) {
            int mid = left + (right - left) / 2;
            /*
            为什么用left + (right - left) / 2而不是(left + right) / 2，
            是因为如果数组很大时(left + right)会爆int
            */
            if (numbers[mid] < numbers[right]) {
                right = mid;
            } else if (numbers[mid] > numbers[right]) {
                left = mid + 1;
            } else {
                right --;
                /*
                right --用于缩减范围，实际上和遍历的时间复杂度是一样的，用下面的会更加直观：

                int min = left;
                for(int i = left + 1; i < right; i++) {
                    if (numbers[i] < numbers[min]) {
                        min = i;
                    }
                }
                return numbers[min];
                */
            }
        }
        return numbers[left];
    }
}
```
&emsp;&emsp;相比于直接暴力遍历，二分法可将时间复杂度由$O(N)$降为$O(logN)$。

## 12.矩阵中的路径
&emsp;&emsp;[12.矩阵中的路径](https://leetcode.cn/problems/ju-zhen-zhong-de-lu-jing-lcof/)&emsp;难度：medium\
&emsp;&emsp;DFS的模板题，这里用DFS+回溯的方法，需要一个二维数组visited用于判断是否走过，一个一维数组便于遍历字符串。\
&emsp;&emsp;在DFS中，递归结束的条件：
* false : 搜索超出边界、该点已经走过、当前点与目标字符不符
* true : 字符串匹配完
```java
class Solution {
    public boolean exist(char[][] board, String word) {
        if (word.length() == 0) {
            return true;
        }
        boolean[][] visited = new boolean[board.length][board[0].length];
        char[] charsofword = word.toCharArray();
        for (int i = 0; i < board.length; i++) {
            for (int j = 0;j < board[0].length; j++) {
                /*
                遍历矩阵，使其可以从任意一个点开始搜索找到匹配的字符串
                如果某个点开始的dfs查找成功则直接返回true
                */
                if (dfs(board, charsofword, visited, 0, i, j)) {
                    return true;
                }
            }
        }
        return false; // 所有点开始都找不到匹配的字符串
    }
    private boolean dfs(char[][] board, char[] charsofword, boolean[][] visited, int index, int i, int j) {
        //index用于标记当前字符串的下标，从0开始
        if (i < 0 || i > board.length - 1 || j < 0 || j > board[0].length - 1 || // 超出边界
            visited[i][j] == true || // 该点已走过
            charsofword[index] != board[i][j]) { // 与字符串不匹配
            return false;
        }
        if (index == charsofword.length - 1) { // 当字符串匹配完说明符合
            return true;
        }
        visited[i][j] = true; // 标记当前点已走过
        boolean res = dfs(board, charsofword, visited, index + 1, i + 1, j)
                    || dfs(board, charsofword, visited, index + 1, i - 1, j)
                    || dfs(board, charsofword, visited, index + 1, i, j + 1)
                    || dfs(board, charsofword, visited, index + 1, i, j - 1);
                    // 往四个方向搜索
        visited[i][j] = false; // 回溯，将visited置为false
        return res;
    }
}
```

## 14.I.剪绳子
&emsp;&emsp;[14.I.剪绳子](https://leetcode.cn/problems/jian-sheng-zi-lcof/)&emsp;难度：medium\
&emsp;&emsp;碰到求最优解的问题第一反应是动态规划：
* 对于长度为n的绳子剪掉后的最大乘积，可以通过前面比n小的绳子转化一部分过来
* 设剪掉的第一段长度为`j`，剪掉的长度至少为2，因为剪掉的长度为1对乘积没有改变，所以在内嵌循环中从2开始
* 第一次剪掉以后，需要对剩下的绳子进行处理，如果不剪剩下的绳子，则乘积为`j * (i - j)`，如果剪，乘积为`j * dp[i - j]`，两者取最大值，`max(j * (i - j), j * dp[i - j])`
* 针对每次剪掉的第一段的长度不同，当前绳子的最大乘积`dp[i]`会不断变化，所以也需要对`dp[i]`进行更新
```java
class Solution {
    public int cuttingRope(int n) {
        int[] dp = new int[n+1];
        dp[0] = dp[1] = 0;
        dp[2] = 1;
        for (int i = 3; i <= n; i++) {
            for (int j = 2; j < i; j++) { //每次选取剪第一段的长度
                int temp = Math.max(j * (i - j), j * dp[i - j]);
                dp[i] = Math.max(dp[i], temp);
            }
        }
        return dp[n];
    }
}
```
&emsp;&emsp;题解区有关于贪心的解法，有涉及到关于数论的知识：
* 根据奇偶证明，任何大于1的数可以由2和3相加而成
* `2*2=1*4`，`2*3>1*5`，所以将数字拆解成2和3，得到的乘积最大
* `2*2*2<3*3`，3越多，乘积越大
```java
class Solution {
    public int cuttingRope(int n) {
        if (n < 4) {
            return n - 1; 
        }
        int productofthree = 1;
        while (n > 4) {
            productofthree *= 3;
            n -= 3;
        }
        return productofthree * n; //最后n会小于等于4，将3的乘积乘于最后的n就是绳子的乘积
    }
}
```
&emsp;&emsp;用贪心的话时间复杂度可由$O(N^2)$降为$O(N)$，并且由于不用dp数组，空间复杂度由$O(N)$降为$O(1)$。

## 14.II.剪绳子II
&emsp;&emsp;[14.II.剪绳子II](https://leetcode.cn/problems/jian-sheng-zi-ii-lcof/)&emsp;难度：medium\
&emsp;&emsp;这题与上题题面一致，唯一不同的就是加大了n的测试范围，使得上面的代码会溢出，而DP的解法使得取余以后max函数不能比较大小，所以这里对贪心的解法进行取余：
```java
class Solution {
    public int cuttingRope(int n) {
        if (n < 4) {
            return n - 1; 
        }
        long productofthree = 1;
        while (n > 4) {
            productofthree *= 3;
            productofthree %= 1000000007;
            n -= 3;
        }
        return (int)(productofthree * n % 1000000007);
    }
}
```
&emsp;&emsp;当然DP如果换成BigInteger也是可以用的，不过效率很低。

## 15.二进制中1的个数
&emsp;&emsp;[15.二进制中1的个数](https://leetcode.cn/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)&emsp;难度：easy\
&emsp;&emsp;我是铸币，只会位运算，但相比转成字符串再数还是没那么铸币：
```java
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int cnt = 0;
        for (int i = 0; i < 32; i++) {
            if ((n & 1) == 1) {
                cnt++;
            }
            n >>= 1;
        }
        return cnt;
    }
}
```

## 16.数值的整数次方
&emsp;&emsp;[16.数值的整数次方](https://leetcode.cn/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)&emsp;难度：medium\
&emsp;&emsp;快速幂算法模板题：
* $$x^n = (x^2)^\frac{n}{2}$$
* 对于这种形式有两种情况：
* $n$为偶数，$x^n$ = $(x^2)^{\lfloor\frac{n}{2}\rfloor}$
* $n$为奇数，$x^n$ = $x(x^2)^{\lfloor\frac{n}{2}\rfloor}$\
```java
class Solution {
    public double myPow(double x, int n) {
        if (n == 0) {
            return 1.0;
        }
        if (n < 0) { // 负数次方的处理
            x = 1 / x;
            n = -n;
        }
        double res = 1;
        while (n != 0) {
            if (n % 2 != 0) { // n为奇数就多乘一次x，因为最后n肯定会为1所以始终会赋值给res
                res *= x;
            }
            x *= x;
            n /= 2; // 位运算会快一些，但这题如果不变数据类型用位运算的话会卡用例
        }
        return res;
    }
}
```
&emsp;&emsp;本质是二分法，所以时间复杂度为O(logN)。

## 17.打印从1到最大的n位数
&emsp;&emsp;[17.打印从1到最大的n位数](https://leetcode.cn/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)&emsp;难度：easy\
&emsp;&emsp;应该感叹一下终于有送分题了：
```java
class Solution {
    public int[] printNumbers(int n) {
        int[] list = new int[((int)Math.pow(10,n)) - 1];
        for (int i = 0; i <list.length; i++) {
            list[i] =i + 1;
        }
        return list;
    }
}
```

## 18.删除链表的结点
&emsp;&emsp;[18.删除链表的结点](https://leetcode.cn/problems/shan-chu-lian-biao-de-jie-dian-lcof/)&emsp;难度：easy\
&emsp;&emsp;双指针法：\
![图片](https://s1.328888.xyz/2022/08/14/TsqLI.png)
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
    public ListNode deleteNode(ListNode head, int val) {
        if (head == null) { // 链表为空
            return null;
        }
        if (head.val == val) { // 头节点就是要删除的点
            return head.next;
        }
        ListNode cur = head.next;
        ListNode pre = head;
        while (cur.val != val) { // 遍历链表找到要删除的点
            if(cur.next != null) {
                pre = cur;
                cur = cur.next;
            }
        }
        pre.next = cur.next; // 删除结点
        return head;
    }
}
```
&emsp;&emsp;值的注意的是，这里只返回了链表头结点，实际被删除的点本身并没有释放，指针也没有删除。

## 20.表示数值的字符串

​		[20.表示数值的字符串](https://leetcode.cn/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)	难度：medium

​		一道有限自动机，其实编译原理学的挺好的，真写代码有点抓瞎，一开始用哈希表频繁出错，debug了半天，看了题解才发现自动机少画了加减号的特殊情况。

​		在评论区发现了新解法，同样是自动机的思路，不过通过三个boolean变量然后遍历字符串就可以了，相比较来说反而是更容易想到也更简单的思路，但学完编译原理以后反而忽略了这种。

```java
class Solution {
    public boolean isNumber(String s) {
        if (s.length() == 0) {
            return false;
        }
        s = s.trim();	// 去除字符串前后的空格
        boolean num = false;	// 数字
        boolean point = false;	// 小数点
        boolean e = false;	// e或E
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c >= '0' && c <= '9') {	// 遇到数字
                num = true;
            }else if (c == '.' && !point && !e){	// 遇到小数点的前提是小数点只出现过一次小数点且没有e
                point = true;
            }else if ((c == 'e' || c == 'E') && !e && num) {	// 遇到e的前提是e只出现过一次且数字在e前面出现过
                e = true;
                num = false;	// e后面要跟数字，一旦出现把num置为false，防止1e这种情况出现
            }else if ((c == '+' || c == '-') && (i == 0 || s.charAt(i-1) == 'e' || s.charAt(i-1) == 'E')) {
                // 遇到加减号的情况，加减号只能出现在字符串开头(i == 0)或e的后面，一开始就是没考虑过这种情况导致6+1这种样例出错
                // do nothing
            }else {
                return false;
            }
        }
        return num;
    }
}
```

​		被各种样例卡了好久，难度是medium但恶心程度犹胜hard，只能说面试时敢出这种我就敢挂，实在不行也只能在面试官面前画自动机 :)

## 21.调整数组顺序使奇数位于偶数前面

​		[21.调整数组顺序使奇数位于偶数前面](https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)	难度：easy

​		泪目了，罗勇上课讲过的案例真实出现在剑指offer上面，到现在都感谢学校里的数据结构课。

​		运用快排的思想，唯一需要注意的是`left`和`right`是在变动的，所以在循环中每次也要判断`left < right`。

```java
class Solution {
    public int[] exchange(int[] nums) {
        int left = 0;
        int right = nums.length - 1;
        while (left < right) {
            while ((left < right) && nums[left] % 2 != 0) {	// 奇数在数组左边，所以遇到奇数就跳过
                left++;										// 如果用位运算会更快点
            }
            while ((left < right) && nums[right] % 2 == 0) { // 同理
                right--;
            }
            int temp = nums[left];
            nums[left] = nums[right];
            nums[right] = temp;
        }
        return nums;
    }
}
```

## 22.链表中倒数第K个节点

​		[链表中倒数第K个结点](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)	难度：easy

​		一开始用了最容易想到的办法，遍历链表找出元素个数，然后定位最终位置即可：

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
    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode node = head;
        int length = 1;
        while (node.next != null) {		// 计算链表长度
            length++;
            node = node.next;
        }
        length -= k;	// 倒数第k个，所以长度-k
        while (length != 0) {	// 再从头走到倒数第k个
            head = head.next;
            length--;
        }
        return head;
    }
}
```

​		还有双指针写法，让快指针比慢指针先走k步，这样快指针走到链表尾部时慢指针就到了链表倒数第k位。

​		虽然多建了一个节点变量，但只遍历了一遍：

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
    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode fast = head;
        ListNode slow = head;
        while (k != 0) {	// 快指针比慢指针快走k步
            fast = fast.next;
            k--;
        }
        while (fast != null) {	// 等快指针到尾部说明慢指针到了倒数第k位
            fast = fast.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

## 24.反转链表

​		[24.反转链表](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/)	难度：easy

​		我记得我写过两三遍的反转链表了，但看了一下发现没有提交记录，我也不记得是在哪写的了。

​		迭代法，用三个指针（虽然我一开始不知道这个叫迭代法）。

![图片](https://s1.328888.xyz/2022/08/25/weY6d.png)

​		代码如下：

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
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode pre = null;
        ListNode cur = head;
        ListNode next = null;
        while (cur != null) {
            next = cur.next;	// next指针后移
            cur.next = pre;		// 倒置指针
            pre = cur;			// pre指针后移
            cur = next;			// cur指针后移
        }
        return pre;
    }
}
```

​		还可以用递归，递归法容易想到，但深度为N，空间复杂度较高。

## 25.合并两个排序的链表

​		[25.合并两个排序的链表](https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)	难度：easy

​		好像之前在严蔚敏的数据结构课本上看到过这题，只需要遍历两个链表逐次比较就可以了，难点在于需要设置一个伪头节点在第一轮将节点添加到合并链表中。

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
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode head = new ListNode(0);	// 伪头节点，把两个链表中的节点都接到它后面
        ListNode node = head;				// 用于遍历
        while (l1 != null && l2 != null) {
            if (l1.val < l2.val) {			// l1的值小于l2就把l1接到合并链表后面
                node.next = l1;
                node = node.next;
                l1 = l1.next;
            }else {							// 反之把l2接后面
                node.next = l2;
                node = node.next;
                l2 = l2.next;
            }
        }
        if (l1 != null) {					// 按照上面的循环条件，最后肯定会有其中一个链表的最后一个节点没有接上，所以这里判断是哪个
            node.next = l1;
        }
        if (l2 != null) {
            node.next = l2;
        }
        return head.next;					// 伪头结点的下一个就是合并链表的真正起点
    }
}
```

## 26.树的子结构

​		[26.树的子结构](https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/)	难度：medium

​		有关二叉树的题目一般都有关递归，关于递归最重要的一点是：不要细想递归实现的细节，而是清楚递归函数调用的目的。

​		这道题首先需要明确的一点是：`isSubStructure()`函数的目的就是确认B是不是A的子结构。

​		B是A的子结构有三种情况：

* B的根节点就是A的根节点，那么只需要判断B下面的结构与A是否匹配，调用`search()`函数
* B的根节点在A的左子树下，那么就往左递归，返回`isSubStructure(A.left, B)`
* B的根节点在A的右子树下，那么就往右递归，返回`isSubStructure(A.right, B)`

​		在判断下面结构是否匹配的函数`search()`中，终止条件就是看值是否相等以及是否遍历完。

```java
class Solution {
    public boolean isSubStructure(TreeNode A, TreeNode B) {
        // 根据题意，A树或B树为空直接返回false
        if (A == null || B == null) {
            return false;
        }
        // 分别对应：B的根对应A的根、B的根在A左子树下、B的根在A右子树下
        return search(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
    }

    public boolean search(TreeNode A, TreeNode B) {
        // 终止条件1：B为空代表已经全部匹配完成，返回true
        if (B == null) {
            return true;
        }
        // 终止条件2：A为空代表匹配错误，返回false
        if (A == null) {
            return false;
        }
        // 返回当前根节点是否相等，并去匹配当前根节点下的左右子树
        return A.val == B.val && search(A.left, B.left) && search(A.right, B.right);
    }
}
```

## 27.二叉树的镜像

​		[27.二叉树的镜像](https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/)	难度：easy

​		很经典的翻转二叉树问题，只需要从根节点下面开始递归交换左右子树就可以了。

```java
class Solution {
    public TreeNode mirrorTree(TreeNode root) {
        // 树为空
        if (root == null) {
            return root;
        }
        // 递归交换子树
        TreeNode temp = root.left;
        root.left = mirrorTree(root.right);
        root.right = mirrorTree(temp);
        return root;
    }
}
```

## 28.对称的二叉树

​		[28.对称的二叉树](https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/)	难度：easy

​		一颗二叉树是对称的，那么它左边的左边要和右边的右边相同，左边的右边要和右边的左边相同。

​		如图：

![图片](https://s1.328888.xyz/2022/09/28/sUgU7.png)

​		接下来就是思考递归结束的条件：

* 如果对应的值不相等，那肯定不是镜像的，返回false
* 如果有一边节点为空，另一边还有节点，说明不对称，返回false
* 如果两边节点都为空，说明两边都走完了，对称，返回true

​		代码如下：
```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        // 如果树为空直接返回true
        if (root == null) {
            return true;
        }
        // 从两边开始递归
        return compare(root.left, root.right);
    }
    public boolean compare(TreeNode left, TreeNode right) {
        // 两边节点都为空
        if (left == null && right == null) {
            return true;
        }
        // 有了上边的前提，这里只要有一个为空说明不对称
        if (left == null || right == null) {
            return false;
        }
        // 判断值是否相等以及继续往深处递归，后两个是分别判断左边的左边和右边的右边相等以及左边的右边与右边的左边相等
        return left.val == right.val && compare(left.left, right.right) && compare(left.right, right.left);
    } 
}
```

## 29.顺时针打印矩阵

​		[29.顺时针打印矩阵](https://leetcode.cn/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)	难度：easy

​		模拟题，难点在于边界的控制。

```java
class Solution {
    public int[] spiralOrder(int[][] matrix) {
        int row = matrix.length;
        // 一开始在这交了一发RE就是没判空
        if (row == 0) {
            return new int[0];
        }
        int col = matrix[0].length;
        // ans数组用于装顺序答案，大小为矩阵的长×宽
        int[] ans = new int[row * col];
        // index用于给数组定位
        int index = 0;
        // 四个边界
        int left = 0, right = col-1, top = 0, bottom = row-1;
        // 死循环，出去的条件就是边界出了问题
        while (true) {
            // 从左往右
            for (int i = left; i <= right; i++) {
                ans[index++] = matrix[top][i];
            }
            top++;	// 走完顶上一行，top下降
            if (top > bottom) {		// if条件就是边界超出
                break;
            }
            // 从上往下
            for (int i = top; i <= bottom; i++) {
                ans[index++] = matrix[i][right];
            }
            right--;	// 走完右边一列，right往左
            if(right < left) {
                break;
            }
            // 从右往左
            for (int i = right; i >= left; i--) {
                ans[index++] = matrix[bottom][i];
            }
            bottom--;	// 走完底部一行，bottom往上
            if (bottom < top) {
                break;
            }
            // 从下往上
            for (int i = bottom; i >= top; i--) {
                ans[index++] = matrix[i][left];
            }
            left++;		// 走完左边一列，left往右
            if (left > right) {
                break;
            }
        }
        return ans;   
    }
}
```

## 30.包含min函数的栈

​		[30.包含min函数的栈](https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/)	难度：easy

​		不太喜欢这种脱裤子放屁的题目，实际题解还是利用栈来实现栈，不过这题的关键点在于怎么让`min()`函数的时间复杂度达到_O(1)_。

​		一开始的设想是自己自定义一个数据类型来动态维护最小值，或者使用双栈，其中一个栈作为辅助栈存最小值，但在评论区看到了一个很不错的解法，它并不省空间，甚至比双栈还要占空间，但它的代码十分好懂，方法简洁清晰。

​		关键点还是在于最小值的维护，在这个算法中，push和pop操作都操作了两个值，其中一个是当前值，一个是当前栈中的最小值，在每次push和pop操作中都会对最小值进行比较。

​		假设现在有2、3、1三个数需要入栈：

![图片](https://s1.328888.xyz/2022/09/29/MBb8k.png)

​		代码如下：

```java
class MinStack {

    /** initialize your data structure here. */
    private Stack<Integer> s = new Stack<Integer>();
    int MIN = Integer.MAX_VALUE;		// 定义一个最小值，用于记录当前栈中最小值，初始为最大整数值
    public MinStack() {
        
    }
    
    public void push(int x) {
        s.push(MIN);					// 首先推最小值入栈
        if (x < MIN) {					// 如果当前要推入的值小于当前栈中的最小值
            MIN = x;					// 更新当前栈的最小值
        }
        s.push(x);						// 推入当前值
    }
    
    public void pop() {
        s.pop();						// 先弹出栈顶值
        if (MIN < s.peek()) {			// 如果之前记录的最小值小于当前栈中最小值
            MIN = s.peek();				// 更新
        }
        s.pop();						// 弹出记录值
    }
    
    public int top() {					// top就是读取当前栈顶元素
        return s.peek();				// 由于push和pop操作都动了两个数，所以栈顶肯定是真实的栈顶元素，不会碰到MIN
    }
    
    public int min() {					// MIN变量中存的就是当前栈中的最小值
        return MIN;
    }
}
```

## 31.栈的压入、弹出序列

​		[31.栈的压入、弹出序列](https://leetcode.cn/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)	难度：medium

​		记得以前数据结构学到栈的时候，就会出两个序列来判断栈能否达成，没想到还真有这样的编程题。

​		同样是模拟，真的建一个栈来模拟序列中的操作：

```java
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {
        Stack<Integer> stack = new Stack<Integer>();
        int index = 0;					// 用于popped的下标
        for (int i = 0; i < pushed.length; i++) {
            stack.push(pushed[i]);		// 将pushed数组中的元素入栈
            while (!stack.empty() && stack.peek() == popped[index]) {		// 先判断栈非空，不然peek匹配不到
                stack.pop();			// 如果栈顶和popped当前元素相同说明可以出栈了
                index++;				// 指针向前
            }
        }
        return stack.empty();			// 看最后栈是否为空，
    }
}
```

## 32.I.从上到下打印二叉树

​		[32.I.从上到下打印二叉树](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)	难度：medium

​		二叉树的层序遍历，思路就是低配版的BFS，用队列存储每一层的所有节点，并对每个节点的左右子节点加入到队列中。

```java
class Solution {
    public int[] levelOrder(TreeNode root) {
        // 树为空
        if (root == null) {
            return new int[0];
        }
        // BFS中的队列
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        // 用来存最后的遍历序列
        ArrayList<Integer> ans = new ArrayList<Integer>();
        // 先将根节点加入到队列中
        queue.add(root);
        while (!queue.isEmpty()) {			// 循环的条件是队列非空，队列空了就是树遍历完了
            TreeNode node = queue.poll();
            ans.add(node.val);				// 队列弹出的点加入到数组中
            if (node.left != null) {		// 将左子节点加入到队列中
                queue.add(node.left);
            }
            if (node.right != null) {		// 将右子节点加入到队列中
                queue.add(node.right);
            }
        }
        // 题目返回的是int类型的数组，这里要进行转换
        int[] finalAns = new int[ans.size()];
        for (int i = 0; i < finalAns.length; i++) {
            finalAns[i] = ans.get(i);
        }
        return finalAns;
    }
}
```

## 32.II.从上到下打印二叉树 II

​		[32.II.从上到下打印二叉树 II](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)	难度：easy

​		是上题的变种，区别在于这里需要将二叉树分层存入数组中，只需要在上题的基础上加一个for循环存入每层的节点。

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        // 树为空
        if (root == null) {
            List<List<Integer>> e = new ArrayList<>();
            return e;
        }
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        List<List<Integer>> ans = new ArrayList<>();
        queue.add(root);
        while (!queue.isEmpty()) {
            // temp用于存每层的所有节点
            List<Integer> temp = new ArrayList<>();
            // for循环写自减而不是自增，因为循环里有pop操作，队列的大小会变
            for (int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                temp.add(node.val);
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
            }
            // 将每层的节点再放进二维数组中
            ans.add(temp);
        }
        // 直接返回二维数组就好
        return ans;
    }
}
```

## 32.III.从上到下打印二叉树 III

​		[32.III.从上到下打印二叉树 III](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)	难度：medium

​		和上题的区别就是这里需要反向存储，奇数行存储正向的层节点，偶数行存储反向的层节点，所以在层数时进行特判就可以了。

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) {
            List<List<Integer>> e = new ArrayList<>();
            return e;
        }
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        List<List<Integer>> ans = new ArrayList<>();
        // row代表当前的行数
        int row = 1;
        queue.add(root);
        while (!queue.isEmpty()) {
            List<Integer> temp = new ArrayList<>();
            for (int i = queue.size(); i > 0; i--) {
                TreeNode node = queue.poll();
                temp.add(node.val);
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
            }
            // 如果是偶数行就反转数组
            if (row % 2 == 0) {
                Collections.reverse(temp);
            }
            ans.add(temp);
            // 行数递增
            row++;
        }
        return ans;
    }
}
```

## 33.二叉搜索树的后序遍历序列

​		[33.二叉搜索树的后序遍历序列](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)	难度：medium

​		二叉搜索树的性质是，对于二叉树的任意一个节点，它的左节点小于它本身，右节点大于它本身，因此对于根节点来说，它的左子树都是二叉搜索树，右子树也都是二叉搜索树。

​		后序遍历的顺序为“左节点，右节点，根节点”。

​		因此对于一个二叉搜索树的后序遍历序列来说，有两点是确定的：

* 序列最后一个值一定是根节点的值
* 对序列从前往后遍历，第一个大于根节点的值一定是根节点的右子节点，此时这个点左边的节点都是根节点的左子树，右边的值（包括本身）都是根节点的右子树

​		由此就可以想到用递归分治的思路解决：

```java
class Solution {
    public boolean verifyPostorder(int[] postorder) {
        return solve(postorder, 0, postorder.length - 1);
    }
    public boolean solve(int[] postorder, int left, int right) {
        // 递归的终止条件是left >= right，代表着只有一个或者没有节点了
        if (left >= right) {
            return true;
        }
        // 从头开始遍历数组，找到第一个大于根节点的点
        int pointer = left;
        while (postorder[pointer] < postorder[right]) {
            pointer++;
        }
        // 找到以后要验证该点后面的值是否都大于根节点，保证这是一颗二叉搜索树
        int temp = pointer;
        while (temp < right) {
            if (postorder[temp] < postorder[right]) {
                return false;
            }
            temp++;
        }
        // 继续递归根的左子树和右子树
        return solve(postorder, left, pointer - 1) && solve(postorder, pointer, right - 1);
    }
}
```

​		题解区有关于辅助单调栈的解法，可以将时间复杂度由$O(N)$降为$O(N^2)$，但单调栈的方法不易理解，在题解区有一种十分容易理解的方法：仿照序列构建一颗二叉搜索树，当然不用真正的建树，只需要判断是否符合二叉搜索树的结构：

```java
class Solution {
    // cnt用于统计还剩多少个节点没放进树中
    int cnt = 0;
    public boolean verifyPostorder(int[] postorder) {
        if (postorder == null || postorder.length == 1){
            return true;
        }
        // 初始为数组长度，-1是为了下标
        cnt = postorder.length - 1;
        // 开始递归建树
        build(postorder, Integer.MIN_VALUE, Integer.MAX_VALUE);
        // 如果放完了说明这就是二叉搜索树
        if (cnt < 0) {
            return true;
        }else {
            return false;
        }
    }
    public void build(int[] postorder, int min, int max) {
        // 如果放完了就返回
        if (cnt < 0) {
            return ;
        }
        // 取数组最后一个值作为根
        int root = postorder[cnt];
        if (root >= max || root <= min) {// 不在上下界范围内直接返回
            return ;
        }
        // 都符合条件就减少
        cnt--;
        build(postorder, root, max);	// 建右子树
        build(postorder, min, root);	// 建左子树
        // root在不断刷新递归的上下界，达到建树目的
    }
}
```

## 34.二叉树中和为某一值的路径

​		[34.二叉树中和为某一值的路径](https://leetcode.cn/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)	难度：medium

​		DFS+回溯，从二叉树的根节点开始深度遍历，找到和为目标值的路径，递归结束的路径为节点为空（已经到了叶子节点）。

```java
class Solution {
    // 装多条路径的最后答案
    List<List<Integer>> ans = new ArrayList<>();
    // 用来存储深搜路径
    List<Integer> path = new ArrayList<Integer>();
    public List<List<Integer>> pathSum(TreeNode root, int target) {
        dfs(root, target);
        return ans;
    }
    public void dfs(TreeNode root, int target) {
        // 为空说明上一层递归已经到了叶子节点
        if (root == null) {
            return ;
        }
        // 路径中加入当前点
        path.add(root.val);
        target -= root.val;
        // 符合和为目标值且为叶子节点的路径
        if (target == 0 && root.left == null && root.right == null) {
            // 不能直接ans.add(path)，这样是加入了一个对象，但是随着后面的remove，path是会改变的
            // 所以new ArrayList(path)相当于复制了一个数组进去
            ans.add(new ArrayList(path));
            // 为什么这里不直接返回
            // 提前返回会导致后面的move没有执行，影响回溯结果
            // 相当于找新路径时前面的点还没删干净，所以可能会生成一条不可能出现的路径
        }
        dfs(root.left, target);
        dfs(root.right, target);
        // 删掉数组中最后一个点回溯
        path.remove(path.size() - 1);
    }
}
```

