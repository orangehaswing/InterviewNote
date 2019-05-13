# Two Sum法

**LeetCode 1 Two Sum**

给定数组，返回两个元素和等于target

```
Given nums = [2, 7, 11, 15], target = 9,
Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

用HashMap/HashSet方式存放其中一个值；

```
public int[] twoSum(int[] numbers, int target) {
    int[] result = new int[2];
    Map<Integer, Integer> map = new HashMap<Integer, Integer>();
    for (int i = 0; i < numbers.length; i++) {
        if (map.containsKey(target - numbers[i])) {
            result[1] = i + 1;
            result[0] = map.get(target - numbers[i]);
            return result;
        }
        map.put(numbers[i], i + 1);
    }
    return result;
}
```

**LeetCode 167 Two Sum II - Input array is sorted**

给定排序数组，找到两个元素和等于target。使用two pointers方法。

```
Input: numbers = [2,7,11,15], target = 9
Output: [1,2]
Explanation: The sum of 2 and 7 is 9. Therefore index1 = 1, index2 = 2.
```

```
public int[] twoSum(int[] num, int target) {
    int[] indice = new int[2];
    if (num == null || num.length < 2) return indice;
    int left = 0, right = num.length - 1;
    while (left < right) {
        int v = num[left] + num[right];
        if (v == target) {
            indice[0] = left + 1;
            indice[1] = right + 1;
            break;
        } else if (v > target) {
            right --;
        } else {
            left ++;
        }
    }
    return indice;
}
```

**LintCode 2Sum II**

给定数组，找出有多少对数组满足sum大于 target，返回对数

```
numbers=[2, 7, 11, 15], target=24
return 1
```

Array 排序，然后使用Two Pointers方式， Binary Search。给定条件是大于target的所有对数，每次满足

num[left]+num[right] > target，那么其中所有的left右边部分都满足大于target。对数是right-left这么多，然后right--。

```
	public int twoSum2(int[] nums, int target) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        int count = 0;
        int left = 0;
        int right = nums.length - 1;
        Arrays.sort(nums);
        while (left < right) {
            if (nums[left] + nums[right] > target) {
                count += (right - left);
                right--;
            } else {
                left++;
            }
        }
        return count;
    }
```

**LeetCode 653 Two Sum IV - Input is a BST**

给定一颗二叉查找树，找到两个数等于target

```
input: 
    5
   / \
  3   6
 / \   \
2   4   7
Target = 9
Output: True
input: 
    5
   / \
  3   6
 / \   \
2   4   7
Target = 9
Output: True
```

使用HashSet的方式

```
  public boolean findTarget(TreeNode root, int k) {
        HashSet<Integer> set = new HashSet<>();
        return dfs(root, set, k);
    }
    
    public boolean dfs(TreeNode root, HashSet<Integer> set, int k){
        if(root == null)return false;
        if(set.contains(k - root.val))return true;
        set.add(root.val);
        return dfs(root.left, set, k) || dfs(root.right, set, k);
    }
```

使用中序遍历，然后二分查找法

```
 public boolean findTarget(TreeNode root, int k) {
        List<Integer> nums = new ArrayList<>();
        inorder(root, nums);
        for(int i = 0, j = nums.size()-1; i<j;){
            if(nums.get(i) + nums.get(j) == k)return true;
            if(nums.get(i) + nums.get(j) < k)i++;
            else j--;
        }
        return false;
    }
    
    public void inorder(TreeNode root, List<Integer> nums){
        if(root == null)return;
        inorder(root.left, nums);
        nums.add(root.val);
        inorder(root.right, nums);
    }
```

**LeetCode 523 Continuous Subarray Sum**

给定数组，判断连续子串相加能否整除target

```
Input: [23, 2, 4, 6, 7],  k=6
Output: True
Explanation: Because [2, 4] is a continuous subarray of size 2 and sums up to 6.
Input: [23, 2, 6, 4, 7],  k=6
Output: True
Explanation: Because [23, 2, 6, 4, 7] is an continuous subarray of size 5 and sums up to 42.
```

```
Some damn it! test cases:
[0], 0 -> false;
[5, 2, 4], 5 -> false;
[0, 0], 100 -> true;
[1,5], -6 -> true;
etc...

