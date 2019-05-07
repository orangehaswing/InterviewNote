# Hash桶算法

LeetCode 448  Find All Numbers Disappeared in an Array

给定数组，有的元素出现一次，有的出现两次，找出没有出现的元素(所有元素都在1-n)中

```
Input:
[4,3,2,7,8,2,3,1]
Output:
[5,6]
```

不使用额外空间，时间复杂度O(n)，采用hash桶的方式，在原有数组上，增加base N

```
public List<Integer> findDisappearedNumbers(int[] nums) {
        List<Integer> res = new ArrayList<>();
        int n = nums.length;
        for (int i = 0; i < nums.length; i ++) {
          nums[(nums[i]-1) % n] += n;
        }
        for (int i = 0; i < nums.length; i ++){
          if (nums[i] <= n) res.add(i+1);
        } 
        return res;
 }
```

