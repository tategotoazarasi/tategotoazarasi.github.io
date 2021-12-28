---
title: "LeetCode 472. 连接词"
date: 2021-12-28T15:56:48+08:00
draft: false
tags: ["leetcode", "困难", "深度优先搜索", "字典树", "数组", "字符串", "动态规划"]
math: true
---

给你一个 **不含重复** 单词的字符串数组 `words` ，请你找出并返回 `words` 中的所有 **连接词** 。

**连接词** 定义为：一个完全由给定数组中的至少两个较短单词组成的字符串。

<!--more-->

**示例 1：**

**输入：** words = ["cat","cats","catsdogcats","dog","dogcatsdog","hippopotamuses","rat","ratcatdogcat"]
**输出：** ["catsdogcats","dogcatsdog","ratcatdogcat"]
**解释：** "catsdogcats" 由 "cats", "dog" 和 "cats" 组成;
"dogcatsdog" 由 "dog", "cats" 和 "dog" 组成;
"ratcatdogcat" 由 "rat", "cat", "dog" 和 "cat" 组成。

**示例 2：**

**输入：** words = ["cat","dog","catdog"]
**输出：** ["catdog"]

**提示：**

- \\(1 <= words.length <= 10^4\\)
- `0 <= words[i].length <= 1000`
- `words[i]` 仅由小写字母组成
- \\(0 <= sum(words[i].length) <= 10^5\\)

```cpp
class Solution {
    public:
        static std::vector < std::string > findAllConcatenatedWordsInADict(std::vector < std::string > & );
};

class TrieNode {
    public:
        bool is_end;
    bool is_concatenated;
    char ch;
    TrieNode *nexts[26] = {nullptr, nullptr, nullptr, nullptr, nullptr, nullptr, nullptr, nullptr, nullptr, nullptr,
                           nullptr, nullptr, nullptr, nullptr, nullptr, nullptr, nullptr, nullptr, nullptr, nullptr,
                           nullptr, nullptr, nullptr, nullptr, nullptr, nullptr};


    explicit TrieNode(char);

    void insert(const std::string & str);

    bool dfs(TrieNode * ,
        const std::string & , int, bool);
};

vector < string > Solution::findAllConcatenatedWordsInADict(vector < string > & words) {
    sort(words.begin(), words.end(), [ & ](const string & a,
        const string & b) {
        return a.size() < b.size();
    });
    auto ans = vector < string > ();
    TrieNode node = TrieNode(0);
    for (const string & word: words) {
        if (word.length() == 0) {
            continue;
        }
        if (node.dfs( & node, word, 0, false)) {
            ans.push_back(word);
        } else {
            node.insert(word);
        }
    }
    return ans;
}

TrieNode::TrieNode(char ch) {
    this -> ch = ch;
    this -> is_concatenated = false;
    this -> is_end = false;
}

void TrieNode::insert(const string & str) {
    auto node = this -> nexts[str[0] - 'a'];
    if ((node) == nullptr) {
        node = new TrieNode(str[0]);
        this -> nexts[str[0] - 'a'] = node;
    }
    if (str.length() == 1) {
        node -> is_end = true;
        return;
    } else return node -> insert(str.substr(1));
}

bool TrieNode::dfs(TrieNode * root,
    const string & str, int start, bool flag) {
    if (this -> ch == 0) { //根节点
        auto node = this -> nexts[str[start] - 'a'];
        if (node == nullptr) {
            return false;
        }
        if (node -> dfs(root, str, start, flag)) {
            return true;
        }
        return false;
    } else { //非根节点
        //到一个单词结束处
        if (this -> is_end) {
            if (start == str.length() - 1) {
                return flag;
            }
            auto res = root -> dfs(root, str, start + 1, true);
            if (res) {
                return true;
            }
        }
        TrieNode * node = nullptr;
        if (str[start + 1] - 'a' >= 0) {
            node = this -> nexts[str[start + 1] - 'a'];
        }
        return node != nullptr && node -> dfs(root, str, start + 1, flag);
    }
}
```