public class Solution {
    public boolean checkSubarraySum(int[] nums, int k) {
        // Since the size of subarray is at least 2.
        if (nums.length <= 1) return false;
        // Two continuous "0" will form a subarray which has sum = 0. 0 * k == 0 will always be true.
        for (int i = 0; i < nums.length - 1; i++) {
            if (nums[i] == 0 && nums[i + 1] == 0) return true;
        }

        // At this point, k can't be "0" any longer.
        if (k == 0) return false;
        // Let's only check positive k. Because if there is a n makes n * k = sum, it is always true -n * -k = sum.
        if (k < 0) k = -k;

        Map<Integer, Integer> sumToIndex = new HashMap<>();
        int sum = 0;
        sumToIndex.put(0, -1);

        for (int i = 0; i < nums.length; i++) {
            sum += nums[i];
            // Validate from the biggest possible n * k to k
            for (int j = (sum / k) * k; j >= k; j -= k) {
                if (sumToIndex.containsKey(sum - j) && (i - sumToIndex.get(sum - j) > 1)) return true;
            }
            if (!sumToIndex.containsKey(sum)) sumToIndex.put(sum, i);
        }

        return false;
    }
}
```

**LeetCode 1010 Pairs of Songs With Total Durations Divisible by 60**

一个数组，找出几对，满足相加和可以整除60

```
Input: [30,20,150,100,40]
Output: 3
Explanation: Three pairs have a total duration divisible by 60:
(time[0] = 30, time[2] = 150): total duration 180
(time[1] = 20, time[3] = 100): total duration 120
(time[1] = 20, time[4] = 40): total duration 60
Input: [60,60,60]
Output: 3
Explanation: All three pairs have a total duration of 120, which is divisible by 60.
```

使用 two sum方法，hash查找

这里有个比较好用的技巧,不必判断hash的key是否存在，如果不存在，直接返回value = 0；

```
index += hashMap.getOrDefault(temp,0);
```

```
	public int numPairsDivisibleBy60(int[] time) {
        int index = 0;

        HashMap<Integer,Integer> hashMap = new HashMap<>();
        for (int i = 0; i < time.length; i++) {
             int temp = (60-time[i]%60)%60;
            index += hashMap.getOrDefault(temp,0);

            temp = time[i]%60;
            hashMap.put(temp,1+hashMap.getOrDefault(temp,0));
        }

        return index;
    }
```

**LeetCode 628 Maximum Product of Three Numbers**

给定数组(有正有负)，输出其中三个元素相乘的最大乘积。

```
Input: [1,2,3]
Output: 6
Input: [-1,-2,-3]
Output: -6
```

三个数相乘，分为两类：max1 * max2 * max3； max1 * min1 * min2

```
    public int maximumProduct(int[] nums) {
        int max1 = Integer.MIN_VALUE;
        int max2 = Integer.MIN_VALUE;
        int max3 = Integer.MIN_VALUE;
        int min1 = Integer.MAX_VALUE;
        int min2 = Integer.MAX_VALUE;

        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > max1) {
                max3 = max2;
                max2 = max1;
                max1 = nums[i];
            } else if (nums[i] > max2) {
                max3 = max2;
                max2 = nums[i];
            } else if (nums[i] > max3) {
                max3 = nums[i];
            }

            if (nums[i] < min1) {
                min2 = min1;
                min1 = nums[i];
            } else if (nums[i] < min2) {
                min2 = nums[i];
            }

        }
        return Math.max(max1 * max2 * max3, max1 * min1 * min2);
    }
```

**LeetCode 16 3sum Closest**

给定数组，三个数相加sum，与target绝对差最小值

```
Given array nums = [-1, 2, 1, -4], and target = 1.
The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
```

2Sum(two pointers方法)+一层遍历

```
public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums); // nLog(n)
        long result = nums[0] + nums[1] + nums[2];
        
        for (int i = 0; i < nums.length - 2; i++) {
            int start = i + 1;
            int end = nums.length - 1;
            while (start < end) {
                long sum = nums[start] + nums[end] + nums[i];
                if (sum > target) {
                    end--;
                } else {
                    start++;
                }
                result = Math.abs(sum - target) < Math.abs(result - target) ? sum : result;
            }
        }
        return (int)result;
    }
```

**LeetCode 259 3Sum Smaller**

给定数组，满足三个数之和sum，小于target条件的所有组合数量

```
given nums = [-2, 0, 1, 3], and target = 2.
Return 2. Because there are two triplets which sums are less than 2:
[-2, 0, 1]
[-2, 0, 3]
```

2sum法(two pointers)+一层遍历

```
public int threeSumSmaller(int[] nums, int target) {
        Arrays.sort(nums);

        int cnt = 0;
        for (int i = 0; i < nums.length-2; i++) {
            int start = i+1;
            int end = nums.length-1;
            while (start < end){
                if (nums[i] + nums[start] + nums[end] < target){
                    cnt += end-start;
                    start++;
                }else {
                    end--;
                }
            }
        }

        return cnt;
    }
```

15 3sum

给定数组，求三个数之和等于0的所有组合

```
Given array nums = [-1, 0, 1, 2, -1, -4],
A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

2sum(two pointers) + 一次遍历。Hashset去重

```
public List<List<Integer>> threeSum(int[] nums) {
        HashSet<List<Integer>> res = new HashSet<>();
        Arrays.sort(nums);

        for (int i = 0; i < nums.length-2; i++) {
            int start = i+1;
            int end = nums.length -1;

            while (start < end){
                if (nums[i] + nums[start] + nums[end] == 0){
                    ArrayList<Integer> temp = new ArrayList<>();
                    res.add(Arrays.asList(nums[i],nums[start],nums[end]));
                    start++;
                    while (start < end && nums[start-1] == nums[start]){
                        start++;
                    }
                }else if (nums[i] + nums[start] + nums[end] > 0){
                    end--;
                }else {
                    start++;
                }
            }
        }

        List<List<Integer>> r = new ArrayList<>();

        for (List<Integer> a:res) {
            r.add(a);
        }

        return r;
    }
```













































