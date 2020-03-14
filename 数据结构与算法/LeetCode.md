#### [994. 腐烂的橘子](https://leetcode-cn.com/problems/rotting-oranges/)

在给定的网格中，每个单元格可以有以下三个值之一：

- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。

每分钟，任何与腐烂的橘子（在 4 个正方向上）相邻的新鲜橘子都会腐烂。

返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`。

**示例 1：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/16/oranges.png)**

```
输入：[[2,1,1],[1,1,0],[0,1,1]]
输出：4
```

**示例 2：**

```
输入：[[2,1,1],[0,1,1],[1,0,1]]
输出：-1
解释：左下角的橘子（第 2 行， 第 0 列）永远不会腐烂，因为腐烂只会发生在 4 个正向上。
```

**示例 3：**

```
输入：[[0,2]]
输出：0
解释：因为 0 分钟时已经没有新鲜橘子了，所以答案就是 0 。
```

非常考验细节，开始时，没有腐烂的橘子要返回-1，没有新鲜橘子要返回0

**腐烂同时开始，因此最短时间其实就是所有橘子同时开始腐蚀的时间，只要一次bfs即可，不需要回溯**

将所有腐烂的橘子加入栈中，进行bfs，最后如果还有新鲜橘子就返回-1，否则返回结果

```java
class Solution {

    public int orangesRotting(int[][] grid) {
        if(grid==null||grid.length<=0)return -1;
        //检测是否存在新鲜橘子
        boolean find=false;
        for(int i=0;i<grid.length;i++){
            for(int j=0;j<grid[0].length;j++){
                if(grid[i][j]==1){
                    find=true;
                }

            }
        }
        if(!find)return 0;
        
        //将所有腐烂的橘子加入栈中
        Stack<Pair> s=new Stack<>();
        for(int i=0;i<grid.length;i++){
            for(int j=0;j<grid[0].length;j++){
                if(grid[i][j]==2){//腐烂开始
                    s.add(new Pair(i,j));
                }
            }
        }
        //没有腐烂的橘子
        if(s.empty())return -1;
        //仍有新鲜橘子表明腐烂不玩所有的橘子
        int res=bfs(grid,s);
        for(int i=0;i<grid.length;i++){
            for(int j=0;j<grid[0].length;j++){
                if(grid[i][j]==1){
                    return -1;
                }
            }
        }
        
        return res;
    }
    //腐蚀所有橘子
    private int bfs(int[][] grid,Stack<Pair> stack){
        Stack<Pair> next=new Stack<>();//下一层要腐烂的数据
        for(Pair p:stack){
            
            int x=p.x;
            int y=p.y;
            if(0<=x-1&&grid[x-1][y]==1){
                next.add(new Pair(x-1,y));
                grid[x-1][y]=2;//腐烂
            }    
            if(x+1<grid.length&&grid[x+1][y]==1){
                next.add(new Pair(x+1,y));
                grid[x+1][y]=2;//腐烂
            }
            if(0<=y-1&&grid[x][y-1]==1){
                next.add(new Pair(x,y-1));
                grid[x][y-1]=2;//腐烂
            }
            if(y+1<grid[0].length&&grid[x][y+1]==1){
                next.add(new Pair(x,y+1));
                grid[x][y+1]=2;//腐烂
            }
        }
        int tmp=0;
        if(!next.empty()){
            tmp=1+bfs(grid,next);
        }
        return tmp;
    }
    
    class Pair{
        int x;
        int y;
        public Pair(int x,int y){
            this.x=x;
            this.y=y;
        }
    }
}
```



