---
title: binary-tree-postorder-traversal
tags: [Import-7704]
created: '2021-12-23T06:40:42.237Z'
modified: '2021-12-30T13:57:59.985Z'
---

---
title: "LeetCode 145. 二叉树的后序遍历"
date: 2021-12-23T14:40:42+08:00
draft: false
tags: ["leetcode", "简单", "栈", "树", "深度优先搜索", "二叉树"]
math: true
---

给定一个二叉树，返回它的 _后序_ 遍历。

<!--more-->

**示例:**

**输入:** [1,null,2,3]

```
   1
    \
     2
    /
   3
```

**输出:** [3,2,1]

**进阶:**  递归算法很简单，你可以通过迭代算法完成吗？

```java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> ret = new LinkedList<>();
        if (root == null) {
            return ret;
        }
        if (root.left != null) {
            ret.addAll(postorderTraversal(root.left));
        }
        if (root.right != null) {
            ret.addAll(postorderTraversal(root.right));
        }
        ret.add(root.val);
        return ret;
    }
}
```
