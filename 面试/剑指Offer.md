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

### 49. 丑数

我们把只包含因子 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。 

**示例:**

```
输入: n = 10
输出: 12
解释: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 是前 10 个丑数。
```

三指针迭代，丑数来源于基准数 \*  2 ,3,5，唯一的难点在于如何去重，换一种角度，相当于有三个基础队列

 A：{1\*2，2\*2，3\*2，4\*2，5*2，6\*2，8*2，10\*2......}

B：{1\*3，2\*3，3\*3，4\*3，5\*3，6\*3，8\*3，10\*3......}

C：{1\*5，2\*5，3\*5，4\*5，5\*5，6\*5，8*5，10\*5......}

三个队列合并构成丑数队列，难点在于如何去重，假设有三个指针,i,j,k分别指向三个队列，那么每次选择丑数队列中的最小的一个指针位置作为基准数分别乘以2,3,5，那么就可以选出当前最小的丑数加入丑数队列，同时更新指针

```java
class Solution {
    public int nthUglyNumber(int n) {
        if(n==1)return 1;
        
        int[] ugly=new int[n+1];//丑数队列
        ugly[1]=1;//第一个丑数
        int i=1;
        int j=1;
        int k=1;
        for(int ite=2;ite<=n;ite++){//三指针迭代获取当前丑数
            int tmp=Math.min(ugly[i]*2,Math.min(ugly[j]*3,ugly[k]*5));//选出当前最小丑数
            //指针更新
            if(tmp == ugly[i]*2)i++;
            if(tmp == ugly[j]*3)j++;
            if(tmp == ugly[k]*5)k++;
            ugly[ite]=tmp;//加入丑数队列
        }
        
        return ugly[n];
    }

}
```

### 55 - I. 二叉树的深度

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

例如：

给定二叉树 `[3,9,20,null,null,15,7]`，

```
    3
   / \
  9  20
    /  \
   15   7
```

返回它的最大深度 3 。

简单递归题

```java
class Solution {
    public int maxDepth(TreeNode root) {
        if(root==null)return 0;
        return Math.max(1+maxDepth(root.left),1+maxDepth(root.right));
    }
}
```

### 55 - II. 平衡二叉树

输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

**示例 1:**

给定二叉树 `[3,9,20,null,null,15,7]`

```
    3
   / \
  9  20
    /  \
   15   7
```

返回 `true` 。

**示例 2:**

给定二叉树 `[1,2,2,3,3,null,null,4,4]`

```
       1
      / \
     2   2
    / \
   3   3
  / \
 4   4
```

返回 `false` 。

递归计算层数，递归的过程中判断是否是平衡二叉树

```java
class Solution {
    static boolean flag;
    public boolean isBalanced(TreeNode root) {
        flag=true;
        int res=dfs(root);
        return flag;
    }
    
    public int dfs(TreeNode root){
        if(root==null)return 0;
        int left=dfs(root.left);
        int right=dfs(root.right);
        if(Math.abs(left-right)>1)flag=false;
        return 1+Math.max(left,right);
    }
}
```

### 56 - I 数组中数字出现的次数

一个整型数组 `nums` 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

**示例 1：**

```
输入：nums = [4,1,4,6]
输出：[1,6] 或 [6,1]
```

**示例 2：**

```
输入：nums = [1,2,10,4,1,4,3,3]
输出：[2,10] 或 [10,2]
```

二进制运算问题，其它数字出现两次，只有两个不同的数字，利用\^与&的性质，先异或一遍数组，a=b\^c，b与c就是不同的数字，按照二进制规律，b与c至少必有一位是不同的，因此随便找出一个1的位置，指定一个数，只有这个位是1，其余都是0，根据这个数可以将数组分成两组，这个位为1的是一组，为0的是一组，可以保证两组只有一个不同的数组，其余都是两两相同的数字（使用&）

```java
class Solution {
    public int[] singleNumbers(int[] nums) {
        int x=0;
        for(int num:nums)x^=num;
        
        //找到第一个异或值为1的位置
        int med=1;
        while(x!=0){
            if((x&1)==1){
                break;
            }
            x>>=1;
            med<<=1;
        }
        
        int[] res=new int[2];
        //利用&操作识别1的位置
        for(int num:nums){
            if((med&num)==med){
                res[0]^=num;
            }else{
                res[1]^=num;
            }
        }
        
        return res;
    }
}
```

### 56 - II. 数组中数字出现的次数 II

在一个数组 `nums` 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

**示例 1：**

```
输入：nums = [3,4,3,3]
输出：4
```

**示例 2：**

```
输入：nums = [9,1,7,9,7,9,7]
输出：1
```

实在想不到二进制解法，有用二进制+状态机进行求解的。此处使用容易想到的排序解法

