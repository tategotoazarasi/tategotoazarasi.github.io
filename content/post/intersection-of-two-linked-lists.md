---
title: "LeetCode 160. 相交链表"
date: 2021-12-24T08:29:53+08:00
draft: false
tags: ["leetcode", "简单", "哈希表", "链表", "双指针"]
math: true
---

给你两个单链表的头节点  `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 `null` 。

图示两个链表在节点 `c1` 开始相交 **：**

![](https://tategotoazarasi.github.io/images/160_statement.png)

题目数据 **保证** 整个链式结构中不存在环。

<!--more-->

**注意**，函数返回结果后，链表必须 **保持其原始结构** 。

**自定义评测：**

**评测系统** 的输入如下（你设计的程序 **不适用** 此输入）：

- `intersectVal` - 相交的起始节点的值。如果不存在相交节点，这一值为 `0`
- `listA` - 第一个链表
- `listB` - 第二个链表
- `skipA` - 在 `listA` 中（从头节点开始）跳到交叉节点的节点数
- `skipB` - 在 `listB` 中（从头节点开始）跳到交叉节点的节点数

评测系统将根据这些输入创建链式数据结构，并将两个头节点 `headA` 和 `headB` 传递给你的程序。如果程序能够正确返回相交节点，那么你的解决方案将被 **视作正确答案** 。

**示例 1：**

![](https://tategotoazarasi.github.io/images/160_example_1_1.png)

> **输入：** intersectVal = 8, listA = [4,1,8,4,5], listB = [5,6,1,8,4,5], skipA = 2, skipB = 3
> 
> **输出：** Intersected at '8'
> 
> **解释：** 相交节点的值为 8 （注意，如果两个链表相交则不能为 0）。
> 
> 从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,6,1,8,4,5]。
> 
> 在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。

**示例  2：**

![](https://tategotoazarasi.github.io/images/160_example_2.png)

> **输入：** intersectVal = 2, listA = [1,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
> 
> **输出：** Intersected at '2'
> 
> **解释：** 相交节点的值为 2 （注意，如果两个链表相交则不能为 0）。
> 
> 从各自的表头开始算起，链表 A 为 [1,9,1,2,4]，链表 B 为 [3,2,4]。
> 
> 在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。

**示例  3：**

![](https://tategotoazarasi.github.io/images/160_example_3.png)

> **输入：** intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
> 
> **输出：** null
> 
> **解释：** 从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。
> 
> 由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
> 
> 这两个链表不相交，因此返回 null 。

**提示：**

- `listA` 中节点数目为 `m`
- `listB` 中节点数目为 `n`
- \\(1 <= m, n <= 3 \* 10^4\\)
- \\(1 <= Node.val <= 10^5\\)
- `0 <= skipA <= m`
- `0 <= skipB <= n`
- 如果 `listA` 和 `listB` 没有交点，`intersectVal` 为 `0`
- 如果 `listA` 和 `listB` 有交点，`intersectVal == listA[skipA] == listB[skipB]`

**进阶：** 你能否设计一个时间复杂度 `O(m + n)` 、仅用 `O(1)` 内存的解决方案？

## 题解

**思路和算法**

使用双指针的方法，可以将空间复杂度降至 O(1)。

只有当链表 headA 和 headB 都不为空时，两个链表才可能相交。因此首先判断链表 headA 和 headB 是否为空，如果其中至少有一个链表为空，则两个链表一定不相交，返回 null。

当链表 headA 和 headB 都不为空时，创建两个指针 pA 和 pB，初始时分别指向两个链表的头节点 headA 和 headB，然后将两个指针依次遍历两个链表的每个节点。具体做法如下：

- 每步操作需要同时更新指针 pA 和 pB。

- 如果指针 pA 不为空，则将指针 pA 移到下一个节点；如果指针 pB 不为空，则将指针 pB 移到下一个节点。

- 如果指针 pA 为空，则将指针 pA 移到链表 headB 的头节点；如果指针 pB 为空，则将指针 pB 移到链表 headA 的头节点。

- 当指针 pA 和 pB 指向同一个节点或者都为空时，返回它们指向的节点或者 null。

**证明**

下面提供双指针方法的正确性证明。考虑两种情况，第一种情况是两个链表相交，第二种情况是两个链表不相交。

情况一：两个链表相交

链表 headA 和 headB 的长度分别是 m 和 n。假设链表 headA 的不相交部分有 a 个节点，链表 headB 的不相交部分有 b 个节点，两个链表相交的部分有 c 个节点，则有 a+c=m，b+c=n。

- 如果 a=b，则两个指针会同时到达两个链表相交的节点，此时返回相交的节点；

- 如果 a​=b，则指针 pA 会遍历完链表 headA，指针 pB 会遍历完链表 headB，两个指针不会同时到达链表的尾节点，然后指针 pA 移到链表 headB 的头节点，指针 pB 移到链表 headA 的头节点，然后两个指针继续移动，在指针 pA 移动了 a+c+b 次、指针 pB 移动了 b+c+a 次之后，两个指针会同时到达两个链表相交的节点，该节点也是两个指针第一次同时指向的节点，此时返回相交的节点。

情况二：两个链表不相交

链表 headA 和 headB 的长度分别是 m 和 n。考虑当 m=n 和 m​=n 时，两个指针分别会如何移动：

- 如果 m=n，则两个指针会同时到达两个链表的尾节点，然后同时变成空值 null，此时返回 null；

- 如果 m​=n，则由于两个链表没有公共节点，两个指针也不会同时到达两个链表的尾节点，因此两个指针都会遍历完两个链表，在指针 pA 移动了 m+n 次、指针 pB 移动了 n+m 次之后，两个指针会同时变成空值 null，此时返回 null。

**复杂度分析**

- 时间复杂度：O(m+n)，其中 m 和 n 是分别是链表 headA 和 headB 的长度。两个指针同时遍历两个链表，每个指针遍历两个链表各一次。

- 空间复杂度：O(1)。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        HashSet<ListNode> set = new HashSet<>();
        for (ListNode nodeA = headA, nodeB = headB; nodeA != null || nodeB != null; nodeA = nodeA == null ? null : nodeA.next, nodeB = nodeB == null ? null : nodeB.next) {
            if (nodeA != null) {
                if (set.contains(nodeA)) {
                    return nodeA;
                } else {
                    set.add(nodeA);
                }
            }
            if (nodeB != null) {
                if (set.contains(nodeB)) {
                    return nodeB;
                } else {
                    set.add(nodeB);
                }
            }
        }
        return null;
    }
}
```
