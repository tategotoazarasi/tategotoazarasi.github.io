---
title: "AcWing 36. 合并两个排序的链表"
date: 2022-03-07T15:40:12+08:00
draft: false
tags: ["acwing", "简单", "链表", "二路归并"]
math: true
---

输入两个递增排序的链表，合并这两个链表并使新链表中的结点仍然是按照递增排序的。

<!--more-->

#### 数据范围

链表长度 [0,500]。

#### 样例

```
输入：1->3->5 , 2->4->5

输出：1->2->3->4->5->5
```

## 题解

#### 算法

(二路归并) O(n)

1. 新建头部的保护结点 head，设置 current 指针指向 head。
2. 若当前 l1 指针指向的结点的值 val 比 l2 指针指向的结点的值 val 小，则令 current 的 next 指针指向 l1，且 l1 后移；否则指向 l2，且 l2 后移。
3. 然后 current 指针按照上一部设置好的位置后移。
4. 循环以上步骤直到 l1 或 l2 为空。
5. 将剩余的 l1 或 l2 接到 current 指针后边。

#### 时间复杂度

两个链表各遍历一次，所以时间复杂度为 O(n)

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* merge(ListNode* l1, ListNode* l2) {
        ListNode *current = new ListNode(0);
        ListNode *head    = current;
        while(l1 != nullptr && l2 != nullptr) {
            if(l1->val < l2->val) {
                current->next = l1;
                l1            = l1->next;
            } else {
                current->next = l2;
                l2            = l2->next;
            }
            current = current->next;
        }
        if(l1 == nullptr) {
            current->next = l2;
        } else {
            current->next = l1;
        }
        return head->next;
    }
};
```
