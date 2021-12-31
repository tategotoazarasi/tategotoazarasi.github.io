---
title: maximum-depth-of-binary-tree
tags: [Import-7704]
created: '2021-12-18T11:18:20.411Z'
modified: '2021-12-30T13:58:02.512Z'
---

---
title: "LeetCode 104. 二叉树的最大深度"
date: 2021-12-15T09:52:08+08:00
draft: false
tags: ["leetcode", "简单", "树", "深度优先搜索", "广度优先搜索", "二叉树"]
math: true
---

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

<!--more-->

**说明:**  叶子节点是指没有子节点的节点。

**示例：**  
给定二叉树 `[3,9,20,null,null,15,7]`，

```
    3
   / \
  9  20
    /  \
   15   7
```

返回它的最大深度  3 。

```java
class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        if (root.left == null && root.right == null) {
            return 1;
        }
        if (root.left != null && root.right != null) {
            return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
        }
        return maxDepth(Objects.requireNonNullElseGet(root.left, () -> root.right)) + 1;
    }
}
```
