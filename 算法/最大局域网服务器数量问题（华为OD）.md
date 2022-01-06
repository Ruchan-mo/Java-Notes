面试遇到的一些比较难的算法题

1. 局域网最大服务器数（华为OD机考）

   **题目**

   有一个 n * m 的机柜，数字1代表此处存放着服务器。左右相邻的服务器，或者上下相邻的服务器可以组成局域网，问最大的局域网内服务器数量。

   示例：

   ```java
   /**
    * 1 1 0 1 1
    * 0 1 1 0 0
    * 1 0 1 0 0
    * 1 1 1 0 1
    * 0 1 0 0 0
    */
   ```

   输出：10（对角线服务器不能组网）

   **解题**

   坐这道题的时候想过，可能是一道动态规划的题目，自己没有好好复习动态规划，实在是不会用。一开始的思路是，算出一行内相连的服务器数量，再算出上下两行相邻服务器数量，相加即可。但是这个思路有一个问题，比如第一行相连的服务器有 2+2=4 台，然而后面两台并没有和其他服务器连接在一起，所以不能组网。这种方法提交后通过率只有30%，显然是不能通过测试的。

   考完跟一位科班生学妹交流，她提到八皇后问题，并且说了一下解题思路，我马上觉得题目很相似，于是开始尝试用八皇后解法：

   - 先将数据存放到二维数组 `servers[n][m]`
   - 假设从第一个数出发，判断它的上下左右是否有1，有的话，就将坐标移动到1的位置，然后将第一个点置0，并且计数的变量count+1，直到判断上下左右都是0，无法前进，即返回。
   - 如果遇到分叉点，比如左右都有1，那么只需要选择一条路，在遇到0后返回该点，继续前进即可。这很容易让人想到可以用递归实现。
   - 最后一点，判断上下左右的时候，如果是在边边，会遇到突破数组索引范围，导致报错。可以在数据的上下左右放入值为0的数，类似在棋盘的周边围上一圈空格子。

   **代码**

   ```java
   // 存放服务器位置的二维数组
   private static int[][] servers;
   // 每次遍历用来存放相连的服务器数
   private static int count;
   
   /**
    * 首先读取控制台的输入
    *
    */
   public static void main(String[] args) throws IOException {
           BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
           String[] nm = reader.readLine().split(" ");
       	// 读取 n 行 m 列
           int n = Integer.parseInt(nm[0]); 
           int m = Integer.parseInt(nm[1]);
           servers = new int[n+2][m+2]; // 上下左右各多出一圈，正好多出2行2列0
           // 读取数据放入数组中
           for (int i = 1; i < n+1; i++) {
               String[] input = reader.readLine().split(" ");
               for (int j = 0; j < m; j++) {
                   servers[i][j + 1] = Integer.parseInt(input[j]);
               }
           }
       
           int maxCount = 0;
       	// 依次以每个数作为出发点，求出最大的服务器数量
           for (int i = 1; i < n + 1; i++) {
               for (int j = 1; j < m + 1; j++) {
                   // 只挑当前位置为1的点做起点
                   if (servers[i][j] != 1) {
                    continue;
                   }
                   searchServer(i, j);
                   // 取所有遍历中最大的 count
                   maxCount = Math.max(maxCount, count);
                   count = 0;
               }
           }
           System.out.println(maxCount);
       }
   
   	// 核心算法，先将当前点置0，然后从四个方向搜索1
   	static void searchServer(int x, int y) {
           servers[x][y] = 0;
           count++;
           if (servers[x-1][y] == 1) searchServer(x-1, y);
           if (servers[x+1][y] == 1) searchServer(x+1, y);
           if (servers[x][y-1] == 1) searchServer(x, y-1);
           if (servers[x][y+1] == 1) searchServer(x, y+1);
       }
   ```
   
   

