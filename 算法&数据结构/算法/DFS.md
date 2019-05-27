# DFS

## 二叉树路径和

**LeetCode 112 Path Sum**

给定一棵二叉树，判断是否存在从根节点到叶子节点和等于sum的路径，返回true或者false

```
Given the below binary tree and sum = 22,

      5
     / \
    4   8
   /   / \
  11  13  4
 /  \      \
7    2      1
return true, as there exist a root-to-leaf path 5->4->11->2 which sum is 22.
```

递归调用，判断分两种情况，root == null表示找不到，root.left 和root.right都为null表示root为叶子节点，就需要判断val是否等于sum。其他情况往树的下一层找。

```
public boolean hasPathSum_1(TreeNode root, int sum) {
        if (root == null){
            return false;
        }
        return dfs(root,sum);
    }

    public boolean dfs(TreeNode root, int sum){
        if (root == null){
            return false;
        }

        if (root.left == null && root.right == null && root.val == sum){
            return true;
        }

        return  dfs(root.left,sum-root.val) || dfs(root.right,sum-root.val);
    }
```

LeetCode 113 Path Sum II

给定一棵二叉树，计算所有从根节点到叶子节点和等于sum的路径，并保存。

```
Given the below binary tree and sum = 22,

      5
     / \
    4   8
   /   / \
  11  13  4
 /  \    / \
7    2  5   1
Return:
[
   [5,4,11,2],
   [5,8,4,5]
]
```

递归调用,res保存所有的路径，temp保存从根节点到其中一条叶子节点的路径。所以，在添加val后，如果递归完返回，需要把该val删除。如果满足找到的叶子节点（root.left root.right为null），就判断val与sum是否相等，相等添加进res，不相等递归调用返回(回溯思想)。

```
public List<List<Integer>> pathSum(TreeNode root, int sum) {
        List<List<Integer>>  res = new ArrayList<>();
        ArrayList<Integer> temp = new ArrayList<>();

        if (root == null){
            return res;
        }

        dfs(root,sum,res,temp);
        return res;
    }

    public void dfs(TreeNode root,int sum,List<List<Integer>> res,ArrayList<Integer> temp){
        if (root == null){
            return;
        }
        temp.add(root.val);

        if (root.left == null && root.right == null){
            if (root.val == sum){
                res.add(new ArrayList<Integer>(temp));
            }
            return;
        }

        if (root.left != null){
            dfs(root.left,sum-root.val,res,temp);
            temp.remove(temp.size()-1);
        }
        if (root.right != null){
            dfs(root.right,sum-root.val,res,temp);
            temp.remove(temp.size()-1);
        }
    }
```

LeetCode 437 Path Sum III

给定一颗二叉树，计算所有从一个节点到另一个节点和等于sum的路径，并保存。节点不一定是根节点到叶子节点，但是保证从父节点到子节点。

```
root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8
      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1
Return 3. The paths that sum to 8 are:
1.  5 -> 3
2.  5 -> 2 -> 1
3. -3 -> 11
```

从根节点开始递归寻找（pathSum）。每一次递归的根节点开始寻找值等于sum（dfs）

```
 	int res = 0;
    public int pathSum(TreeNode root, int sum) {
        if (root == null){
            return 0;
        }

        dfs(root,sum);
        pathSum(root.left,sum);
        pathSum(root.right,sum);
        return res;
    }

    public void dfs(TreeNode root,int sum){
        if (root == null){
            return;
        }

        if (sum - root.val == 0) res+=1;
        dfs(root.left, sum-root.val);
        dfs(root.right, sum-root.val);
    }
```

LeetCode 257 Binary Tree Paths

给定二叉树，返回所有从根节点到叶子节点的路径

```
Input:
   1
 /   \
2     3
 \
  5
Output: ["1->2->5", "1->3"]
Explanation: All root-to-leaf paths are: 1->2->5, 1->3
```

因为没有限制条件，只需要直接递归到下一个节点并保存，所以不需要回溯，参考寻找路径等于sum II

```
public List<String> binaryTreePaths(TreeNode root) {
        List<String> res = new ArrayList<>();
        if (root == null)
            return res;

        dfs(root,res,"");
        return res;
    }

    public void dfs(TreeNode root, List<String> res, String s){
        if (root == null){
            return;
        }

        if (root.left == null && root.right == null){
            res.add(s + root.val);
        }

        if (root.left != null){
            dfs(root.left,res,s+root.val + "->");
        }
        if (root.right != null){
            dfs(root.right,res,s+root.val + "->");
        }
    }
```

LeetCode 494 Target Sum

本题使用DFS的方法寻找sum值，结构是二叉树路径和。

题意：给定一串不包含负数的list。只能用+ -两种符号，找出所有运算sum等于target的路径数。

```
Input: nums is [1, 1, 1, 1, 1], S is 3. 
Output: 5
Explanation: 
-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3
There are 5 ways to assign symbols to make the sum of nums be target 3.
```

```
	int result = 0;
    public int findTargetSumWays(int[] nums, int S) {
        if (nums == null || nums.length == 0) return result;
        helper(nums, S, 0, 0);
        return result;
    }
    
    public void helper(int[] nums, int target, int pos, long eval){
        if (pos == nums.length) {
            if (target == eval) result++;
            return;
        }
        helper(nums, target, pos + 1, eval + nums[pos]);
        helper(nums, target, pos + 1, eval - nums[pos]);
    }
```

