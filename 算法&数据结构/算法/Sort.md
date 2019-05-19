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

### 快速选择

用于求解 **Kth Element** 问题，使用快速排序的 partition() 进行实现。

需要先打乱数组，否则最坏情况下时间复杂度为 O(N2)。

### 堆排序

用于求解 **TopK Elements** 问题，通过维护一个大小为 K 的堆，堆中的元素就是 TopK Elements。

堆排序也可以用于求解 Kth Element 问题，堆顶元素就是 Kth Element。

快速选择也可以求解 TopK Elements 问题，因为找到 Kth Element 之后，再遍历一次数组，所有小于等于 Kth Element 的元素都是 TopK Elements。

可以看到，快速选择和堆排序都可以求解 Kth Element 和 TopK Elements 问题。

**Kth Element**

[215. Kth Largest Element in an Array (Medium)](https://leetcode.com/problems/kth-largest-element-in-an-array/description/)

题目描述：找到第 k 大的元素。

**排序** ：时间复杂度 O(NlogN)，空间复杂度 O(1)

```
public int findKthLargest(int[] nums, int k) {
    Arrays.sort(nums);
    return nums[nums.length - k];
}
```

**堆排序** ：时间复杂度 O(NlogK)，空间复杂度 O(K)。

```
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> pq = new PriorityQueue<>(); // 小顶堆
    for (int val : nums) {
        pq.add(val);
        if (pq.size() > k)  // 维护堆的大小为 K
            pq.poll();
    }
    return pq.peek();
}
```

**快速选择** ：时间复杂度 O(N)，空间复杂度 O(1)

```
public int findKthLargest(int[] nums, int k) {
    k = nums.length - k;
    int l = 0, h = nums.length - 1;
    while (l < h) {
        int j = partition(nums, l, h);
        if (j == k) {
            break;
        } else if (j < k) {
            l = j + 1;
        } else {
            h = j - 1;
        }
    }
    return nums[k];
}

private int partition(int[] a, int l, int h) {
    int i = l, j = h + 1;
    while (true) {
        while (a[++i] < a[l] && i < h) ;
        while (a[--j] > a[l] && j > l) ;
        if (i >= j) {
            break;
        }
        swap(a, i, j);
    }
    swap(a, l, j);
    return j;
}

private void swap(int[] a, int i, int j) {
    int t = a[i];
    a[i] = a[j];
    a[j] = t;
}
```

### 桶排序

**出现频率最多的 k 个数**

[347. Top K Frequent Elements (Medium)](https://leetcode.com/problems/top-k-frequent-elements/description/)

```
Given [1,1,1,2,2,3] and k = 2, return [1,2].
```

设置若干个桶，每个桶存储出现频率相同的数，并且桶的下标代表桶中数出现的频率，即第 i 个桶中存储的数出现的频率为 i。

把数都放到桶之后，从后向前遍历桶，最先得到的 k 个数就是出现频率最多的的 k 个数。

```
public List<Integer> topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> frequencyForNum = new HashMap<>();
    for (int num : nums) {
        frequencyForNum.put(num, frequencyForNum.getOrDefault(num, 0) + 1);
    }
    List<Integer>[] buckets = new ArrayList[nums.length + 1];
    for (int key : frequencyForNum.keySet()) {
        int frequency = frequencyForNum.get(key);
        if (buckets[frequency] == null) {
            buckets[frequency] = new ArrayList<>();
        }
        buckets[frequency].add(key);
    }
    List<Integer> topK = new ArrayList<>();
    for (int i = buckets.length - 1; i >= 0 && topK.size() < k; i--) {
        if (buckets[i] != null) {
            topK.addAll(buckets[i]);
        }
    }
    return topK;
}
```

**按照字符出现次数对字符串排序**

[451. Sort Characters By Frequency (Medium)](https://leetcode.com/problems/sort-characters-by-frequency/description/)

```
Input:
"tree"

Output:
"eert"

Explanation:
'e' appears twice while 'r' and 't' both appear once.
So 'e' must appear before both 'r' and 't'. Therefore "eetr" is also a valid answer.
```

```
public String frequencySort(String s) {
    Map<Character, Integer> frequencyForNum = new HashMap<>();
    for (char c : s.toCharArray())
        frequencyForNum.put(c, frequencyForNum.getOrDefault(c, 0) + 1);

    List<Character>[] frequencyBucket = new ArrayList[s.length() + 1];
    for (char c : frequencyForNum.keySet()) {
        int f = frequencyForNum.get(c);
        if (frequencyBucket[f] == null) {
            frequencyBucket[f] = new ArrayList<>();
        }
        frequencyBucket[f].add(c);
    }
    StringBuilder str = new StringBuilder();
    for (int i = frequencyBucket.length - 1; i >= 0; i--) {
        if (frequencyBucket[i] == null) {
            continue;
        }
        for (char c : frequencyBucket[i]) {
            for (int j = 0; j < i; j++) {
                str.append(c);
            }
        }
    }
    return str.toString();
}
```

### 荷兰国旗问题

荷兰国旗包含三种颜色：红、白、蓝。

有三种颜色的球，算法的目标是将这三种球按颜色顺序正确地排列。

它其实是三向切分快速排序的一种变种，在三向切分快速排序中，每次切分都将数组分成三个区间：小于切分元素、等于切分元素、大于切分元素，而该算法是将数组分成三个区间：等于红色、等于白色、等于蓝色。

[![img](https://github.com/CyC2018/CS-Notes/raw/master/docs/notes/pics/3b49dd67-2c40-4b81-8ad2-7bbb1fe2fcbd.png)](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/pics/3b49dd67-2c40-4b81-8ad2-7bbb1fe2fcbd.png)

**按颜色进行排序**

[75. Sort Colors (Medium)](https://leetcode.com/problems/sort-colors/description/)

```
Input: [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]
```

题目描述：只有 0/1/2 三种颜色。

```
public void sortColors(int[] nums) {
    int zero = -1, one = 0, two = nums.length;
    while (one < two) {
        if (nums[one] == 0) {
            swap(nums, ++zero, one++);
        } else if (nums[one] == 2) {
            swap(nums, --two, one);
        } else {
            ++one;
        }
    }
}

private void swap(int[] nums, int i, int j) {
    int t = nums[i];
    nums[i] = nums[j];
    nums[j] = t;
}
```









