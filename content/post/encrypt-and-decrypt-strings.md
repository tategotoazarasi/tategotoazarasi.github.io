---
title: "LeetCode 5302. Encrypt and Decrypt Strings"
date: 2022-04-03T17:10:49+08:00
draft: false
tags: ["leetcode", "困难", "字符串","哈希表"]
math: true
---

You are given a character array `keys` containing **unique** characters and a string array `values` containing strings of length 2. You are also given another string array `dictionary` that contains all permitted original strings after decryption. You
should implement a data structure that can encrypt or decrypt a **0-indexed** string.

A string is **encrypted** with the following process:

1. For each character `c` in the string, we find the index `i` satisfying `keys[i] == c` in `keys`.
2. Replace `c` with `values[i]` in the string.

A string is **decrypted** with the following process:

1. For each substring `s` of length 2 occurring at an even index in the string, we find an `i` such that `values[i] == s`. If there are multiple valid `i`, we choose **any** one of them. This means a string could have multiple possible strings it can decrypt to.
2. Replace `s` with `keys[i]` in the string.

Implement the `Encrypter` class:

- `Encrypter(char[] keys, String[] values, String[] dictionary)` Initializes the `Encrypter` class with `keys, values`, and `dictionary`.
- `String encrypt(String word1)` Encrypts `word1` with the encryption process described above and returns the encrypted string.
- `int decrypt(String word2)` Returns the number of possible strings `word2` could decrypt to that also appear in `dictionary`.

<!--more-->

**Example 1:**

**Input**
["Encrypter", "encrypt", "decrypt"]
[[['a', 'b', 'c', 'd'], ["ei", "zf", "ei", "am"], ["abcd", "acbd", "adbc", "badc", "dacb", "cadb", "cbda", "abad"]], ["abcd"], ["eizfeiam"]]
**Output**
[null, "eizfeiam", 2]

**Explanation**

```java
Encrypter encrypter = new Encrypter([['a', 'b', 'c', 'd'], ["ei", "zf", "ei", "am"], ["abcd", "acbd", "adbc", "badc", "dacb", "cadb", "cbda", "abad"]);
encrypter.encrypt("abcd"); // return "eizfeiam".
                           // 'a' maps to "ei", 'b' maps to "zf", 'c' maps to "ei", and 'd' maps to "am".
encrypter.decrypt("eizfeiam"); // return 2.
                              // "ei" can map to 'a' or 'c', "zf" maps to 'b', and "am" maps to 'd'.
                              // Thus, the possible strings after decryption are "abad", "cbad", "abcd", and "cbcd".
                              // 2 of those strings, "abad" and "abcd", appear in dictionary, so the answer is 2.
```

**Constraints:**

- `1 <= keys.length == values.length <= 26`
- `values[i].length == 2`
- `1 <= dictionary.length <= 100`
- `1 <= dictionary[i].length <= 100`
- All `keys[i]` and `dictionary[i]` are **unique**.
- `1 <= word1.length <= 2000`
- `1 <= word2.length <= 200`
- All `word1[i]` appear in `keys`.
- `word2.length` is even.
- `keys`, `values[i]`, `dictionary[i]`, `word1`, and `word2` only contain lowercase English letters.
- At most `200` calls will be made to `encrypt` and `decrypt` **in total**.
- For encryption, use hashmap to map each char of word1 to its value.
- For decryption, use trie to prune when necessary.

## Solution

对于解密操作，不妨逆向思考，即加密所有 dictionary 中的字符串。用哈希表记录每个加密后的字符串的出现次数。这样每次调用 decrypt 时，返回哈希表中 word2​ 的出现次数即可。

```cpp
class Encrypter {
    array<string, 26> mp;
    unordered_map<string, int> cnt;
public:
    Encrypter(vector<char> &keys, vector<string> &values, vector<string> &dictionary) {
        for (int i = 0; i < keys.size(); ++i)
            mp[keys[i] - 'a'] = values[i];
        for (auto &s : dictionary)
            ++cnt[encrypt(s)];
    }

    string encrypt(string word1) {
        string res;
        for (char ch : word1) {
            auto &s = mp[ch - 'a'];
            if (s == "") return "";
            res += s;
        }
        return res;
    }

    int decrypt(string word2) {
        return cnt.count(word2) ? cnt[word2] : 0;    // 防止把不在 cnt 中的字符串加进去
    }
};
```
