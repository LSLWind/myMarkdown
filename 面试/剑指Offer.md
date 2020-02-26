### 03数组中重复的数字

找出数组中重复的数字。


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

```
示例 1：

输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```

```java
class Solution {
    public int findRepeatNumber(int[] nums) {
        Set<Integer>s=new HashSet<>();
        for(int num:nums){
            if(s.contains(num))return num;
            else s.add(num);
        }
        return 0;
    }
}
```

### 04二维数组中的查找

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

 ```
示例:
现有矩阵 matrix 如下：
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
给定 target = 5，返回 true。
给定 target = 20，返回 false。
 ```

从左上角向下看其实是波形增大的，如果从左上角开始走，则target>m[x][y\]时就有两种走法，而从右上角开始则只有一种走法了，从右上角开始，只有两种选择了，大于向下判断，小于向左判断

```java
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if(matrix==null||matrix.length==0||matrix[0].length==0)return false;
        if(target<matrix[0][0]||target>matrix[matrix.length-1][matrix[0].length-1])return false;
        //从右上角开始，只有两种选择了，大于向下判断，小于向左判断
        return dfs(matrix,target,0,matrix[0].length-1);
    }
    private boolean dfs(int[][] m,int target,int x,int y){
        if(x<0||x>=m.length||y<0||y>=m[0].length)return false;//超出范围
        if(target==m[x][y])return true;
        if(target>m[x][y])return dfs(m,target,x+1,y);
        else return dfs(m,target,x,y-1);
    }
}
```

### 05替换空格

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

 ```
示例 1：
输入：s = "We are happy."
输出："We%20are%20happy."
 ```

```java
class Solution {
    public String replaceSpace(String s) {
        return s.replace(" ","%20");
    }
}
```

String不可变，按照替换思路，使用StringBuilder或者直接使用char[]应该更好

### 06从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

**示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

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
        List<Integer>s=new ArrayList<>();
        while(head!=null){
            s.add(head.val);
            head=head.next;
        }
        int[] res=new int[s.size()];
        int ite=0;
        for(int i=s.size()-1;i>=0;i--)res[ite++]=s.get(i);
        return res;
    }
}
 ```

### 07重建二叉树

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

```
例如，给出
前序遍历 preorder = [3,9,20,15,7]
中序遍历 inorder = [9,3,15,20,7]
返回如下的二叉树：
    3
   / \
  9  20
    /  \
   15   7
```

逆递归遍历即可

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
        if(preorder==null||preorder.length==0)return null;
        
        int flag=preorder[0];
        TreeNode root=new TreeNode(flag);
        for(int i=0;i<inorder.length;i++){
            if(inorder[i]==flag){
                root.left=buildTree(Arrays.copyOfRange(preorder,1,i+1),Arrays.copyOfRange(inorder,0,i));           root.right=buildTree(Arrays.copyOfRange(preorder,i+1,preorder.length),Arrays.copyOfRange(inorder,i+1,inorder.length));
                break;
            }
        }
        return root;
    }
}
```

### 09用两个栈实现队列

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 appendTail 和 deleteHead ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，deleteHead 操作返回 -1 )

 ```
示例 1：
输入：
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]
 ```

题目本身很简单，考虑两种数据结构的基础概念与使用方式

```java
class CQueue {
    Stack<Integer>a;
    Stack<Integer>b;

    public CQueue() {
        a=new Stack<>();
        b=new Stack<>();
    }
    
    public void appendTail(int value) {
        a.push(value);
    }
    
    public int deleteHead() {
        if(a.empty())return -1;
        while(!a.empty()){
            b.push(a.pop());
        }
        int res=b.pop();
        while(!b.empty()){
            a.push(b.pop());
        }
        return res;
    }
}
/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int paam_2 = obj.deleteHead();
 */
```

### 10-1 斐波那切数列

写一个函数，输入 n ，求斐波那契（Fibonacci）数列的第 n 项。斐波那契数列的定义如下：

F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

 ```
示例 1：
输入：n = 2
输出：1

示例 2：
输入：n = 5
输出：5
 ```

