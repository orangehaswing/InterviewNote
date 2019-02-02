BST
---

主要利用 BST 中序遍历有序的特点。

在 BST 中寻找两个节点，使它们的和为一个给定值

653. Two Sum IV - Input is a BST (Easy)

Input:
    5
   / \
  3   6
 / \   \
2   4   7

Target = 9

Output: True
使用中序遍历得到有序数组之后，再利用双指针对数组进行查找。

应该注意到，这一题不能用分别在左右子树两部分来处理这种思想，因为两个待求的节点可能分别在左右子树中。

    public boolean findTarget(TreeNode root, int k) {
        List<Integer> nums = new ArrayList<>();
        inOrder(root, nums);
        int i = 0, j = nums.size() - 1;
        while (i < j){
            int sum = nums.get(i) + nums.get(j);
            if (sum == k) return true;
            if (sum < k) i++;
            else j--;
        }
        return false;
    }
    
    private void inOrder(TreeNode root, List<Integer> nums){
        if (root == null) return;
        inOrder(root.left, nums);
        nums.add(root.val);
        inOrder(root.right, nums);
    }

在 BST 中查找两个节点之差的最小绝对值

530. Minimum Absolute Difference in BST (Easy)

Input:
   1
    \
     3
    /
   2

Output:
1
利用 BST 的中序遍历为有序的性质，计算中序遍历中临近的两个节点之差的绝对值，取最小值。

    private int minDiff = Integer.MAX_VALUE;
    private int preVal = -1;
    
    public int getMinimumDifference(TreeNode root) {
        inorder(root);
        return minDiff;
    }
    
    private void inorder(TreeNode node){
        if (node == null) return;
        inorder(node.left);
        if (preVal != -1) minDiff = Math.min(minDiff, Math.abs(node.val - preVal));
        preVal = node.val;
        inorder(node.right);
    }

把 BST 每个节点的值都加上比它大的节点的值

Convert BST to Greater Tree (Easy)

Input: The root of a Binary Search Tree like this:
              5
            /   \
           2     13

Output: The root of a Greater Tree like this:
             18
            /   \
          20     13
先遍历右子树。

    private int sum = 0;
    public TreeNode convertBST(TreeNode root) {
        traver(root);
        return root;
    }
    
    private void traver(TreeNode root) {
        if (root == null) return;
        if (root.right != null) traver(root.right);
        sum += root.val;
        root.val = sum;
        if (root.left != null) traver(root.left);
    }

寻找 BST 中出现次数最多的节点

501. Find Mode in Binary Search Tree (Easy)

   1
    \
     2
    /
   2
return [2].

    private int cnt = 1;
    private int maxCnt = 1;
    private TreeNode preNode = null;
    private List<Integer> list;
    
    public int[] findMode(TreeNode root) {
        list = new ArrayList<>();
        inOrder(root);
        int[] ret = new int[list.size()];
        int idx = 0;
        for (int num : list) {
            ret[idx++] = num;
        }
        return ret;
    }
    
    private void inOrder(TreeNode node) {
        if (node == null) return;
        inOrder(node.left);
        if (preNode != null) {
            if (preNode.val == node.val) cnt++;
            else cnt = 1;
        }
        if (cnt > maxCnt) {
            maxCnt = cnt;
            list.clear();
            list.add(node.val);
        } else if (cnt == maxCnt) {
            list.add(node.val);
        }
        preNode = node;
        inOrder(node.right);
    }

寻找 BST 的第 k 个元素

230. Kth Smallest Element in a BST (Medium)

递归解法：

    public int kthSmallest(TreeNode root, int k) {
        int leftCnt = count(root.left);
        if (leftCnt == k - 1) return root.val;
        if (leftCnt > k - 1) return kthSmallest(root.left, k);
        return kthSmallest(root.right, k - leftCnt - 1);
    }
    
    private int count(TreeNode node) {
        if (node == null) return 0;
        return 1 + count(node.left) + count(node.right);
    }

中序遍历解法：

    private int cnt = 0;
    private int val;
    
    public int kthSmallest(TreeNode root, int k) {
        inOrder(root, k);
        return val;
    }
    
    private void inOrder(TreeNode node, int k) {
        if (node == null) return;
        inOrder(node.left, k);
        cnt++;
        if (cnt == k) {
            val = node.val;
            return;
        }
        inOrder(node.right, k);
    }

根据有序链表构造平衡的 BST

109. Convert Sorted List to Binary Search Tree (Medium)

Given the sorted linked list: [-10,-3,0,5,9],

One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:

      0
     / \
   -3   9
   /   /
 -10  5

    public TreeNode sortedListToBST(ListNode head) {
        if (head == null) return null;
        int size = size(head);
        if (size == 1) return new TreeNode(head.val);
        ListNode pre = head, mid = pre.next;
        int step = 2;
        while (step <= size / 2) {
            pre = mid;
            mid = mid.next;
            step++;
        }
        pre.next = null;
        TreeNode t = new TreeNode(mid.val);
        t.left = sortedListToBST(head);
        t.right = sortedListToBST(mid.next);
        return t;
    }
    
    private int size(ListNode node) {
        int size = 0;
        while (node != null) {
            size++;
            node = node.next;
        }
        return size;
    }


tags:数据结构


