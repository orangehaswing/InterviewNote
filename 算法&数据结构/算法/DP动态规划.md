# DP动态规划

## Number

**LeetCode 343 Integer Break**

给定一个整数，可以由多个数相加，求多个数乘机的最大值

```
Input: 2
Output: 1
Explanation: 2 = 1 + 1, 1 × 1 = 1.
Input: 10
Output: 36
Explanation: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36.
```

DP

```
class Solution {
    public int integerBreak(int n) {
        //dp[i] means output when input = i, e.g. dp[4] = 4 (2*2),dp[8] = 18 (2*2*3)...
        int[] dp = new int[n + 1];
        dp[1] = 1;
       // fill the entire dp array
        for (int i = 2; i <= n; i++) {
       //let's say i = 8, we are trying to fill dp[8]:if 8 can only be broken into 2 parts, the answer could be among 1 * 7, 2 * 6, 3 * 5, 4 * 4... but these numbers can be further broken. so we have to compare 1 with dp[1], 7 with dp[7], 2 with dp[2], 6 with dp[6]...etc
            for (int j = 1; j <= i / 2; j++) {
               // use Math.max(dp[i],....)  so dp[i] maintain the greatest value
                dp[i] = Math.max(dp[i],Math.max(j, dp[j]) * Math.max(i - j, dp[i - j]));
            }
        }
        return dp[n];
    }
}
```

LeetCode 55 Jump Game

给定一个数组，初始在index=0位置。每个元素代表可以最大跳数。判断能否跳到最后一个位置。

```
Example 1:
Input: [2,3,1,1,4]
Output: true
Explanation: Jump 1 step from index 0 to 1, then 3 steps to the last index.

Example 2:
Input: [3,2,1,0,4]
Output: false
Explanation: You will always arrive at index 3 no matter what. Its maximum
             jump length is 0, which makes it impossible to reach the last index.
```

记录每个点往后跳的值，比较得出最大值，当位置i>max，就不能跳到，否则可以继续从i点跳。

```
public boolean canJump(int[] nums) {
        if (nums == null || nums.length == 0){
            return false;
        }

        int max = 0;
        for (int i = 0; i < nums.length; i++) {
            if (i > max){
                return false;
            }
            max = Math.max(nums[i] + i, max);
        }

        return true;
    }
```

## Matrix

**LeetCode 62 Unique Paths**

在M*N的方格中，机器人在左上角，终点在右下角。机器人只能向右或者向下。总共有多少条路

```
Input: m = 3, n = 2
Output: 3
Explanation:
From the top-left corner, there are a total of 3 ways to reach the bottom-right corner:
1. Right -> Right -> Down
2. Right -> Down -> Right
3. Down -> Right -> Right
```

转移方程`dp[i][j] = dp[i-1][j] + dp[i][j-1]`

```
public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (i == 0 || j== 0){
                    dp[i][j] = 1;
                }else {
                    dp[i][j] = dp[i-1][j] + dp[i][j-1];
                }
            }
        }

        return dp[m-1][n-1];
    }
```

**LeetCode 63 Unique Paths II**

在62的基础上，增加障碍物。判断有多少路

```
Input:
[
  [0,0,0],
  [0,1,0],
  [0,0,0]
]
Output: 2
Explanation:
There is one obstacle in the middle of the 3x3 grid above.
There are two ways to reach the bottom-right corner:
1. Right -> Right -> Down -> Down
2. Down -> Down -> Right -> Right
```

参考62，当在第一行或者第一列时，如果存在障碍物，该行或列之后的值都为0。所以只要增加上第一行或第一列判断就可

```
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int m = obstacleGrid.length;
        int n = obstacleGrid[0].length;
        int[][] dp = new int[m][n];

        for (int i = 0; i < m; i++) {
            if (obstacleGrid[i][0] == 1){
                dp[i][0] = 0;
                break;
            }else {
                dp[i][0] = 1;
            }
        }

        for (int j = 0; j < n; j++) {
            if (obstacleGrid[0][j] == 1){
                dp[0][j] = 0;
                break;
            }else {
                dp[0][j] = 1;
            }
        }

        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (obstacleGrid[i][j] == 1){
                    dp[i][j] = 0;
                }else {
                    dp[i][j] = dp[i-1][j] + dp[i][j-1];
                }
            }

        }
        
        return dp[m-1][n-1];
    }
```

