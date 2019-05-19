# Binary Search

LeetCode 744 Find Smallest Letter Greater Than Target

一组排序的characters 数组，和target，找到比target大的最小元素。target大于等于所有值，返回第一个元素

```
Input:
letters = ["c", "f", "j"]
target = "a"
Output: "c"
Input:
letters = ["c", "f", "j"]
target = "g"
Output: "j"
Input:
letters = ["c", "f", "j"]
target = "j"
Output: "c"
```

直接遍历

```
public char nextGreatestLetter(char[] letters, char target) {
        for (char c : letters) {
            if (target < c){
                return c;
            }
        }

        return letters[0];

    }
```

二分查找

```
public char nextGreatestLetter(char[] letters, char target) {
        int start = 0;
        int end = letters.length-1;

        if (target >= letters[end]){
            return letters[0];
        }

        while (start < end){
            int mid = (end - start)/2;

            if (target >= letters[mid]){
                start = mid + 1;
            }else {
                end = mid;
            }
        }

        return letters[end];

    }
```









