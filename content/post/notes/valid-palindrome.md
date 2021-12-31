---
title: valid-palindrome
tags: [Import-7704]
created: '2021-12-21T01:07:06.382Z'
modified: '2021-12-30T13:58:05.077Z'
---

---
title: "LeetCode 125. 验证回文串"
date: 2021-12-21T09:07:06+08:00
draft: false
tags: ["leetcode", "简单", "双指针", "字符串"]
math: true
---

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

<!--more-->

**说明：** 本题中，我们将空字符串定义为有效的回文串。

**示例 1:**

> **输入:** "A man, a plan, a canal: Panama"
> 
> **输出:** true
> 
> **解释：** "amanaplanacanalpanama" 是回文串

**示例 2:**

> **输入:** "race a car"
> 
> **输出:** false
> 
> **解释：** "raceacar" 不是回文串

**提示：**

- \\(1 <= s.length <= 2 \* 105\\)
- 字符串 `s` 由 ASCII 字符组成

```java
class Solution {
    public boolean isPalindrome(String s) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (Character.isDigit(c) || Character.isAlphabetic(c)) {
                sb.append(Character.toLowerCase(c));
            }
        }
        s = sb.toString();
        for (int i = 0, j = s.length() - 1; i < j; i++, j--) {
            if (s.charAt(i) != s.charAt(j)) {
                return false;
            }
        }
        return true;
    }
}
```
