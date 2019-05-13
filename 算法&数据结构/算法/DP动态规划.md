# DP动态规划

LeetCode 343 Integer Break

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

LeetCode 62 Unique Paths

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

LeetCode 63 Unique Paths II

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











