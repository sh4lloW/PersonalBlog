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

### 10.II.青蛙跳台阶问题
&emsp;&emsp;[10.II.青蛙跳台阶问题](https://leetcode.cn/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)&emsp;难度：easy\
&emsp;&emsp;经典的DP问题，实际也就是斐波那契数列问题，解法与上题一致。\
&emsp;&emsp;但值得注意的是这题默认fib(0)没有意义，在0阶台阶时也会有一种解法，所以dp[0]=1。

### 11.旋转数组的最小数字
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

### 12.矩阵中的路径
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

### 14.I.剪绳子
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

### 14.II.剪绳子II
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

### 15.二进制中1的个数
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

### 16.数值的整数次方
&emsp;&emsp;[16.数值的整数次方](https://leetcode.cn/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)&emsp;难度：medium\
&emsp;&emsp;快速幂算法模板题：
* $x^n$ = $(x^2)^\frac{n}{2}$
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
&emsp;&emsp;本质是二分法，所以时间复杂度为$O(logN)$。

### 17.打印从1到最大的n位数
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

### 18.删除链表的结点
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

