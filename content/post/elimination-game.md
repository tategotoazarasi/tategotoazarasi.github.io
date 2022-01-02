---
title: "LeetCode 390. 消除游戏"
date: 2022-01-02T17:04:49+08:00
draft: false
tags: ["leetcode", "中等", "数学"]
math: true
---

给定一个从 1 到 n 排序的整数列表。  
首先，从左到右，从第一个数字开始，每隔一个数字进行删除，直到列表的末尾。  
第二步，在剩下的数字中，从右到左，从倒数第一个数字开始，每隔一个数字进行删除，直到列表开头。  
我们不断重复这两步，从左到右和从右到左交替进行，直到只剩下一个数字。  
返回长度为 n 的列表中，最后剩下的数字。

<!--more-->

**示例：**

**输入:**
n = 9,
<u>1</u> 2 <u>3</u> 4 <u>5</u> 6 <u>7</u> 8 <u>9</u>
2 <u>4</u> 6 <u>8</u>
<u>2</u> 6
6

**输出:**
6

```cpp
class Solution {
    public:
        int lastRemaining(int n) {
            int num_amount = n;
            int loop_cnt = 0;
            int a0 = 1, d = 1;
            while (num_amount != 1) {
                // 奇数个数字
                if (num_amount % 2 == 1) {
                    a0 = a0 + d;
                }
                // 偶数个数字
                else if (num_amount % 2 == 0) {
                    bool left_to_right = (loop_cnt % 2 == 0);
                    if (left_to_right) {
                        a0 = a0 + d;
                    } else
                        a0 = a0;
                }
                loop_cnt++;
                d *= 2;
                num_amount /= 2;
            }
            return a0;
        }
};
```