直接递归会造成很多不必要的计算，使用备忘录存储已计算值

```java
class Solution {
    long[] s=new long[101];//s[i]表示s[i]的斐波那契数列值
    {
        s[0]=0;
        s[1]=1;
    }
    public int fib(int n) {
        if(n==0||n==1)return n;
        if(s[n]!=0)return (int)s[n];//有记录则直接返回
        
        long res=((long)fib(n-1)+(long)fib(n-2))%1000000007;
        s[n]=res;
        //System.out.println(res);
        return (int)res;
    }
}
```

### 10-2 青蛙跳台阶问题

一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

```
示例 1：
输入：n = 0
输出：1
示例 2：
输入：n = 7
输出：21
```

斐波那契数列的应用，还是使用备忘录进行记录

```java
class Solution {
    int[] f=new int[101];
    {
        f[0]=1;
        f[1]=1;
        f[2]=2;
    }
    public int numWays(int n) {
        //f(n)=f(n-1)+f(n-2)
        if(f[n]!=0)return f[n];
        f[n]=(numWays(n-1)+numWays(n-2))%1000000007;
        return f[n];
    }
}
```

### 11旋转数组的最小数字

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 [3,4,5,1,2] 为 [1,2,3,4,5] 的一个旋转，该数组的最小值为1。  

```
示例 1：
输入：[3,4,5,1,2]
输出：1
示例 2：
输入：[2,2,2,0,1]
输出：0
```

```java
class Solution {
    public int minArray(int[] numbers) {
        if(numbers==null||numbers.length==0)return 0;

        for(int i=numbers.length-1;i>0;i--){
            if(numbers[i-1]>numbers[i])return numbers[i];
        }
        return numbers[0];
    }
}
```

### 12 矩阵中的路径

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的3×4的矩阵中包含一条字符串“bfce”的路径（路径中的字母用加粗标出）。

[["a","b","c","e"],
["s","f","c","s"],
["a","d","e","e"]]

但矩阵中不包含字符串“abfb”的路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

 ```
示例 1：
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
示例 2：
输入：board = [["a","b"],["c","d"]], word = "abcd"
输出：false
 ```

使用bt，应该没有什么更简便的算法了

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        if(board==null||board.length==0||board[0].length==0)return false;
        if(word==null||word.equals(""))return false;

        for(int i=0;i<board.length;i++){
            for(int j=0;j<board[i].length;j++){
                if(board[i][j]==word.charAt(0)){
                    if(bt(board,word,i,j))return true;
                }
            }
        }
        return false;
    }
    private boolean bt(char[][] board,String word,int x,int y){
        if(word==null||word.equals(""))return true;
        if(x<0||x>=board.length||y<0||y>=board[0].length||board[x][y]==' ')return false; 
        if(board[x][y]!=word.charAt(0))return false;
        //继续递归
        char tmp=board[x][y];
        board[x][y]=' ';//' '表示该路径已走过
        if(bt(board,word.substring(1),x-1,y))return true;
        if(bt(board,word.substring(1),x+1,y))return true;
        if(bt(board,word.substring(1),x,y-1))return true;
        if(bt(board,word.substring(1),x,y+1))return true;
        board[x][y]=tmp;
        return false;
    } 
}
```

### 14-I 剪绳子-1

给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m≥1），每段绳子的长度记为 k[0],k[1]...k[m] 。请问 k[0]\*k[1]*...*k[m] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

```
示例 1：
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1

