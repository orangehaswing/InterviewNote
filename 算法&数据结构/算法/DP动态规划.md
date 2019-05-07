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











