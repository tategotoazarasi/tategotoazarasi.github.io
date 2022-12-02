---
title: "LeetCode 1588. 所有奇数长度子数组的和"
date: 2022-12-02T14:13:41+08:00
draft: false
tags: ["leetcode", "简单", "数组", "数学", "前缀和"]
math: true
---

给你一个正整数数组 `arr` ，请你计算所有可能的奇数长度子数组的和。

**子数组** 定义为原数组中的一个连续子序列。

请你返回 `arr` 中 **所有奇数长度子数组的和** 。

<!--more-->

**示例 1：**

    输入：arr = [1,4,2,5,3]
    输出：58
    解释：所有奇数长度子数组和它们的和为：
    [1] = 1
    [4] = 4
    [2] = 2
    [5] = 5
    [3] = 3
    [1,4,2] = 7
    [4,2,5] = 11
    [2,5,3] = 10
    [1,4,2,5,3] = 15
    我们将所有值求和得到 1 + 4 + 2 + 5 + 3 + 7 + 11 + 10 + 15 = 58

**示例 2：**

    输入：arr = [1,2]
    输出：3
    解释：总共只有 2 个长度为奇数的子数组，[1] 和 [2]。它们的和为 3 。

**示例 3：**

    输入：arr = [10,11,12]
    输出：66

**提示：**

- `1 <= arr.length <= 100`
- `1 <= arr[i] <= 1000`

**进阶：**

你可以设计一个 O(n) 时间复杂度的算法解决此问题吗？

## 题解

#### 方法一：暴力

最简单的方法是遍历数组 \\(\textit{arr}\\) 中的每个长度为奇数的子数组，计算这些子数组的和。由于只需要计算所有长度为奇数的子数组的和，不需要分别计算每个子数组的和，因此只需要维护一个变量 \\(\textit{sum}\\) 存储总和即可。

实现方面，令数组 \\(\textit{arr}\\) 的长度为 \\(n\\)，子数组的开始下标为 \\(\textit{start}\\)，长度为 \\(\textit{length}\\)，结束下标为 \\(\textit{end}\\)，则有 \\(0 \le \textit{start} \le \textit{end} < n\\)，\\(\textit{length} = \textit{end} - \textit{start} + 1\\) 为奇数。遍历符合上述条件的子数组，计算所有长度为奇数的子数组的和。

```Java [sol1-Java]
class Solution {
    public int sumOddLengthSubarrays(int[] arr) {
        int sum = 0;
        int n = arr.length;
        for (int start = 0; start < n; start++) {
            for (int length = 1; start + length <= n; length += 2) {
                int end = start + length - 1;
                for (int i = start; i <= end; i++) {
                    sum += arr[i];
                }
            }
        }
        return sum;
    }
}
```

**复杂度分析**

- 时间复杂度：\\(O(n^3)\\)，其中 \\(n\\) 是数组 \\(\textit{arr}\\) 的长度。长度为奇数的子数组的数量是 \\(O(n^2)\\)，对于每个子数组需要 \\(O(n)\\) 的时间计算子数组的和，因此总时间复杂度是 \\(O(n^3)\\)。

- 空间复杂度：\\(O(1)\\)。

#### 方法二：前缀和

方法一中，对于每个子数组需要使用 \\(O(n)\\) 的时间计算子数组的和。如果能将计算每个子数组的和的时间复杂度从 \\(O(n)\\) 降低到 \\(O(1)\\)，则能将总时间复杂度从 \\(O(n^3)\\) 降低到 \\(O(n^2)\\)。

为了在 \\(O(1)\\) 的时间内得到每个子数组的和，可以使用前缀和。创建长度为 \\(n + 1\\) 的前缀和数组 \\(\textit{prefixSums}\\)，其中 \\(\textit{prefixSums}[0] = 0\\)，当 \\(1 \le i \le n\\) 时，\\(\textit{prefixSums}[i]\\) 表示数组 \\(\textit{arr}\\) 从下标 \\(0\\) 到下标 \\(i - 1\\) 的元素和。

得到前缀和数组 \\(\textit{prefixSums}\\) 之后，对于 \\(0 \le \textit{start} \le \textit{end} < n\\)，数组 \\(\textit{arr}\\) 的下标范围 \\([\textit{start}, \textit{end}]\\) 的子数组的和为 \\(\textit{prefixSums}[\textit{end} + 1] - \textit{prefixSums}[\textit{start}]\\)，可以在 \\(O(1)\\) 的时间内得到每个子数组的和。

```Java [sol2-Java]
class Solution {
    public int sumOddLengthSubarrays(int[] arr) {
        int n = arr.length;
        int[] prefixSums = new int[n + 1];
        for (int i = 0; i < n; i++) {
            prefixSums[i + 1] = prefixSums[i] + arr[i];
        }
        int sum = 0;
        for (int start = 0; start < n; start++) {
            for (int length = 1; start + length <= n; length += 2) {
                int end = start + length - 1;
                sum += prefixSums[end + 1] - prefixSums[start];
            }
        }
        return sum;
    }
}
```

