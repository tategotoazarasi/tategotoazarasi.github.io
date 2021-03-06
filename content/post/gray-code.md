---
title: "LeetCode 89. 格雷编码"
date: 2022-01-08T16:57:11+08:00
draft: false
tags: ["leetcode", "中等", "位运算", "数学", "回溯"]
math: true
---

**n 位格雷码序列** 是一个由 `2n` 个整数组成的序列，其中：

- 每个整数都在范围 `[0, 2n - 1]` 内（含 `0` 和 `2n - 1`）
- 第一个整数是 `0`
- 一个整数在序列中出现 **不超过一次**
- 每对 **相邻** 整数的二进制表示 **恰好一位不同** ，且
- **第一个** 和 **最后一个** 整数的二进制表示 **恰好一位不同**

给你一个整数 `n` ，返回任一有效的 **n 位格雷码序列** 。

<!--more-->

**示例 1：**

**输入：** n = 2
**输出：** [0,1,3,2]
**解释：**
[0,1,3,2] 的二进制表示是 [00,01,11,10] 。

- 0 **_0_** 和 0 **_1_** 有一位不同
- **_0_** 1 和 **_1_** 1 有一位不同
- 1 **_1_** 和 1 **_0_** 有一位不同
- **_1_** 0 和 **_0_** 0 有一位不同
  [0,2,3,1] 也是一个有效的格雷码序列，其二进制表示是 [00,10,11,01] 。
- **_0_** 0 和 **_1_** 0 有一位不同
- 1 **_0_** 和 1 **_1_** 有一位不同
- **_1_** 1 和 **_0_** 1 有一位不同
- 0 **_1_** 和 0 **_0_** 有一位不同

**示例 2：**

**输入：** n = 1
**输出：** [0,1]

**提示：**

- `1 <= n <= 16`

## 题解

#### 前言

关于格雷编码的知识，读者可以查阅百度百科「[格雷码](https://baike.baidu.com/item/%E6%A0%BC%E9%9B%B7%E7%A0%81)」。

#### 方法一：对称生成

**思路与算法**

假设我们已经获取到 n−1 位的格雷码序列 \\(G\_{n-1}\\)，我们只需要将 \\(G\_{n-1}\\) 对称翻转，记作 \\(G\_{n-1}^T\\)。\\(G\_{n-1}\\) 的首元素和 \\(G\_{n-1}^T\\) 的尾元素都是相同的，反之亦然。如果我们给 \\(G\_{n-1}^T\\) 的每个元素都加上 2n−1，记作 \\((G\_{n-1}^T)^{'}\\)，则 \\(G\_{n-1}\\)的首元素和 \\((G\_{n-1}^T)^{'}\\) 的尾元素只有一位不相同，反之亦然。因此 \\(G\_{n-1}\\) 和 \\((G\_{n-1}^T)^{'}\\) 拼接的序列 \\(G_n=[G\_{n-1},(G\_{n-1}^T)^{'}]\\) 满足 n 位的格雷码的定义。初始值 \\(G_0=[0]\\)。

**代码**

```cpp
class Solution {
public:
    vector<int> grayCode(int n) {
        vector<int> ret;
        ret.reserve(1 << n);
        ret.push_back(0);
        for (int i = 1; i <= n; i++) {
            int m = ret.size();
            for (int j = m - 1; j >= 0; j--) {
                ret.push_back(ret[j] | (1 << (i - 1)));
            }
        }
        return ret;
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(2^n)\\)，其中 n 为格雷码序列的位数。递推过程的时间复杂度为 \\(O( {\textstyle \sum\_{i=1}^{n}2^{i-1}} )=O(2^n)\\)。

- 空间复杂度：O(1)。注意返回值不计入空间复杂度。

#### 方法二：二进制数转格雷码

**思路与算法**

如果我们有一个二进制数序列，我们也可以将它直接转换成格雷码序列。假设 n 位二进制数为 b，对应的格雷码为 g，转换规则如下：

g(i)=b(i+1)⊕b(i), 0≤i<n

其中 ⊕ 是按位异或运算，g(i) 和 b(i) 分别表示 g 和 b 的第 i 位，且 b(n)=0。

上述转换规则的证明如下：

> 考虑 n 位二进制数 \\( b_i \\) 和对应的转换码 \\(g_i\\)，并且 \\(b\_{i+1}=b_i+1\\) 也是 n 位二进制数。\\(b\_{i+1}\\) 与 \\(b_i\\) 的区别在于 \\(b\_{i+1}\\)将 \\(b_i\\) 二进制下末位连接的 1 全部变成 0，然后将最低位的 0 变成 1。假设变化涉及到的二进制位数为 k 位，则按照上述转换规则， \\(g\_{i+1}\\)与 \\(g_i\\)只有在第 k−1 位不相同，其他位都相同。因此转换得到的码相邻的数只有一位不同，而转换码第一个整数和最后一个整数分别由二进制数 0 和 \\(2^n−1\\) 转换而来，也只有一位不同。因为二进制数的取值范围为 \\([0, 2^n)\\)，且上述转换规则为一对一映射，因此得到的转换码也是互不相同的，且取值范围也在 \\([0, 2^n)\\)，得证。

**代码**

```cpp
class Solution {
public:
    vector<int> grayCode(int n) {
        vector<int> ret(1 << n);
        for (int i = 0; i < ret.size(); i++) {
            ret[i] = (i >> 1) ^ i;
        }
        return ret;
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(2^n)\\)，其中 n 为格雷码序列的位数。每个整数转换为格雷码的时间复杂度为 O(1)，总共有 2n 个转换。

- 空间复杂度：O(1)。注意返回值不计入空间复杂度。
