# String

LeetCode 3 Longest Substring Without Repeating Characters

```
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

用hashset和快慢指针，hashset放不重复子串。对j位置字符做包含比较，如果不包含，加入hashset。如果包含，把从i位置开始的字符删除，直到不包含j位置的字符。

```
public int lengthOfLongestSubstring(String s) {
    int i = 0, j = 0, max = 0;
    Set<Character> set = new HashSet<>();
    
    while (j < s.length()) {
        if (!set.contains(s.charAt(j))) {
            set.add(s.charAt(j++));
            max = Math.max(max, set.size());
        } else {
            set.remove(s.charAt(i++));
        }
    }
    
    return max;
}
```

LeetCode 5 Longest Palindromic Substring

最长回文子序列

```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
Input: "cbbd"
Output: "bb"
```

思路都是使用指针，从两端相等点向中心方向靠近，或者从中心位置向两端远离，判断每移动一格，记录相等的String和length。

```
private int lo, maxLen;
public String longestPalindrome(String s) {
	int len = s.length();
	if (len < 2)
		return s;
	
    for (int i = 0; i < len-1; i++) {
     	extendPalindrome(s, i, i);  //assume odd length, try to extend Palindrome as possible
     	extendPalindrome(s, i, i+1); //assume even length.
    }
    return s.substring(lo, lo + maxLen);
}

private void extendPalindrome(String s, int j, int k) {
	while (j >= 0 && k < s.length() && s.charAt(j) == s.charAt(k)) {
		j--;
		k++;
	}
	if (maxLen < k - j - 1) {
		lo = j + 1;
		maxLen = k - j - 1;
	}
}
```

```
	public String longestPalindrome(String s) {
       int len = 0;
        String res = "";
        for (int i = 0; i < s.length(); i++) {
            for (int j = s.length()-1; j >=i ; j--) {
                if (s.charAt(i) == s.charAt(j)){
                    StringBuilder subs = new StringBuilder(s.substring(i,j+1));
                    if (subs.toString().equals(subs.reverse().toString())){
                        if (len < subs.length()){
                            res =subs.toString();
                            len = subs.length();
                        }
                         break;
                    }
                }
            }
        }

        return res; 
    }
```

**LeetCode 415 Add Strings**

两个代表正数的字符串，求sum

```
public String addStrings(String num1, String num2) {
      StringBuilder sb = new StringBuilder();
        int carry = 0;
        for (int i = num1.length() - 1, j = num2.length() - 1; i >= 0 || j >= 0; i--, j--) {
            int x = i < 0 ? 0 : num1.charAt(i)-'0';
            int y = j < 0 ? 0 : num2.charAt(j)-'0';

            sb.append((x + y + carry) % 10);
            carry = (x + y+ carry) / 10;
        }
        
        if (carry > 0){
            sb.append(carry);
        }
        
        return sb.reverse().toString();
    }
```

**LeetCode 165 Compare Version Numbers**

两个版本version1，version2，如果version1>version2,返回1；version1>version2,返回-1；其他返回0

```
Input: version1 = "0.1", version2 = "1.1"
Output: -1
Input: version1 = "1.01", version2 = "1.001"
Output: 0
Explanation: Ignoring leading zeroes, both “01” and “001" represent the same number “1”
Input: version1 = "1.0", version2 = "1.0.0"
Output: 0
Explanation: The first version number does not have a third level revision number, which means its third level revision number is default to "0"
```

按照`\\.` 切分，每一位比较，长度短的，默认补0；

```
public static int compareVersion(String version1, String version2) {
        String[] s1 = version1.split("\\.");
        String[] s2 = version2.split("\\.");
        int len = Math.max(s1.length,s2.length);

        for (int i = 0; i < len; i++) {
            Integer v1 = i<s1.length?Integer.parseInt(s1[i]):0;
            Integer v2 = i<s2.length?Integer.parseInt(s2[i]):0;

            int compare = v1.compareTo(v2);
            if (compare != 0){
                return compare;
            }
        }

        return 0;
    }
