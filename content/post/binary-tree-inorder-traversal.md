---
title: "LeetCode 94. 二叉树的中序遍历"
date: 2021-12-13T10:43:57+08:00
draft: false
tags: ["leetcode", "简单", "栈", "树", "深度优先搜索", "二叉树"]
math: true
---

给定一个二叉树的根节点 `root` ，返回它的 **中序**  遍历。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/inorder_1.jpg)

> **输入：** root = [1,null,2,3]
>
> **输出：** [1,3,2]

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
> **输出：** [2,1]

**示例 5：**

![](https://tategotoazarasi.github.io/images/inorder_4.jpg)

> **输入：** root = [1,null,2]
>
> **输出：** [1,2]

**提示：**

- 树中节点数目在范围 `[0, 100]` 内
- `-100 <= Node.val <= 100`

**进阶:**  递归算法很简单，你可以通过迭代算法完成吗？

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        if (root == null) {
            return new LinkedList<>();
        }
        List<Integer> list = new LinkedList<>();
        if (root.left != null) {
            list.addAll(inorderTraversal(root.left));
        }
        list.add(root.val);
        if (root.right != null) {
            list.addAll(inorderTraversal(root.right));
        }
        return list;
    }
}
```