示例 2:
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36
```

```java
class Solution {
    int[] s=new int[100];//s[i]代表i能拆分的最大值
    //初始化,s实际上就是备忘录
    {
        s[1]=1;
        s[2]=1;
        s[3]=2;
        s[4]=4;
        s[5]=6;
        s[6]=9;
    }
    public int cuttingRope(int n) {
        if(s[n]!=0)return s[n];
        
        int res=n-1;
        //从[1,n-1]这个范围递归遍历
        for(int i=1;i<n;i++){
            s[n-i]=cuttingRope(n-i);//保存记录，递归相乘
            res=Math.max(res,i*s[n-i]);
        }
        return res;
    }
}
```

### 14-II 剪绳子-II

同14-I相同，只不过这回n的数字范围很大，2<=n<=1000，同时 答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。 

此时使用同14-I的解法，那么int和long都会溢出，可以使用BigInteger

```java
import java.math.BigInteger;
class Solution {
    BigInteger[] s=new BigInteger[1001];//s[i]代表i能拆分的最大值
    //初始化,s实际上就是备忘录
    {
        s[1]=new BigInteger("1");
        s[2]=new BigInteger("1");
        s[3]=new BigInteger("2");
        s[4]=new BigInteger("4");
        s[5]=new BigInteger("6");
        s[6]=new BigInteger("9");
    }
    private BigInteger lsl(int n){
        if(s[n]!=null)return s[n];
        
        BigInteger res=new BigInteger(String.valueOf(n-1));
        for(int i=1;i<n;i++){
            s[n-i]=lsl(n-i);
            BigInteger tmp=s[n-i].multiply(new BigInteger(String.valueOf(i)));
            res=res.max(tmp);
        }
        return res;
    }
    public int cuttingRope(int n) {
        return Integer.valueOf(lsl(n).mod(new BigInteger("1000000007")).toString());
    }
    
}
```

### 15二进制中1的个数

请实现一个函数，输入一个整数，输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。

```
示例 1：
输入：00000000000000000000000000001011
输出：3
解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。
示例 2：
输入：00000000000000000000000010000000
输出：1
解释：输入的二进制串 00000000000000000000000010000000 中，共有一位为 '1'。
```

二进制基本运算，需要注意的是有符号移位为>>符号，无符号移位为>>>

```java
public class Solution {
    public int hammingWeight(int n) {
        int res=0;
        while(n!=0){
            if((n&1)==1)res++;
            n>>>=1;
        }
        return res;
    }
}
```

### 16数值的整数次方

实现函数double Power(double base, int exponent)，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

 ```
示例 1:
输入: 2.00000, 10
输出: 1024.00000
示例 2:
输入: 2.10000, 3
输出: 9.26100
 ```

```java
class Solution {
    public double myPow(double x, int n) {
        //快速求次方，将指数化为2进制进行快速运算
        boolean flag=n>0?true:false;
        n=Math.abs(n);
        return flag?pow(x,n):1/pow(x,n);
    }
    private double pow(double x,int n){
        if(n==0)return 1;
        if(n==1)return x;
        if(n==2)return x*x;
        return n%2==0?pow(x*x,n/2):x*pow(x*x,n/2);
    }
}
```

### 32-1从上到下打印二叉树

从上到下打印出二叉树的每个节点，同一层的节点按照从左到右的顺序打印。

```
例如:给定二叉树: `[3,9,20,null,null,15,7]`,
    3
   / \
  9  20
    /  \
   15   7
返回：
[3,9,20,15,7]
```

基础bfs

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
    public int[] levelOrder(TreeNode root) {
        if(root==null)return new int[0];
        
        List<TreeNode>nodes=new ArrayList<>();
        nodes.add(root);
        List<Integer> res=printBinaryTree(nodes,new ArrayList<Integer>());
        int[] realRes=new int[res.size()];
        for(int i=0;i<res.size();i++)realRes[i]=res.get(i);
        return realRes;
    }
    private List<Integer> printBinaryTree(List<TreeNode>nodes,List<Integer> res){
        if(nodes.size()==0)return res;
        List<TreeNode>next=new ArrayList<>();
        for(TreeNode node:nodes){
            res.add(node.val);
            if(node.left!=null)next.add(node.left);
            if(node.right!=null)next.add(node.right);
        }
        return printBinaryTree(next,res);
    }
}
```

### 32 - II 从上到下打印二叉树 II

从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。

```
例如:给定二叉树: `[3,9,20,null,null,15,7]`, 
    3
   / \
  9  20
    /  \
   15   7
返回其层次遍历结果：
[
  [3],
  [9,20],
  [15,7]
]
```

