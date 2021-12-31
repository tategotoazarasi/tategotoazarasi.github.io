---
title: implement-strstr
tags: [Import-7704]
created: '2021-12-18T11:18:20.411Z'
modified: '2021-12-30T13:58:01.454Z'
---

---
title: "LeetCode 28. 实现 strStr()"
date: 2021-12-07T10:17:21+08:00
draft: false
tags: ["leetcode", "简单", "双指针", "字符串", "字符串匹配"]
math: true
---

实现  [strStr()](https://baike.baidu.com/item/strstr/811469)  函数。

给你两个字符串  `haystack` 和 `needle` ，请你在 `haystack` 字符串中找出 `needle` 字符串出现的第一个位置（下标从 0 开始）。如果不存在，则返回   `-1` 。

<!--more-->

**说明：**

当  `needle`  是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当  `needle`  是空字符串时我们应当返回 0 。这与 C 语言的  [strstr()](https://baike.baidu.com/item/strstr/811469)  以及 Java 的  [indexOf()](<https://docs.oracle.com/javase/7/docs/api/java/lang/String.html#indexOf(java.lang.String)>)  定义相符。

**示例 1：**

> **输入：** haystack = "hello", needle = "ll"
> 
> **输出：** 2

**示例 2：**

> **输入：** haystack = "aaaaa", needle = "bba"
> 
> **输出：** -1

**示例 3：**

> **输入：** haystack = "", needle = ""
> 
> **输出：** 0

**提示：**

- \\(0 <= haystack.length, needle.length <= 5 \* 10^4\\)
- `haystack` 和 `needle` 仅由小写英文字符组成

```java
class Solution {
    public int strStr(String haystack, String needle) {
        if (needle.isEmpty() || needle.isBlank()) {
            return 0;
        }
        for (int i = 0; i < haystack.length(); i++) {
            if(haystack.startsWith(needle, i)){
                return i;
            }
        }
        return -1;
    }
}
```