```

## 文章和段落

给一段英文，和禁止词数组。求：除了禁止词外，出现频率最高的词，不区分大小写。

```
Input: 
paragraph = "Bob hit a ball, the hit BALL flew far after it was hit."
banned = ["hit"]
Output: "ball"
Explanation: 
"hit" occurs 3 times, but it is a banned word.
"ball" occurs twice (and no other word does), so it is the most frequent non-banned word in the paragraph. 
Note that words in the paragraph are not case sensitive,
that punctuation is ignored (even if adjacent to words, such as "ball,"), 
and that "hit" isn't the answer even though it occurs more because it is banned.
```

首先需要把段落按照词切分，用正则表达式：

```
String[] word = paragraph.toLowerCase().split("[ !?',;.]+");
```

hashmap记录 word - count，去掉禁止词。遍历找到最大频率的词

```
public String mostCommonWord(String paragraph, String[] banned) {
        String[] words = paragraph.toLowerCase().split("[ !?',;.]+");
        HashMap<String, Integer> map = new HashMap<>();
        for(String word : words) map.put(word, map.getOrDefault(word, 0) + 1);
        for(String word : banned) if(map.containsKey(word)) map.remove(word);
        String res = null;
        for(String word : map.keySet())
            if(res == null || map.get(word) > map.get(res))
                res = word;
        return res;
    }
```

**LeetCode 151 Reverse Words in a String**

输入字符串，按照每个单词翻转，多个空格要去掉

```
Input: "the sky is blue"
Output: "blue is sky the"
Input: "  hello world!  "
Output: "world! hello"
Explanation: Your reversed string should not contain leading or trailing spaces.
Input: "a good   example"
Output: "example good a"
Explanation: You need to reduce multiple spaces between two words to a single space in the reversed string.
```

使用String.trim()方法去掉首尾的空格，split按照空格切分，"\s+"表示一个或多个空格

```
	public String reverseWords(String s) {
        String[] res = s.trim().split("\\s+");
        StringBuilder sb = new StringBuilder();

        for (int i = res.length-1; i >= 0; i--) {
            sb.append(res[i]);
            sb.append(" ");
        }
        sb.deleteCharAt(sb.length()-1);

        return sb.toString();
    }
```

LeetCode 58 Length of Last Word

给定一个由字母和空格组成的字符串，找出最后一个单词的长度。如果最后一个单词不存在，返回0；

```
Example:
Input: "Hello World"
Output: 5
```

用StringBuilder把字符串翻转，然后从第一个开始遍历找到第一个出现空格的位置可能出现的情况如下

"A B";"A ";" A"; "   ";需要先把头尾的空格去掉

```
public int lengthOfLastWord(String s) {
        StringBuilder sb  = new StringBuilder(s);
        sb.reverse();

       String res = sb.toString().trim();
        
        for (int i = 0; i < res.length(); i++) {
            if (res.charAt(i) == ' '){
                return i;
            }
        }

        return res.length();
    }
```

LeetCode 67 Add Binary

给定两个二进制字符串，返回二进制和

```
Example 1:
Input: a = "11", b = "1"
Output: "100"

Example 2:
Input: a = "1010", b = "1011"
Output: "10101"
```

从字符串的最后一个开始加，carry表示进位，StringBuilder保存和

```
public String addBinary(String a, String b) {
        StringBuilder sb = new StringBuilder();

        int i = a.length()-1;
        int j = b.length()-1;
        int carry = 0;
        while (i >= 0 || j >= 0) {
            int sum = carry;
            if (i >= 0){
                sum += a.charAt(i--)-'0';
            }
            if (j >= 0){
                sum += b.charAt(j--)-'0';
            }

            sb.append(sum%2);
            carry = sum/2;
        }

        if (carry > 0){
            sb.append(carry);
        }

        return sb.reverse().toString();
    }
```





















