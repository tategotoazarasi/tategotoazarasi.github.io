---
title: "LeetCode 141. 环形链表"
date: 2021-12-22T10:17:23+08:00
draft: false
tags: ["leetcode", "简单", "哈希表", "链表", "双指针"]
math: true
---

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

如果链表中存在环，则返回 `true` 。 否则，返回 `false` 。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/circularlinkedlist.png)

> **输入：** head = [3,2,0,-4], pos = 1
> 
> **输出：** true
> 
> **解释：** 链表中有一个环，其尾部连接到第二个节点。

**示例  2：**

![](https://tategotoazarasi.github.io/images/circularlinkedlist_test2.png)

> **输入：** head = [1,2], pos = 0
> 
> **输出：** true
> 
> **解释：** 链表中有一个环，其尾部连接到第一个节点。

**示例 3：**

![](https://tategotoazarasi.github.io/images/circularlinkedlist_test3.png)

> **输入：** head = [1], pos = -1
> 
> **输出：** false
> 
> **解释：** 链表中没有环。

**提示：**

- \\(链表中节点的数目范围是 [0, 10^4]\\)
- \\(-10^5 <= Node.val <= 10^5\\)
- `pos` 为 `-1` 或者链表中的一个 **有效索引** 。

**进阶：** 你能用 `O(1)`（即，常量）内存解决此问题吗？

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null) {
            return false;
        }
        ListNode end = new ListNode(-100001);
        for (ListNode node = head; node.next != null; ) {
            if (node.next.val == end.val) {
                return true;
            } else {
                ListNode temp = node.next;
                node.next = end;
                node = temp;
            }
        }
        return false;
    }
}
```