## Tree

**LeetCode 96 Unique Binary Search Trees**

输入n，输出1-n可以组成二叉查找树的个数

```
Input: 3
Output: 5
Explanation:
Given n = 3, there are a total of 5 unique BST's:
   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

`G(n)`: 长度为 n的BST数量.

`F(i, n), 1 <= i <= n`: 个数为n，根节点为i，左子树[1,i-1],右子树[i+1，n]。然后对左右子树递归求解，最初始状态是

G[0] = G[1] = 1

DP函数：

```
G(n) = F(1, n) + F(2, n) + ... + F(n, n).     
G(0)=1, G(1)=1. 
```

推导：

```
F(i, n) = G(i-1) * G(n-i)
G(n) = G(0) * G(n-1) + G(1) * G(n-2) + … + G(n-1) * G(0)
```

```
public int numTrees(int n) {
    int [] G = new int[n+1];
    G[0] = G[1] = 1;
    
    for(int i=2; i<=n; ++i) {
    	for(int j=1; j<=i; ++j) {
    		G[i] += G[j-1] * G[i-j];
    	}
    }

    return G[n];
}
```

**LeetCode 95 Unique Binary Search Trees II**

输入n，输出1-n可以组成二叉查找树的所有情况

```
Input: 3
Output:
[
  [1,null,3,2],
  [3,2,null,1],
  [3,1,null,null,2],
  [2,1,3],
  [1,null,2,null,3]
]
Explanation:
The above output corresponds to the 5 unique BST's shown below:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

思路按照LeetCode 96。按照题目要求，需要记录每一种二叉查找树，所以采用分治法

```
public List<TreeNode> generateTrees(int n) {
        List<TreeNode> res = new ArrayList<>();
        if (n == 0){
            return res;
        }
        
        return generateSubTrees(1,n);
    }

    private List<TreeNode> generateSubTrees(int start, int end) {
        List<TreeNode> res = new ArrayList<>();

        if (start > end){
            res.add(null);
            return res;
        }

        for (int i = start; i <= end ; i++) {
            List<TreeNode> leftSubTrees = generateSubTrees(start,i-1);
            List<TreeNode> rightSubTrees = generateSubTrees(i+1,end);

            for (TreeNode left : leftSubTrees) {
                for (TreeNode right : rightSubTrees){
                    TreeNode root = new TreeNode(i);
                    root.left = left;
                    root.right = right;
                    res.add(root);
                }
            }
        }

        return res;
    }
```

## String

LeetCode 91 Decode Ways

从A-Z代表1-26，现在给定数字字符串，输出由A-Z组成的方式

```
Example 1:
Input: "12"
Output: 2
Explanation: It could be decoded as "AB" (1 2) or "L" (12).

Example 2:
Input: "226"
Output: 3
Explanation: It could be decoded as "BZ" (2 26), "VF" (22 6), or "BBF" (2 2 6).
```

可以用动态规划，从字符串的末尾往前（从前往后遇到 "100","01","10"等数字0出现问题没解决，qaq）

```
public static int numDecodings_1(String s) {
        //DP
        int n = s.length();
        if (n == 0) {
            return 0;
        }

        int[] dp = new int[n + 1];
        dp[n] = 1;
        dp[n-1] = s.charAt(n-1) == '0'?0:1;
        for (int i = n-2; i >= 0; i--) {
            if (s.charAt(i) == '0'){
                continue;
            }else {
                dp[i] = (Integer.parseInt(s.substring(i,i+2)) <= 26)?dp[i+1]+dp[i+2]:dp[i+1];
            }
        }

        return dp[0];
    }
```