**复杂度分析**

- 时间复杂度：\\(O(n^2)\\)，其中 \\(n\\) 是数组 \\(\textit{arr}\\) 的长度。需要 \\(O(n)\\) 的时间计算前缀和数组 \\(\textit{prefixSums}\\)，长度为奇数的子数组的数量是 \\(O(n^2)\\)，对于每个子数组需要 \\(O(1)\\) 的时间计算子数组的和，因此总时间复杂度是 \\(O(n^2)\\)。

- 空间复杂度：\\(O(n)\\)，其中 \\(n\\) 是数组 \\(\textit{arr}\\) 的长度。需要创建长度为 \\(n + 1\\) 的前缀和数组 \\(\textit{prefixSums}\\)。

#### 方法三：数学

方法一和方法二都是通过计算每个子数组的和得到所有奇数长度子数组的和。可以换一种思路，数组中的每个元素都会在一个或多个奇数长度的子数组中出现，如果可以计算出每个元素在多少个长度为奇数的子数组中出现，即可得到所有奇数长度子数组的和。

对于元素 \\(\textit{arr}[i]\\)，记其左边和右边的元素个数分别为 \\(\textit{leftCount}\\) 和 \\(\textit{rightCount}\\)，则 \\(\textit{leftCount} = i\\)，\\(\textit{rightCount} = n - i - 1\\)。如果元素 \\(\textit{arr}[i]\\) 在一个长度为奇数的子数组中，则在该子数组中，元素 \\(\textit{arr}[i]\\) 的左边和右边的元素个数一定同为奇数或同为偶数（注意 \\(0\\) 也是偶数）。

- 当元素 \\(\textit{arr}[i]\\) 的左边和右边的元素个数同为奇数时，在区间 \\([0, \textit{leftCount}]\\) 范围内的奇数有 \\(\textit{leftOdd} = \Big\lfloor \dfrac{\textit{leftCount} + 1}{2} \Big\rfloor\\) 个，在区间 \\([0, \textit{rightCount}]\\) 范围内的奇数有 \\(\textit{rightOdd} = \Big\lfloor \dfrac{\textit{rightCount} + 1}{2} \Big\rfloor\\) 个，包含元素 \\(\textit{arr}[i]\\) 的子数组个数为 \\(\textit{leftOdd} \times \textit{rightOdd}\\)；

- 当元素 \\(\textit{arr}[i]\\) 的左边和右边的元素个数同为偶数时，在区间 \\([0, \textit{leftCount}]\\) 范围内的偶数有 \\(\textit{leftEven} = \Big\lfloor \dfrac{\textit{leftCount}}{2} \Big\rfloor + 1\\) 个，在区间 \\([0, \textit{rightCount}]\\) 范围内的偶数有 \\(\textit{rightEven} = \Big\lfloor \dfrac{\textit{rightCount}}{2} \Big\rfloor + 1\\) 个，包含元素 \\(\textit{arr}[i]\\) 的子数组个数为 \\(\textit{leftEven} \times \textit{rightEven}\\)。

根据上述分析可知，包含元素 \\(\textit{arr}[i]\\) 的奇数长度子数组个数为 \\(\textit{leftOdd} \times \textit{rightOdd} + \textit{leftEven} \times \textit{rightEven}\\)，因此元素 \\(\textit{arr}[i]\\) 对奇数长度子数组元素和的贡献为 \\(\textit{arr}[i] \times (\textit{leftOdd} \times \textit{rightOdd} + \textit{leftEven} \times \textit{rightEven})\\)。

```Java [sol3-Java]
class Solution {
    public int sumOddLengthSubarrays(int[] arr) {
        int sum = 0;
        int n = arr.length;
        for (int i = 0; i < n; i++) {
            int leftCount = i, rightCount = n - i - 1;
            int leftOdd = (leftCount + 1) / 2;
            int rightOdd = (rightCount + 1) / 2;
            int leftEven = leftCount / 2 + 1;
            int rightEven = rightCount / 2 + 1;
            sum += arr[i] * (leftOdd * rightOdd + leftEven * rightEven);
        }
        return sum;
    }
}
```

**复杂度分析**

- 时间复杂度：\\(O(n)\\)，其中 \\(n\\) 是数组 \\(\textit{arr}\\) 的长度。需要遍历数组 \\(\textit{arr}\\) 一次，对于每个元素，需要 \\(O(1)\\) 的时间计算该元素在奇数长度子数组的和中的贡献值，因此总时间复杂度是 \\(O(n)\\)。

- 空间复杂度：\\(O(1)\\)。
