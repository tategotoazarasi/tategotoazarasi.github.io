---
title: "LeetCode 67. 二进制求和"
date: 2021-12-10T10:13:22+08:00
draft: false
tags: ["leetcode", "简单", "位运算", "数学", "字符串", "模拟"]
math: true
---

给你两个二进制字符串，返回它们的和（用二进制表示）。

输入为 **非空** 字符串且只包含数字  `1`  和  `0`。

<!--more-->

**示例  1:**

> **输入:** a = "11", b = "1"
> 
> **输出:** "100"

**示例  2:**

> **输入:** a = "1010", b = "1011"
> 
> **输出:** "10101"

**提示：**

- 每个字符串仅由字符 `'0'` 或 `'1'` 组成。
- \\(1 <= a.length, b.length <= 10^4\\)
- 字符串如果不是 `"0"` ，就都不含前导零。

```java
class Solution {
    public String addBinary(String a, String b) {
        int max_length = Math.max(a.length(), b.length());
        if (b.length() == max_length) {
            String temp = a;
            a = b;
            b = temp;
        }
        int[] ret = new int[max_length];
        boolean round = false;
        for (int i = max_length - 1; i >= 0; i--) {
            char a_char = a.charAt(i);
            int b_index = i - (a.length() - b.length());
            char b_char = b_index >= 0 ? b.charAt(b_index) : 0;
            int a_i = a_char == '1' ? 1 : 0;
            int b_i = b_char == '1' ? 1 : 0;
            int r_i = round ? 1 : 0;
            int sum = a_i + b_i + r_i;
            switch (sum) {
                case 0: {
                    ret[i] = 0;
                    round = false;
                    break;
                }
                case 1: {
                    ret[i] = 1;
                    round = false;
                    break;
                }
                case 2: {
                    ret[i] = 0;
                    round = true;
                    break;
                }
                case 3: {
                    ret[i] = 1;
                    round = true;
                    break;
                }
            }
        }
        StringBuilder sb = new StringBuilder();
        if (round) {
            sb.append('1');
        }
        for (int i = 0; i < max_length; i++) {
            if (ret[i] == 1) {
                sb.append('1');
            } else {
                sb.append('0');
            }
        }
        return sb.toString();
    }
}
```