LeetCode 404 Sum of Left Leaves

给定二叉树，计算所有左叶子的和

```
 	3
   / \
  9  20
    /  \
   15   7
There are two left leaves in the binary tree, with values 9 and 15 respectively. Return 24.
```

DFS+ArrayList保存左叶子节点值

```
	public int sumOfLeftLeaves(TreeNode root) {
         ArrayList<Integer> arr = new ArrayList<>();
        if(root == null){
            return 0;
        }
        dfs(root,arr);

        int sum = 0;
        for(Integer i:arr){
            sum += i;
        }

        return sum;
    }
    
    private void dfs(TreeNode root, ArrayList arr){
        if (root == null){
            return;
        }

        if (root.left != null){
            if (root.left.left == null && root.left.right == null){
                arr.add(root.left.val);
            }
        }

        dfs(root.left,arr);
        dfs(root.right,arr);
    }
```

LeetCode 938 Range Sum of BST

给定一颗二叉查找树，左子树位置和右子树位置。求前序遍历后，左子树到右子树的和

```
Input: root = [10,5,15,3,7,null,18], L = 7, R = 15
Output: 32
Input: root = [10,5,15,3,7,13,18,1,null,6], L = 6, R = 10
Output: 23
```

DFS先序遍历

```
 public int rangeSumBST(TreeNode root, int L, int R) {
         if (root == null){
            return 0;
        }

        ArrayList<Integer> arr = new ArrayList<>();
        inorder(root,arr);

        Collections.sort(arr);

        int sum = 0;
        for(Integer i:arr){
            if (i>= L && i<= R){
                sum += i;
            }
        }

        return sum;
    }

    private void inorder(TreeNode root, ArrayList<Integer> arr){
        if (root == null){
            return;
        }

        inorder(root.left,arr);
        arr.add(root.val);
        inorder(root.right,arr);
    }
```

LeetCode 337 House Robber III

抢劫位置只能隔一层抢劫一层

```
Input: [3,2,3,null,3,null,1]

     3
    / \
   2   3
    \   \ 
     3   1

Output: 7 
Explanation: Maximum amount of money the thief can rob = 3 + 3 + 1 = 7.
```

```
	public int rob(TreeNode root) {
       if (root == null){
            return 0;
        }

        int val = 0;
        if (root.left != null){
            val += rob(root.left.left) + rob(root.left.right);
        }
        if (root.right != null){
            val += rob(root.right.left) + rob(root.right.right);
        }

        return Math.max(val + root.val,rob(root.left)+rob(root.right));
    }
```

LeetCode 100 Same Tree

判断二叉树是否相等

```
Input:     1         1
          / \       / \
         2   3     2   3

        [1,2,3],   [1,2,3]

Output: true
Input:     1         1
          /           \
         2             2

        [1,2],     [1,null,2]

Output: false
```

用DFS

```
public boolean isSameTree(TreeNode p, TreeNode q) {
        if(p == null && q == null){
            return true;
        }

        if (p == null || q == null){
            return false;
        }

        if (p.val != q.val){
            return false;
        }

        return isSameTree(p.left,q.left) && isSameTree(p.right,q.right);
    }
```

**LeetCode 671  Second Minimum Node In a Binary Tree**

找二叉树中的第二小的值。

```
Input: 
    2
   / \
  2   5
     / \
    5   7

Output: 5
Explanation: The smallest value is 2, the second smallest value is 5.
Input: 
    2
   / \
  2   2

Output: -1
Explanation: The smallest value is 2, but there isn't any second smallest value.
```

在不考虑题中的条件设置，直接使用DFS遍历，HashSet记录值。在记录值中找到第二小

```
	public int findSecondMinimumValue(TreeNode root) {
        HashSet<Integer> hashSet = new HashSet<>();

        dfs(root,hashSet);

        int min = Integer.MAX_VALUE;
        int minres = Integer.MAX_VALUE;

        if (hashSet.size() < 2){
            return -1;
        }
        for (Integer i: hashSet){
            if (minres > i){
                if (min > i){
                    minres = min;
                    min = i;
                }else {
                    minres = i;
                }
            }

        }

        return minres;
    }
    private void dfs(TreeNode root, HashSet<Integer> hashSet){
        if (root == null){
            return;
        }

        hashSet.add(root.val);

        if (root.left != null){
            dfs(root.left,hashSet);
        }

        if (root.right != null){
            dfs(root.right,hashSet);
        }
    }
```

LeetCode 572 Subtree of Another Tree

二叉树s，t，判断t是s的子树

```
Given tree s:

     3
    / \
   4   5
  / \
 1   2
Given tree t:
   4 
  / \
 1   2
Return true, because t has the same structure and node values with a subtree of s.
Given tree s:

     3
    / \
   4   5
  / \
 1   2
    /
   0
Given tree t:
   4
  / \
 1   2
```

深度优先遍历，需要遍历s，t；s.left，t；s.right，t；

```
public boolean isSubtree(TreeNode s, TreeNode t) {
        if (s == null){
            return false;
        }
        return dfs(s,t)||isSubtree(s.left,t)||isSubtree(s.right,t);

    }

    private boolean dfs(TreeNode s, TreeNode t){
        if (t == null && s==null){
            return true;
        }

        if (s == null || t == null){
            return false;
        }

        if (s.val != t.val){
            return false;
        }

        return dfs(s.right,t.right) && dfs(s.left,t.left);
    }
```

































