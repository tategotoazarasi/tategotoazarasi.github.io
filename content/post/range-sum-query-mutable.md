---
title: "LeetCode 307. Range Sum Query - Mutable"
date: 2022-04-04T12:46:58+08:00
draft: false
tags: ["leetcode", "中等", "设计", "树状数组", "线段树", "数组"]
math: true
---

Given an integer array `nums`, handle multiple queries of the following types:

1. **Update** the value of an element in `nums`.
2. Calculate the **sum** of the elements of `nums` between indices `left` and `right` **inclusive** where `left <= right`.

Implement the `NumArray` class:

- `NumArray(int[] nums)` Initializes the object with the integer array `nums`.
- `void update(int index, int val)` **Updates** the value of `nums[index]` to be `val`.
- `int sumRange(int left, int right)` Returns the **sum** of the elements of `nums` between indices `left` and `right` **inclusive** (i.e. `nums[left] + nums[left + 1] + ... + nums[right]`).

<!--more-->

**Example 1:**

**Input**

["NumArray", "sumRange", "update", "sumRange"]

[[[1, 3, 5]], [0, 2], [1, 2], [0, 2]]

**Output**

[null, 9, null, 8]

**Explanation**

```java
NumArray numArray = new NumArray([1, 3, 5]);
numArray.sumRange(0, 2); // return 1 + 3 + 5 = 9
numArray.update(1, 2); // nums = [1, 2, 5]
numArray.sumRange(0, 2); // return 1 + 2 + 5 = 8
```

**Constraints:**

- \\(1 <= nums.length <= 3 \* 10^4\\)
- `-100 <= nums[i] <= 100`
- `0 <= index < nums.length`
- `-100 <= val <= 100`
- `0 <= left <= right < nums.length`
- At most \\(3 \* 10^4\\) calls will be made to `update` and `sumRange`.

## Solution

线段树 \\(\textit{segmentTree}\\)是一个二叉树，每个结点保存数组 \\(\textit{nums}\\)在区间 \\([s, e]\\)的最小值、最大值或者总和等信息。

线段树可以用树也可以用数组（堆式存储）来实现。

对于数组实现，假设根结点的下标为 \\(0\\)，如果一个结点在数组的下标为 \\(\textit{node}\\)，那么它的左子结点下标为 \\(\textit{node} \times 2 + 1\\)，右子结点下标为 \\(\textit{node} \times 2 + 2\\)。

- 建树 \\(\textit{build}\\)函数

  我们在结点 \\(\textit{node}\\)保存数组 \\(\textit{nums}\\)在区间 \\([s, e]\\)的总和。

  - \\(s = e\\)时，结点 \\(\textit{node}\\)是叶子结点，它保存的值等于 \\(\textit{nums}[s]\\)。

  - \\(s < e\\)时，结点 \\(\textit{node}\\)的左子结点保存区间 \\(\Big [ s, \Big \lfloor \dfrac{s + e}{2} \Big \rfloor \Big ]\\)的总和，右子结点保存区间 \\(\Big [ \Big \lfloor \dfrac{s + e}{2} \Big \rfloor + 1, e \Big ]\\)的总和，那么结点 \\(\textit{node}\\)保存的值等于它的两个子结点保存的值之和。

  假设 \\(\textit{nums}\\)的大小为 \\(n\\)，我们规定根结点 \\(\textit{node} = 0\\)保存区间 \\([0, n - 1]\\)的总和，然后自下而上递归地建树。

- 单点修改 \\(\textit{change}\\)函数 当我们要修改 \\(\textit{nums}[\textit{index}]\\)的值时，我们找到对应区间 \\([\textit{index}, \textit{index}]\\)的叶子结点，直接修改叶子结点的值为 \\(\textit{val}\\)，并自下而上递归地更新父结点的值。

- 范围求和 \\(\textit{range}\\)函数 给定区间 \\([\textit{left}, \textit{right}]\\)时，我们将区间 \\([\textit{left}, \textit{right}]\\)拆成多个结点对应的区间。

  - 如果结点 \\(\textit{node}\\)对应的区间与 \\([\textit{left}, \textit{right}]\\)相同，可以直接返回该结点的值，即当前区间和。

  - 如果结点 \\(\textit{node}\\)对应的区间与 \\([\textit{left}, \textit{right}]\\)不同，设左子结点对应的区间的右端点为 \\(m\\)，那么将区间 \\([\textit{left}, \textit{right}]\\)沿点 \\(m\\)拆成两个区间，分别计算左子结点和右子结点。

  我们从根结点开始递归地拆分区间 \\([\textit{left}, \textit{right}]\\)。

**复杂度分析**

- 时间复杂度：

  - 构造函数：\\(O(n)\\)，其中 \\(n\\)是数组 \\(\textit{nums}\\)的大小。二叉树的高度不超过 \\(\lceil \log n \rceil + 1\\)，那么 \\(\textit{segmentTree}\\)的大小不超过 \\(2 ^ {\lceil \log n \rceil + 1} - 1 \le 4n\\)，所以 \\(\textit{build}\\)的时间复杂度为 \\(O(n)\\)。

  - \\(\textit{update}\\)函数：\\(O(\log n)\\)。因为树的高度不超过 \\(\lceil \log n \rceil + 1\\)，所以涉及更新的结点数不超过 \\(\lceil \log n \rceil + 1\\)。

  - \\(\textit{sumRange}\\)函数：\\(O(\log n)\\)。每层结点最多访问四个，总共访问的结点数不超过 \\(4 \times (\lceil \log n \rceil + 1)\\)。

- 空间复杂度：\\(O(n)\\)。保存 \\(\textit{segmentTree}\\)需要 \\(O(n)\\)的空间。

```cpp
class NumArray {
private:
    vector<int> segmentTree;
    int n;

    void build(int node, int s, int e, vector<int> &nums) {
        if (s == e) {
            segmentTree[node] = nums[s];
            return;
        }
        int m = s + (e - s) / 2;
        build(node * 2 + 1, s, m, nums);
        build(node * 2 + 2, m + 1, e, nums);
        segmentTree[node] = segmentTree[node * 2 + 1] + segmentTree[node * 2 + 2];
    }

    void change(int index, int val, int node, int s, int e) {
        if (s == e) {
            segmentTree[node] = val;
            return;
        }
        int m = s + (e - s) / 2;
        if (index <= m) {
            change(index, val, node * 2 + 1, s, m);
        } else {
            change(index, val, node * 2 + 2, m + 1, e);
        }
        segmentTree[node] = segmentTree[node * 2 + 1] + segmentTree[node * 2 + 2];
    }

    int range(int left, int right, int node, int s, int e) {
        if (left == s && right == e) {
            return segmentTree[node];
        }
        int m = s + (e - s) / 2;
        if (right <= m) {
            return range(left, right, node * 2 + 1, s, m);
        } else if (left > m) {
            return range(left, right, node * 2 + 2, m + 1, e);
        } else {
            return range(left, m, node * 2 + 1, s, m) + range(m + 1, right, node * 2 + 2, m + 1, e);
        }
    }

public:
    NumArray(vector<int>& nums) : n(nums.size()), segmentTree(nums.size() * 4) {
        build(0, 0, n - 1, nums);
    }

    void update(int index, int val) {
        change(index, val, 0, 0, n - 1);
    }

    int sumRange(int left, int right) {
        return range(left, right, 0, 0, n - 1);
    }
};
```
