---
title: "LeetCode 111. 二叉树的最小深度"
date: 2021-12-16T10:23:00+08:00
draft: false
tags: ["leetcode", "简单", "树", "深度优先搜索", "广度优先搜索", "二叉树"]
math: true
---

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

<!--more-->

**说明：** 叶子节点是指没有子节点的节点。

**示例 1：**

![](https://tategotoazarasi.github.io/images/ex_depth.jpg)

> **输入：** root = [3,9,20,null,null,15,7]
> 
> **输出：** 2

**示例 2：**

> **输入：** root = [2,null,3,null,4,null,5,null,6]
> 
> **输出：** 5

**提示：**

- \\(树中节点数的范围在 `[0, 10^5]` 内\\)
- `-1000 <= Node.val <= 1000`

```java
class Solution {
    public int minDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        root.val = 1;
        queue.add(root);
        while (!queue.isEmpty()) {
            TreeNode node = queue.remove();
            if (node.left == null && node.right == null) {
                return node.val;
            }
            if (node.left != null) {
                node.left.val = node.val + 1;
                queue.add(node.left);
            }
            if (node.right != null) {
                node.right.val = node.val + 1;
                queue.add(node.right);
            }
        }
        return 0;
    }
}
```
