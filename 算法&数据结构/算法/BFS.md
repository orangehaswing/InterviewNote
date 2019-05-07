# BSF

LeetCode 752 Open the Lock

四位数的锁，初始状态是“0000”。每一位都可以加或减，分别有10种状态：'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'

9可以拨到0,0可以拨到9。但是不能拨到死锁的状态`deadends`。最少多少次可以拨到给定密码值

```
Input: deadends = ["0201","0101","0102","1212","2002"], target = "0202"
Output: 6
Explanation:
A sequence of valid moves would be "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202".
Note that a sequence like "0000" -> "0001" -> "0002" -> "0102" -> "0202" would be invalid,
because the wheels of the lock become stuck after the display becomes the dead end "0102".

Input: deadends = ["8888"], target = "0009"
Output: 1
Explanation:
We can turn the last wheel in reverse to move from "0000" -> "0009".

Input: deadends = ["8887","8889","8878","8898","8788","8988","7888","9888"], target = "8888"
Output: -1
Explanation:
We can't reach the target without getting stuck.
```

广度优先寻找，将每一位加一或减一，保存在hashset中。如果找到目标值，输出层数，否则对于每一次的密码，都进行每一位加一或减一。如果遇到deadends，跳过这次。

```
public int openLock(String[] deadends, String target) {
        Queue<String> queue = new LinkedList<>();

        HashSet<String> dead = new HashSet<>(Arrays.asList(deadends));
        HashSet<String> visited = new HashSet<>();
        queue.offer("0000");
        visited.add("0000");
        int index = 0;
        //BSF
        while (!queue.isEmpty()){
            int size = queue.size();

            while (size > 0){
                String s = queue.poll();

                if (dead.contains(s)){
                    size--;
                    continue;
                }

                if (s.equals(target)){
                    return index;
                }

                for (int i = 0; i < 4; i++) {
                    char c = s.charAt(i);
                    //增
                    String s1 = s.substring(0,i) + (c=='9'?0:c-'0'+1) + s.substring(i+1);
                    //减
                    String s2 = s.substring(0,i) + (c=='0'?9:c-'0'-1) + s.substring(i+1);
                    if (!visited.contains(s1) && !dead.contains(s1)){
                        queue.offer(s1);
                        visited.add(s1);
                    }
                    if (!visited.contains(s2) && !dead.contains(s2)){
                        queue.offer(s2);
                        visited.add(s2);
                    }
                }
                size--;
            }
        index++;
        }
        return -1;
    }
```

LeetCode 102 Binary Tree Level Order Traversal

给定二叉树，从左到右，层次遍历输出节点值

```
Given binary tree [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
return its level order traversal as:
[
  [3],
  [9,20],
  [15,7]
]
```

BFS

```
	public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null){
            return res;
        }

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()){
            int size = queue.size();
            ArrayList<Integer> temp = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                temp.add(node.val);
                if (node.left != null){
                    queue.offer(node.left);
                }
                if (node.right != null){
                    queue.offer(node.right);
                }
            }
            res.add(temp);
        }
        
        return res;
    }
```