同上，基础bfs

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>>res=new ArrayList<>();
        if(root==null)return res;
        List<TreeNode>nodes=new ArrayList<>();
        nodes.add(root);
        return bfs(nodes,res);
    }
    private List<List<Integer>> bfs(List<TreeNode> nodes,List<List<Integer>> res){
        if(nodes.size()==0)return res;
        List<TreeNode> tmp=new ArrayList<>();
        List<Integer> val=new ArrayList<>();
        for(TreeNode node:nodes){
            val.add(node.val);
            if(node.left!=null)tmp.add(node.left);
            if(node.right!=null)tmp.add(node.right);
        }
        res.add(val);
        return bfs(tmp,res);
    }
}
```

### 32 - III. 从上到下打印二叉树 III

请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

```
例如:给定二叉树: `[3,9,20,null,null,15,7]`, 
    3
   / \
  9  20
    /  \
   15   7
[
  [3],
  [20,9],
  [15,7]
]
```

 还是基础bfs，只不过加个判断标志

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>>res=new ArrayList<>();
        if(root==null)return res;
        
        List<TreeNode>nodes=new ArrayList<>();
        nodes.add(root);
        return bfs(nodes,res,true);
    }
    private List<List<Integer>> bfs(List<TreeNode> nodes,List<List<Integer>> res,boolean flag){
        if(nodes.size()==0)return res;
        
        List<TreeNode> tmp=new ArrayList<>();
        List<Integer> val=new ArrayList<>();
        //下一层
        for(TreeNode node:nodes){
            if(node.left!=null)tmp.add(node.left);
            if(node.right!=null)tmp.add(node.right);
        }
        //当前层取值
        if(flag){
            for(int i=0;i<nodes.size();i++)val.add(nodes.get(i).val);
        }else{
            for(int i=nodes.size()-1;i>=0;i--)val.add(nodes.get(i).val);
        }
        res.add(val);
        return bfs(tmp,res,!flag);
    }
}
```

### 59 - I. 滑动窗口的最大值

给定一个数组 `nums` 和滑动窗口的大小 `k`，请找出所有滑动窗口里的最大值。

**示例:**

```
输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7] 
解释: 

  滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**提示：**

你可以假设 *k* 总是有效的，在输入数组不为空的情况下，1 ≤ k ≤ 输入数组的大小。

维护一个maxIndex，即最大值索引，每次滑动时只要比较其余最右边的值即可，如果索引不在范围则需要重新更新索引，对于从小到大有序的数组，这个算法则变成了O(n^2)

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums==null||nums.length<=1)return nums;
        
        int maxIndex=0;
        int[] res=new int[nums.length-k+1];
        int left=0;
            
        //初始化
        for(int i=0;i<k;i++)maxIndex=nums[maxIndex]>nums[i]?maxIndex:i;
        res[left++]=nums[maxIndex];
        
        for(int right=k;right<nums.length;right++){
            if(left<=maxIndex&&maxIndex<=right){
                maxIndex=nums[maxIndex]>nums[right]?maxIndex:right;
            }else{
                maxIndex=left;
                for(int i=left;i<=right;i++)maxIndex=nums[maxIndex]>nums[i]?maxIndex:i;
            }
            res[left++]=nums[maxIndex];
        }

        return res;
    }
}
```

### 59 - II. 队列的最大值

请定义一个队列并实现函数 `max_value` 得到队列里的最大值，要求函数`max_value`、`push_back` 和 `pop_front` 的时间复杂度都是O(1)。

若队列为空，`pop_front` 和 `max_value` 需要返回 -1

**示例 1：**

```
输入: 
["MaxQueue","push_back","push_back","max_value","pop_front","max_value"]
[[],[1],[2],[],[],[]]
输出: [null,null,null,2,1,2]
```

**示例 2：**

```
输入: 
["MaxQueue","pop_front","max_value"]
[[],[],[]]
输出: [null,-1,-1]
```

要维护一个最大值，保证最大值为O(1)的话，入队或者出队的时候总有一个是达不到O(1)的

