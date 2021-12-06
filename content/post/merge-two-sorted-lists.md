---
title: "LeetCode 21. 合并两个有序链表"
date: 2021-12-06T08:53:47+08:00
draft: false
tags: ["leetcode", "简单", "递归", "链表"]
math: true
---

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

**示例 1：**

![](https://tategotoazarasi.github.io/images/merge_ex1.jpg)

> **输入：** l1 = [1,2,4], l2 = [1,3,4]
>
> **输出：** [1,1,2,3,4,4]

**示例 2：**

> **输入：** l1 = [], l2 = []
>
> **输出：** []

**示例 3：**

> **输入：** l1 = [], l2 = [0]
>
> **输出：** [0]

**提示：**

- 两个链表的节点数目范围是 `[0, 50]`
- `-100 <= Node.val <= 100`
- `l1` 和 `l2` 均按 **非递减顺序** 排列

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode list = new ListNode();
        ListNode ret = list;
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                list.next = new ListNode(list1.val);
                list = list.next;
                list1 = list1.next;
            } else {
                list.next = new ListNode(list2.val);
                list = list.next;
                list2 = list2.next;
            }
        }
        if (list1 == null) {
            list.next = list2;
        } else {
            list.next = list1;
        }
        return ret.next;
    }
}
```