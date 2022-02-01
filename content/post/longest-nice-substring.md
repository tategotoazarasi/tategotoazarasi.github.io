---
title: "LeetCode 1763. 最长的美好子字符串"
date: 2022-02-01T19:52:49+08:00
draft: false
math: true
tags: ["leetcode", "简单", "位运算", "哈希表", "字符串", "滑动窗口"]
---

当一个字符串 `s`  包含的每一种字母的大写和小写形式 **同时**  出现在 `s`  中，就称这个字符串  `s`  是 **美好** 字符串。比方说，`"abABB"`  是美好字符串，因为  `'A'` 和  `'a'`  同时出现了，且  `'B'` 和  `'b'`  也同时出现了。然而，`"abA"`  不是美好字符串因为  `'b'`  出现了，而  `'B'`  没有出现。

给你一个字符串  `s` ，请你返回  `s`  最长的  **美好子字符串** 。如果有多个答案，请你返回  **最早**  出现的一个。如果不存在美好子字符串，请你返回一个空字符串。

<!--more-->

**示例 1：**

> **输入：** s = "YazaAay"
>
> **输出：** "aAa"
>
> **解释：** "aAa" 是一个美好字符串，因为这个子串中仅含一种字母，其小写形式 'a' 和大写形式 'A' 也同时出现了。
> "aAa" 是最长的美好子字符串。

**示例 2：**

> **输入：** s = "Bb"
>
> **输出：** "Bb"
>
> **解释：** "Bb" 是美好字符串，因为 'B' 和 'b' 都出现了。整个字符串也是原字符串的子字符串。

**示例 3：**

> **输入：** s = "c"
>
> **输出：** ""
>
> **解释：** 没有美好子字符串。

**示例 4：**

> **输入：** s = "dDzeE"
>
> **输出：** "dD"
>
> **解释：** "dD" 和 "eE" 都是最长美好子字符串。
> 由于有多个美好子字符串，返回 "dD" ，因为它出现得最早。

**提示：**

- `1 <= s.length <= 100`
- `s`  只包含大写和小写英文字母。
- 穷举每个子串检查是否是美好字符串。

## 题解

由于字符串的长度比较小，因此可以枚举所有可能的子字符串，然后检测该字符串是否为美好的字符串，并得到长度最长的美好字符串。

- 题目关于美好字符串的定义为: 字符串中的每个字母的大写和小写形式同时出现在该字符串中。检测时，由于英文字母 ‘a’−‘z’ 最多只有 26 个, 因此可以利用二进制位来进行标记，lower 标记字符中出现过小写英文字母，upper 标记字符中出现过大写英文字母。如果满足 lower=upper ，我们则认为字符串中所有的字符都满足大小写形式同时出现，则认定该字符串为美好字符串。

- 题目要求如果有多个答案，返回在字符串中最早出现的那个。此时，只需要首先检测从以字符串索引 0 为起始的子字符串。

```cpp
class Solution {
public:
    pair<int, int> dfs(string s, int start, int end) {
        if(start == end) {
            return {start, 0};
        }
        int lower     = 0;
        int upper     = 0;
        int max_start = 0;
        int max_len   = 0;
        for(int i = start; i <= end; i++) {
            char ch = s[i];
            if(islower(ch)) {
                lower |= 1 << (ch - 'a');
            } else {//isupper
                upper |= 1 << (ch - 'A');
            }
        }
        if(lower == upper) {//是美好字符串
            return {start, end - start + 1};
        }
        //不是美好字符串
        int not_nice = lower ^ upper;//无法构成美好字符串的字符
        int i        = start;
        while(i <= end) {
            if(((not_nice >> (tolower(s[i]) - 'a')) & 1) == 1) {//在not_nice中
                i++;
                continue;
            }
            int j = i + 1;
            while(j <= end && (((not_nice >> (tolower(s[j]) - 'a')) & 1) != 1)) {
                j++;
            }
            auto [next_start, next_len] = dfs(s, i, j - 1);
            if(max_len < next_len) {
                max_len   = next_len;
                max_start = next_start;
            }
            i = j;
        }
        return {max_start, max_len};
    }

    string longestNiceSubstring(string s) {
        auto [max_start, max_len] = dfs(s, 0, s.length() - 1);
        if(max_len == 0) {
            return "";
        }
        return s.substr(max_start, max_len);
    }
};
```