```java
class MaxQueue {

    private Node max;
    private Node root;
    private Node tail;
    class Node{
        int val;
        Node next=null;
        public Node(int val){this.val=val;}
    }
    
    public MaxQueue() {
        this.root=new Node(-1);
        root.next=null;
        this.max=root;
        this.tail=root;
    }
    
    public int max_value() {
        if(root==tail)return -1;
        return max.val;

    }
    
    public void push_back(int value) {
        Node tmp=new Node(value);
        tail.next=tmp;
        tail=tmp;

        if(max==root||max.val<=value)max=tmp;
    }
    
    public int pop_front() {
        if(tail==root)return -1;
        
        root=root.next;//root并不代表实际节点
        if(root==max){//最大值要出去了，重新更新max
            Node head=root.next;//head-tail才代表实际的队列
            max=head;
            while(head!=null){
                max=max.val<=head.val?head:max;
                head=head.next;
            }
        }
        max=(max==null)?root:max;
        
        return root.val;
    }
}

/**
 * Your MaxQueue object will be instantiated and called as such:
 * MaxQueue obj = new MaxQueue();
 * int param_1 = obj.max_value();
 * obj.push_back(value);
 * int param_3 = obj.pop_front();
 */
```

### 60 n个骰子的点数

把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。

你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。

**示例 1:**

```
输入: 1
输出: [0.16667,0.16667,0.16667,0.16667,0.16667,0.16667]
```

**示例 2:**

```
输入: 2
输出: [0.02778,0.05556,0.08333,0.11111,0.13889,0.16667,0.13889,0.11111,0.08333,0.05556,0.02778]
```

**限制：**

```
1 <= n <= 11
```

典型的递归求解，每一次都记录中间值，层层推进求解

```java
class Solution {
    private static double BASE=1.0/6.0;
    public double[] twoSum(int n) {
        Map<Integer,Double>s=new TreeMap<>();
        //初始化
        for(int i=1;i<=6;i++){
            s.put(i,BASE);
        }
        
        for(int i=1;i<n;i++){
             Map<Integer,Double>tmp=new TreeMap<>();
            for(int j=1;j<=6;j++){
                for(int count:s.keySet()){
                    tmp.put(j+count,tmp.getOrDefault(j+count,0.0)+s.get(count)*BASE);
                }
            }
            s=tmp;
        }
        double[] res=new double[s.size()];
        int ite=0;
        for(int count:s.keySet())res[ite++]=s.get(count);
        return res;
    }
}
```

### 65不用加减乘除做加法

写一个函数，求两个整数之和，要求在函数体内不得使用 “+”、“-”、“*”、“/” 四则运算符号。 

**示例:**

```
输入: a = 1, b = 1
输出: 2
```

**提示：**

- `a`, `b` 均可能是负数或 0
- 结果不会溢出 32 位整数

这题位运算还是背下来吧，毕竟位运算这种模拟加法用法基本就这题，很容易就忘掉。。。

^ 亦或 ----相当于无进位的求和， 想象10进制下的模拟情况：（如:19+1=20；无进位求和就是10，而非20；因为它不管进位情况）

& 与 ----相当于求每位的进位数， 先看定义：1&1=1；1&0=0；0&0=0；即都为1的时候才为1，正好可以模拟进位数的情况,还是想象10进制下模拟情况：（9+1=10，如果是用&的思路来处理，则9+1得到的进位数为1，而不是10，所以要用<<1向左再移动一位，这样就变为10了）；

这样公式就是：a+b=（a^b) + ((a&b)<<1) 即：每次无进位只+ 每次得到的进位数--------我们需要不断重复这个过程，直到进位数为0为止（进位数为0则直接返回a^b就好）；

具体的，按照a+b=（a^b) + ((a&b)<<1) 公式，很容易写出递归式

```java
class Solution {
    public int add(int a, int b) {
        return b==0?a:add(a^b,(a&b)<<1);
    }
}
```

### 66构建乘积数组

给定一个数组 `A[0,1,…,n-1]`，请构建一个数组 `B[0,1,…,n-1]`，其中 `B` 中的元素 `B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]`。不能使用除法。

**示例:**

```
输入: [1,2,3,4,5]
输出: [120,60,40,30,24]
```

**提示：**

