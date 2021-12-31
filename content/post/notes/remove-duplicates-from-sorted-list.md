---
title: remove-duplicates-from-sorted-list
tags: [Import-7704]
created: '2021-12-18T11:18:20.412Z'
modified: '2021-12-30T13:58:03.667Z'
---

---
title: "LeetCode 83. 删除排序链表中的重复元素"
date: 2021-12-12T17:09:28+08:00
draft: false
tags: ["leetcode", "简单", "链表"]
math: true
---

存在一个按升序排列的链表，给你这个链表的头节点 `head` ，请你删除所有重复的元素，使每个元素 **只出现一次** 。

返回同样按升序排列的结果链表。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/list1.jpg)

> **输入：** head = [1,1,2]
>
> **输出：** [1,2]

**示例 2：**

![](https://tategotoazarasi.github.io/images/list2.jpg)

> **输入：** head = [1,1,2,3,3]
>
> **输出：** [1,2,3]

**提示：**

- 链表中节点数目在范围 `[0, 300]` 内
- `-100 <= Node.val <= 100`
- 题目数据保证链表已经按升序排列

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) {
            return null;
        }
        for (ListNode iterator = head; iterator.next != null; iterator = iterator.next) {
            while (iterator.val == iterator.next.val) {
                if (iterator.next.next == null) {
                    iterator.next = null;
                    return head;
                }
                iterator.next = iterator.next.next;
            }
        }
        return head;
    }
}
```
