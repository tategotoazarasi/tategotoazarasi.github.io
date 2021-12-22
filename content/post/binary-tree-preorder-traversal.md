---
title: "LeetCode 144. 二叉树的前序遍历"
date: 2021-12-22T10:33:10+08:00
draft: false
tags: ["leetcode", "简单", "栈", "树", "深度优先搜索", "二叉树"]
math: true
---

给你二叉树的根节点 `root` ，返回它节点值的  **前序** 遍历。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/inorder_1.jpg)

> **输入：** root = [1,null,2,3]
> 
> **输出：** [1,2,3]

**示例 2：**

> **输入：** root = []
> 
> **输出：** []

**示例 3：**

> **输入：** root = [1]
> 
> **输出：** [1]

**示例 4：**

![](https://tategotoazarasi.github.io/images/inorder_5.jpg)

> **输入：** root = [1,2]
> 
> **输出：** [1,2]

**示例 5：**

![](https://tategotoazarasi.github.io/images/inorder_4.jpg)

> **输入：** root = [1,null,2]
> 
> **输出：** [1,2]

**提示：**

- 树中节点数目在范围 `[0, 100]` 内
- `-100 <= Node.val <= 100`

**进阶：** 递归算法很简单，你可以通过迭代算法完成吗？

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> ret = new LinkedList<Integer>();
        if (root == null) {
            return ret;
        }
        ret.add(root.val);
        ret.addAll(preorderTraversal(root.left));
        ret.addAll(preorderTraversal(root.right));
        return ret;
    }
}
```
