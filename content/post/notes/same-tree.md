---
title: same-tree
tags: [Import-7704]
created: '2021-12-18T11:18:20.412Z'
modified: '2021-12-30T13:58:04.045Z'
---

---
title: "LeetCode 100. 相同的树"
date: 2021-12-14T10:03:50+08:00
draft: false
tags: ["leetcode", "简单", "树", "深度优先搜索", "广度优先搜索", "二叉树"]
math: true
---

给你两棵二叉树的根节点 `p` 和 `q` ，编写一个函数来检验这两棵树是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/ex1.jpg)

> **输入：** p = [1,2,3], q = [1,2,3]
> 
> **输出：** true

**示例 2：**

![](https://tategotoazarasi.github.io/images/ex2.jpg)

> **输入：** p = [1,2], q = [1,null,2]
> 
> **输出：** false

**示例 3：**

![](https://tategotoazarasi.github.io/images/ex3.jpg)

> **输入：** p = [1,2,1], q = [1,1,2]
> 
> **输出：** false

**提示：**

- 两棵树上的节点数目都在范围 `[0, 100]` 内
- \\(-10^4 <= Node.val <= 10^4\\)

```java
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) {
            return true;
        }
        if (p == null || q == null) {
            return false;
        }
        return isSameTree(p.left, q.left) && p.val == q.val && isSameTree(p.right, q.right);
    }
}
```