```java
class Solution {
    public int singleNumber(int[] nums) {
        Arrays.sort(nums);
        for(int i=0;i<nums.length-1;i++){
            if(nums[i]==nums[i+1])i+=2;
            else return nums[i];
        }
        return nums[nums.length-1];
    }
}
```

### 57. 和为s的两个数字

输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。

**示例 1：**

```
输入：nums = [2,7,11,15], target = 9
输出：[2,7] 或者 [7,2]
```

**示例 2：**

```
输入：nums = [10,26,30,31,47,60], target = 40
输出：[10,30] 或者 [30,10]
```

递增数组，常规二分搜索

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        //二分查找
        int[] res=new int[2];
        if(nums==null||nums.length<=1)return res;
        
        for(int i=0;i<nums.length-1;i++){
            int search=Arrays.binarySearch(nums,i+1,nums.length,target-nums[i]);
            if(search>=0){
                res[0]=nums[i];
                res[1]=target-nums[i];
                return res;
            }
        }
        
        return res;
    }
}
```

### 57 - II. 和为s的连续正数序列

输入一个正整数 `target` ，输出所有和为 `target` 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

**示例 1：**

```
输入：target = 9
输出：[[2,3,4],[4,5]]
```

**示例 2：**

```
输入：target = 15
输出：[[1,2,3,4,5],[4,5,6],[7,8]]
```

使用队列（实际上就是滑动窗口）的方式，维护一个顺序正整数序列，每次判断该数加上去是否符合条件即可，值得一提的是返回二维数组可以使用List<int[]>这种方式

```java
class Solution {
    public int[][] findContinuousSequence(int target) {
        //以这种方式转为二维数组
        List<int[]> res=new ArrayList<>();
        
        int tmp=0;//累加量
        List<Integer> s=new ArrayList<>();//数字队列
        
        for(int i=target/2+1;i>=1;i--){
            if(tmp<target){
                tmp+=i;
                s.add(i);//加入队列
            }
            if(tmp>target){//移除头部最大元素
                tmp-=s.get(0);
                s.remove(0);//移除头部最大元素                
            }
            if(tmp==target){
                int[] oneRes=new int[s.size()];
                int ite=0;
                for(int j=s.size()-1;j>=0;j--){
                    oneRes[ite++]=s.get(j);
                }
                res.add(oneRes);//连续正整数序列
                tmp-=s.get(0);
                s.remove(0);//移除头部最大元素
            }
        }
        
        int[][] realRes=new int[res.size()][];
        int ite=0;
        for(int i=res.size()-1;i>=0;i--){
            realRes[ite++]=res.get(i);
        }
        
        return realRes;

    }
}
```

### 58 - I. 翻转单词顺序

输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student. "，则输出"student. a am I"。

**示例 1：**

```
输入: "the sky is blue"
输出: "blue is sky the"
```

**示例 2：**

```
输入: "  hello world!  "
输出: "world! hello"
解释: 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
```

**示例 3：**

```
输入: "a good   example"
输出: "example good a"
解释: 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
```

设置标志位进行翻转匹配，O(n)

```java
class Solution {
    public String reverseWords(String s) {
        if(s==null||s.equals(""))return "";
        
        StringBuilder res=new StringBuilder();

        int r=-1;
        for(int i=s.length()-1;i>=0;i--){
            if(s.charAt(i)!=' '){
                if(r==-1){//不在字段内则设置
                    r=i;
                }
            }else{
                if(r!=-1){//l到达空格位置
                    res.append(s.substring(i+1,r+1)).append(" ");
                    r=-1;
                }
            }
        }
        if(r!=-1){//最左侧加一次
            res.append(s.substring(0,r+1).trim());
        }
        
        return res.toString().trim();
    }
}
```

### 58 - II. 左旋转字符串

字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

**示例 1：**

```
输入: s = "abcdefg", k = 2
输出: "cdefgab"
```

**示例 2：**

```
输入: s = "lrloseumgh", k = 6
输出: "umghlrlose"
```

```java
class Solution {
    public String reverseLeftWords(String s, int n) {
        if(n==s.length())return s;
        
        return s.substring(n)+s.substring(0,n);
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

### 61 扑克牌中的顺子

从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

**示例 1:**

```
输入: [1,2,3,4,5]
输出: True
```

**示例 2:**

```
输入: [0,0,1,2,5]
输出: True
```

简单逻辑思维题

```java
class Solution {
    public boolean isStraight(int[] nums) {
        Arrays.sort(nums);
        int zeroCount=0;
        for(int i=0;i<5;i++){
            if(nums[i]==0)zeroCount++;
        }
        
        for(int i=4;i>0;i--){
            if(nums[i]!=nums[i-1]+1){
                //有相同数字,false
                if(nums[i]==nums[i-1]&&nums[i]!=0)return false;
                //相差大于1，消耗一个0重置判断
                if(zeroCount==0)return false;
                zeroCount--;
                nums[0]=nums[i]-1;
                Arrays.sort(nums);
                i=5;
            }
        }
        
        return true;
    }
}
```

### 62 圆圈中最后剩下的数字

0,1,,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字。求出这个圆圈里剩下的最后一个数字。

例如，0、1、2、3、4这5个数字组成一个圆圈，从数字0开始每次删除第3个数字，则删除的前4个数字依次是2、0、4、1，因此最后剩下的数字是3。

**示例 1：**

```
输入: n = 5, m = 3
输出: 3
```

**示例 2：**

```
输入: n = 10, m = 17
输出: 2
```

使用List与模拟法

```java
class Solution {
    public int lastRemaining(int n, int m) {
        List<Integer> list=new ArrayList<>();
        for(int i=0;i<n;i++){
            list.add(i);
        }
        int c=(m-1)%n;
        
        while(list.size()!=1){
            list.remove(c);
            c=(c+m-1)%list.size();
        }
        
        return list.get(0);
    }
}
```

另一种解法是约瑟夫环，递推公式...数学解法

### 63.股票的最大利润

假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

**示例 1:**

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
```

**示例 2:**

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

 最开始想复杂了，按照常规简化暴力法

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices==null)return 0;
        //买卖一次，双指针
        int a=0;
        int b=prices.length-1;
        int res=0;
        while(a<=b){
            int minIndex=a;
            int maxIndex=b;
            for(int i=a;i<b;i++){
                if(prices[i]<=prices[minIndex])minIndex=i;
            }
            for(int i=b;i>=0;i--){
                if(prices[i]>=prices[maxIndex])maxIndex=i;
            }
            if(minIndex<=maxIndex){
                res=Math.max(res,prices[maxIndex]-prices[minIndex]);
                break;
            }else{
                for(int i=minIndex+1;i<=b;i++)res=Math.max(res,prices[i]-prices[minIndex]);
                for(int i=a;i<maxIndex;i++)res=Math.max(res,prices[maxIndex]-prices[i]);
                a=maxIndex+1;
                b=minIndex-1;
            }
        }
        
        return res;
    }
}
```

实际上只要动态维护一个min即可，因为后来的值如果大于min，那么这就是最佳值min，如果小于，则当前值以前的其实全部判断完了，更新min，以后的值以当前值为最佳min

```java
class Solution {
    public int maxProfit(int[] prices) {
        if(prices==null||prices.length<=1)return 0;
        //买卖一次，双指针
        int min=prices[0];
        int res=0;
        for(int i=1;i<prices.length;i++){
            res=Math.max(res,prices[i]-min);
            min=Math.min(min,prices[i]);
        }
        return res;
    }
}
```

### 64 求1+2+…+n

求 `1+2+...+n` ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

**示例 1：**

```
输入: n = 3
输出: 6
```

**示例 2：**

```
输入: n = 9
输出: 45
```

利用and操作的短路特性实现if或三元判断，没想到

```java
class Solution {
    public int sumNums(int n) {
        int res=n;
        boolean flag= (n!=0&&((res+=sumNums(n-1)) > 0));
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

### 67 把字符串转换成整数

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

### 68 二叉搜索树的最近公共祖先

给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root==null)return null;
        if((root.left==p&&root.right==q)||(root.left==q&&root.right==p))return root;
        if((root==p&&exist(p,q))||(root==q&&exist(q,p)))return root;
        
        if(exist(root.left,p)&&exist(root.left,q)){
            return lowestCommonAncestor(root.left,p,q);
        }
          
        if(exist(root.right,p)&&exist(root.right,q)){
            return lowestCommonAncestor(root.right,p,q);
        }
                     
        return root;
    }
    private boolean exist(TreeNode root,TreeNode sub){
        if(root==null)return false;
        if(root==sub)return true;
        return exist(root.left,sub)||exist(root.right,sub);
    }
}
```

然而，题目中给定的是二叉搜索树，利用二叉搜索树的性质可以方便求解，审题要注意

就算如此，这个算法速度仍然太低，因为使用了太多次重复搜索，更巧妙的办法是

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root==null)return null;
        if(root==p||root==q)return root;
        
        TreeNode left=lowestCommonAncestor(root.left,p,q);
        TreeNode right=lowestCommonAncestor(root.right,p,q);
        if(left!=null&&right!=null)return root;
        
        if(left!=null)return left;
        if(right!=null)return right;
        return null;

    }
}
```

关键点在于if(left!=null&&right!=null)return root;此处将节点是否在同一个子树的判断融入递归当中，如果不在同一个子树当中，那么必定有left与right都不为null



