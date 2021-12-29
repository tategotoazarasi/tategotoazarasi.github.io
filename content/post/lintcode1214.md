---
title: "LintCode 1214. 许可证密钥格式"
date: 2021-12-29T10:33:57+08:00
draft: false
tags: ["lintcode", "简单", "模拟法", "字符串", "谷歌"]
math: true
---

给定一个用字符串 S 表示的许可证，其中仅仅包含了数字、字母和短横线。字符串被 N 个短横线“-”切分为了 N+1 组。

给定一个数字 K，要求重新整理字符串的格式，使得除了第一组之外的每个组正好 K 个字符，第一组长度可以比 K 小，但也至少要包含一个字符。此外，对于两个组之间必须要插入一个短横线，所有的小写字母都要转换为大写字母。

<!--more-->

> 字符串 S 的长度不会超过 12000，而且 K 是一个正整数。  
> 字符串 S 仅仅包含大小写字母、数字和短横线“-”。  
> 字符串 S 非空。

## 样例

```
输入: S = "5F3Z-2e-9-w", K = 4
输出: "5F3Z-2E9W"
解释: 字符串S切分为两个部分, 每个部分有4个字符。
注意原串中两个额外的横线是多余的，可以删掉。
```

```
输入: S = "2-5g-3-J", K = 2
输出: "2-5G-3J"
解释: 字符串S切分为了三部分, 每个部分有两个字符，除了第一个部分，因为如原题所述，第一部分长度只有1。
```

```cpp
class Solution {
    public:
        /**
         * @param S: a string
         * @param K: a integer
         * @return: return a string
         */
        string licenseKeyFormatting(string & S, int K) {
            ostringstream S2 = ostringstream();
            ostringstream output = ostringstream();
            int len = 0;
            for (char c: S) {
                if (isalnum(c)) {
                    len++;
                    if (isdigit(c) || isupper(c)) {
                        S2 << c;
                    } else if (islower(c)) {
                        S2 << static_cast < char > (toupper(c));
                    }
                }
            }
            string str = S2.str();
            int first = len % K;
            int average = len / K;
            if (first == 0) {
                first = K;
            }
            string str1 = str.substr(0, first);
            string str2 = str.substr(first);
            for (char c: str1) {
                output << c;
            }
            int count = 0;
            for (char c: str2) {
                if (count == 0) {
                    output << '-';
                }
                output << c;
                count++;
                count %= K;
            }
            return output.str();
        }
};
```
