---
title: "LeetCode 101. 对称二叉树"
date: 2021-12-14T10:23:43+08:00
draft: false
tags: ["leetcode", "简单", "树", "深度优先搜索", "广度优先搜索", "二叉树"]
math: true
---

给定一个二叉树，检查它是否是镜像对称的。

<!--more-->

例如，二叉树  `[1,2,2,3,4,4,3]` 是对称的。

```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

但是下面这个  `[1,2,2,null,3,null,3]` 则不是镜像对称的:

```
    1
   / \
  2   2
   \   \
   3    3
```

**进阶：**

你可以运用递归和迭代两种方法解决这个问题吗？

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        reverse(root.left);
        return isSameTree(root.left, root.right);
    }

    private boolean isSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) {
            return true;
        }
        if (p == null || q == null) {
            return false;
        }
        return isSameTree(p.left, q.left) && p.val == q.val && isSameTree(p.right, q.right);
    }

    private void reverse(TreeNode node) {
        if (node == null) {
            return;
        } else if (node.left == null && node.right != null) {
            node.left = node.right;
            node.right = null;
        } else if (node.right == null && node.left != null) {
            node.right = node.left;
            node.left = null;
        } else {
            TreeNode temp = node.left;
            node.left = node.right;
            node.right = temp;
        }
        if (node.left != null) {
            reverse(node.left);
        }
        if (node.right != null) {
            reverse(node.right);
        }
    }
}
```