- 所有元素乘积之和不会溢出 32 位整数
- `a.length <= 100000`

最初想用C[i][j\]存储从i到j的总乘积，不过超过了内存

```java
class Solution {
    public int[] constructArr(int[] a) {
        if(a==null||a.length==0)return new int[0];
        if(a.length==1)return a;
        int[][] C=new int[a.length][a.length];
        //从上到下打表
        for(int i=a.length-1;i>=0;i--){
            for(int j=i;j>=0;j--){
                if(i==j)C[j][i]=a[j];
                else C[j][i]=a[j]*C[j+1][i];
            }
        }
       int[] res=new int[a.length];
        res[0]=C[1][a.length-1];
        res[a.length-1]=C[0][a.length-2];
        for(int i=1;i<a.length-1;i++){
            res[i]=C[0][i-1]*C[i+1][a.length-1];
        }
        return res;
    }
}
```

另一个更巧妙的方法是，构建前后缀乘积，也就是对整个B，先算前面的乘积（因为这部分是连续相乘的，不用构建二维数组），同理，在构造后面的乘积（分两次计算）

```java
class Solution {
    public int[] constructArr(int[] a) {
        if(a==null||a.length==0)return new int[0];
        if(a.length==1)return a;
        
        int[] res=new int[a.length];
        //基准乘数为1
        for(int i=0;i<a.length;i++)res[i]=1;
        
        int left=1;
        for(int i=0;i<a.length;i++){
            res[i]*=left;
            left*=a[i];
        }
        int right=1;
        for(int i=a.length-1;i>=0;i--){
            res[i]*=right;
            right*=a[i];
        }
        
        return res;
    }
}
```

### 67. 把字符串转换成整数

写一个函数 StrToInt，实现把字符串转换成整数这个功能。不能使用 atoi 或者其他类似的库函数。 

首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。

当我们寻找到的第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字组合起来，作为该整数的正负号；假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成整数。

该字符串除了有效的整数部分之后也可能会存在多余的字符，这些字符可以被忽略，它们对于函数不应该造成影响。

注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换。

在任何情况下，若函数不能进行有效的转换时，请返回 0。

**说明：**

假设我们的环境只能存储 32 位大小的有符号整数，那么其数值范围为 [−231, 231 − 1]。如果数值超过这个范围，请返回  INT_MAX (231 − 1) 或 INT_MIN (−231) 。

**示例 1:**

```
输入: "42"
输出: 42
```

**示例 2:**

```
输入: "   -42"
输出: -42
解释: 第一个非空白字符为 '-', 它是一个负号。
     我们尽可能将负号与后面所有连续出现的数字组合起来，最后得到 -42 。
```

**示例 3:**

```
输入: "4193 with words"
输出: 4193
解释: 转换截止于数字 '3' ，因为它的下一个字符不为数字。
```

**示例 4:**

```
输入: "words and 987"
输出: 0
解释: 第一个非空字符是 'w', 但它不是数字或正、负号。
     因此无法执行有效的转换。
```

**示例 5:**

```
输入: "-91283472332"
输出: -2147483648
解释: 数字 "-91283472332" 超过 32 位有符号整数范围。 
     因此返回 INT_MIN (−231) 。
```

较难处理的是溢出判断，要么使用更高精度的数，要么使用溢出判断

```java
class Solution {
    public int strToInt(String str) {
        if(str==null||str.equals(""))return 0;
        str=str.trim();
        if(str.equals(""))return 0;
        
        char flag=str.charAt(0)=='-'?'-':'+';
        int begin=(str.charAt(0)=='-'||str.charAt(0)=='+')?1:0;
        long res=0;
        for(int i=begin;i<str.length();i++){
            if('0'<=str.charAt(i)&&str.charAt(i)<='9'){
                res=res*10+str.charAt(i)-48;
                if(flag=='+'&&res>Integer.MAX_VALUE)return Integer.MAX_VALUE;
                if(flag=='-'&&-res<Integer.MIN_VALUE)return Integer.MIN_VALUE;
            }else break;
        }
        return flag=='-'?(int)-res:(int)res;
    }
}
```

