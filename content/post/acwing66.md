---
title: "AcWing 66. 两个链表的第一个公共结点"
date: 2022-03-06T16:18:54+08:00
draft: false
tags: ["acwing", "简单", "链表"]
math: true
---

输入两个链表，找出它们的第一个公共结点。

当不存在公共节点时，返回空节点。

<!--more-->

#### 数据范围

链表长度 [1,2000]。

#### 样例

```
给出两个链表如下所示：
A：        a1 → a2
                   ↘
                     c1 → c2 → c3
                   ↗
B:     b1 → b2 → b3

输出第一个公共节点c1
```

## 题解

如果有公共结点肯定是在后面重叠，且后面部分都是共同的。

#### 方法 1

先计算出两个链表的长度，可以让比较长的先走两个链表长度之差的步数，两个再一起走。

```cpp
class Solution {
public:
    ListNode *findFirstCommonNode(ListNode *headA, ListNode *headB) {
        auto p = headA, q = headB;
        int la = 0, lb = 0;
        for (auto t = headA; t; t = t->next) la ++;
        for (auto t = headB; t; t = t->next) lb ++;
        int k = la - lb;
        if (la < lb) {
            p = headB, q = headA;
            k = lb - la;
        }
        while(k --) {
            p = p->next;
        }
        while(p) {
            if (p == q) return p;
            p = p->next;
            q = q->next;
        }
        return nullptr;
    }
};
```

#### 方法 2

不同部分为 a， 和 b，公共部分为 c；a + c + b = b + c + a;让两个一起走，a 走到头就转向 b， b 走到头转向 a，则在公共部分相遇。

```cpp
class Solution {
public:
    ListNode *findFirstCommonNode(ListNode *headA, ListNode *headB) {
        auto p = headA, q = headB;
        while(p != q) {
            if(p) p = p->next;
            else p = headB;
            if (q) q = q->next;
            else q = headA;
        }
        return p;
    }
};
```
