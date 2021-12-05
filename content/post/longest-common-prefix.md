---
title: "LeetCode 14. 最长公共前缀"
date: 2021-12-05T17:39:18+08:00
draft: false
tags: ["leetcode", "简单", "字符串"]
math: true
---

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串  `""`。

<!--more-->

**示例 1：**

> **输入：** strs = ["flower","flow","flight"]
>
> **输出：** "fl"

**示例 2：**

> **输入：** strs = ["dog","racecar","car"]
>
> **输出：** ""
>
> **解释：** 输入不存在公共前缀。

**提示：**

- `1 <= strs.length <= 200`
- `0 <= strs[i].length <= 200`
- `strs[i]` 仅由小写英文字母组成

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if (strs.length == 1) {
            return strs[0];
        }
        StringBuilder sb = new StringBuilder();
        if (strs[0].length() == 0) {
            return "";
        }
        for (short i = 0; ; i++) {
            if (i >= strs[0].length()) {
                return sb.toString();
            }
            char ch = strs[0].charAt(i);
            for (short j = 1; j < strs.length; j++) {
                if (i >= strs[j].length()) {
                    return sb.toString();
                }
                if (strs[j].charAt(i) != ch) {
                    return sb.toString();
                }
            }
            sb.append(ch);
        }
    }
}
```
