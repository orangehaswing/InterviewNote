# Sort

```
重载排序方法，使用lambd表达式
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
```

LeetCode 56 Merge Intervals

给定线段，合并所有重合区域，并输出新的线段区间

```
Input: [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].
Input: [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considered overlapping.
```

先对线段的开始位置排序，然后对每一段线段的头和尾与前一个保存值比较，最后转换成数组

```
public int[][] merge(int[][] intervals) {
		//排序
        Arrays.sort(intervals,(a,b)->a[0]- b[0]);
        List<int[]> ret = new ArrayList<>();
        int[] prev = null;
        for(int[] inter:intervals){
            if (prev == null || inter[0] > prev[1]){
                ret.add(inter);
                prev = inter;
            }else if (inter[1] > prev[1]){
                prev[1] = inter[1];
            }
        }

		// ArrayList转换数组
        return ret.toArray(new int[ret.size()][2]);
    }
```

LeetCode 75 Sort Colors

国企颜色问题

```
Input: [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]
```

使用partition交换方法

```
	public void sortColors(int[] nums) {
        int j = 0;
        int k = nums.length-1;
        for (int i = 0; i <= k; i++) {
            if (nums[i] == 0){
                swap(nums,i,j);
                j++;
            }else if (nums[i] == 2){
                swap(nums,i,k);
                k--;
                i--;
            }
        }
    }
    
    private void swap(int[] nums, int i, int j){
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
```

LeetCode 349 Intersection of Two Arrays

求两个数组的交集

```
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2]
Input: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
Output: [9,4]
```

方法一：排序+hashset去重

```
 	public int[] intersection(int[] nums1, int[] nums2) {
        Arrays.sort(nums1);
        Arrays.sort(nums2);

        HashSet<Integer> hashSet = new HashSet<>();
        for (int i = 0, j = 0; i <nums1.length && j<nums2.length ; ) {
            if (nums1[i] > nums2[j]){
                j++;
            }else if (nums1[i] < nums2[j]){
                i++;
            }else {
                hashSet.add(nums1[i]);
                i++;
                j++;
            }
        }

        int[] r = new int[hashSet.size()] ;
        int i = 0;
        for (Integer k:hashSet) {
            r[i++] = k;
        }

        return r;
    }
```

方法二，使用两个hashset，遍历比较

```
	public int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> set = new HashSet<>();
        Set<Integer> intersect = new HashSet<>();
        for (int i = 0; i < nums1.length; i++) {
            set.add(nums1[i]);
        }
        for (int i = 0; i < nums2.length; i++) {
            if (set.contains(nums2[i])) {
                intersect.add(nums2[i]);
            }
        }
        int[] result = new int[intersect.size()];
        int i = 0;
        for (Integer num : intersect) {
            result[i++] = num;
        }
        return result;
    }
```

方法三：取出其中一个数组的所有值，在另一个数组中二分查找

```
public int[] intersection(int[] nums1, int[] nums2) {
        Set<Integer> set = new HashSet<>();
        Arrays.sort(nums2);
        for (Integer num : nums1) {
            if (binarySearch(nums2, num)) {
                set.add(num);
            }
        }
        int i = 0;
        int[] result = new int[set.size()];
        for (Integer num : set) {
            result[i++] = num;
        }
        return result;
    }
    
    public boolean binarySearch(int[] nums, int target) {
        int low = 0;
        int high = nums.length - 1;
        while (low <= high) {
            int mid = low + (high - low) / 2;
            if (nums[mid] == target) {
                return true;
            }
            if (nums[mid] > target) {
                high = mid - 1;
            } else {
                low = mid + 1;
            }
        }
        return false;
    }
```













