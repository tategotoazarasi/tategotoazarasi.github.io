---
title: "LeetCode 382. 链表随机节点"
date: 2022-01-16T10:00:10+08:00
draft: false
tags: ["leetcode", "中等", "水塘抽样", "链表", "数学", "随机化"]
math: true
---

给你一个单链表，随机选择链表的一个节点，并返回相应的节点值。每个节点 **被选中的概率一样** 。

<!--more-->

实现 `Solution` 类：

- `Solution(ListNode head)` 使用整数数组初始化对象。
- `int getRandom()` 从链表中随机选择一个节点并返回该节点的值。链表中所有节点被选中的概率相等。

**示例：**

![](https://tategotoazarasi.github.io/images/getrand-linked-list.jpg)

**输入**
["Solution", "getRandom", "getRandom", "getRandom", "getRandom", "getRandom"]
[[[1, 2, 3]], [], [], [], [], []]

**输出**
[null, 1, 3, 2, 2, 3]

**解释**
Solution solution = new Solution([1, 2, 3]);
solution.getRandom(); // 返回 1
solution.getRandom(); // 返回 3
solution.getRandom(); // 返回 2
solution.getRandom(); // 返回 2
solution.getRandom(); // 返回 3
// getRandom() 方法应随机返回 1、2、3 中的一个，每个元素被返回的概率相等。

**提示：**

- \\(链表中的节点数在范围 [1, 10^4] 内\\)
- \\(-10^4 <= Node.val <= 10^4\\)
- \\(至多调用 getRandom 方法 10^4 次\\)

**进阶：**

- 如果链表非常大且长度未知，该怎么处理？
- 你能否在不使用额外空间的情况下解决此问题？

## 题解

从链表头开始，遍历整个链表，对遍历到的第 i 个节点，随机选择区间 [0,i) 内的一个整数，如果其等于 0，则将答案置为该节点值，否则答案不变。

该算法会保证每个节点的值成为最后被返回的值的概率均为 \\(\dfrac{1}{n}\\)，证明如下：

$$\begin{aligned} &P(第\ i\ 个节点的值成为最后被返回的值)\\=&P(第\ i\ 次随机选择的值= 0) \times P(第\ i+1\ 次随机选择的值\ne 0) \times \cdots \times P(第\ n\ 次随机选择的值\ne 0)\\=&\dfrac{1}{i} \times (1-\dfrac{1}{i+1}) \times \cdots \times (1-\dfrac{1}{n})\\=&\dfrac{1}{i} \times \dfrac{i}{i+1} \times \cdots \times \dfrac{n-1}{n}\\=&\dfrac{1}{n}\end{aligned}$$

### 复杂度分析

- 时间复杂度：初始化为 O(1)，随机选择为 O(n)，其中 n 是链表的元素个数。

- 空间复杂度：O(1)。我们只需要常数的空间保存若干变量。

```cpp
class Solution {
    ListNode *head;

public:
    Solution(ListNode *head) {
        this->head = head;
    }

    int getRandom() {
        int i = 1, ans = 0;
        for (auto node = head; node; node = node->next) {
            if (rand() % i == 0) { // 1/i 的概率选中（替换为答案）
                ans = node->val;
            }
            ++i;
        }
        return ans;
    }
};
```
