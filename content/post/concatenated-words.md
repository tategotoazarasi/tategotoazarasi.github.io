---
title: "LeetCode 472. 连接词"
date: 2021-12-28T15:56:48+08:00
draft: false
tags:
  ["leetcode", "困难", "深度优先搜索", "字典树", "数组", "字符串", "动态规划"]
math: true
---

给你一个 **不含重复** 单词的字符串数组 `words` ，请你找出并返回 `words` 中的所有 **连接词** 。

**连接词** 定义为：一个完全由给定数组中的至少两个较短单词组成的字符串。

<!--more-->

**示例 1：**

> **输入：** words = ["cat","cats","catsdogcats","dog","dogcatsdog","hippopotamuses","rat","ratcatdogcat"]
> 
> **输出：** ["catsdogcats","dogcatsdog","ratcatdogcat"]
> 
> **解释：** "catsdogcats" 由 "cats", "dog" 和 "cats" 组成;
> 
> "dogcatsdog" 由 "dog", "cats" 和 "dog" 组成;
> 
> "ratcatdogcat" 由 "rat", "cat", "dog" 和 "cat" 组成。

**示例 2：**

> **输入：** words = ["cat","dog","catdog"]
> 
> **输出：** ["catdog"]

**提示：**

- \\(1 <= words.length <= 10^4\\)
- `0 <= words[i].length <= 1000`
- `words[i]` 仅由小写字母组成
- \\(0 <= sum(words[i].length) <= 10^5\\)

## 题解

判断一个单词是不是连接词，需要判断这个单词是否完全由至少两个给定数组中的更短的非空单词（可以重复）组成。判断更短的单词是否在给定数组中，可以使用字典树实现。

为了方便处理，首先将数组 words 按照字符串的长度递增的顺序排序，排序后可以确保当遍历到任意单词时，比该单词短的单词一定都已经遍历过，因此可以根据已经遍历过的全部单词判断当前单词是不是连接词。

在将数组 words 排序之后，遍历数组，跳过空字符串，对于每个非空单词，判断该单词是不是连接词，如果是连接词则将该单词加入结果数组，如果不是连接词则将该单词加入字典树。

判断一个单词是不是连接词的做法是在字典树中深度优先搜索。从该单词的第一个字符（即下标 0 处的字符）开始，在字典树中依次搜索每个字符对应的结点，可能有以下几种情况：

- 如果一个字符对应的结点是单词的结尾，则找到了一个更短的单词，从该字符的后一个字符开始搜索下一个更短的单词；

- 如果一个字符对应的结点在字典树中不存在，则当前的搜索结果失败，回到上一个单词的结尾继续搜索。

如果找到一个更短的单词且这个更短的单词的最后一个字符是当前单词的最后一个字符，则当前单词是连接词。由于数组 words 中没有重复的单词，因此在判断一个单词是不是连接词时，该单词一定没有加入字典树，由此可以确保判断连接词的条件成立。

**由于一个连接词由多个更短的非空单词组成，如果存在一个较长的连接词的组成部分之一是一个较短的连接词，则一定可以将这个较短的连接词换成多个更短的非空单词，因此不需要将连接词加入字典树。**

**复杂度分析**

- 时间复杂度：\\(O(nlogn+∑0≤i<n​li2​)\\)，其中 n 是数组 words 的长度，li​ 是单词 words[i] 的长度。对数组 words 按照字符串的长度递增的顺序排序需要 O(nlogn) 的时间，最坏情况下判断单词 words[i] 是不是连接词的时间复杂度是 O(li2​)。

- 空间复杂度：\\(O(∑0≤i<n​li​×∣S∣)\\)，其中 n 是数组 words 的长度，li​ 是单词 words[i] 的长度，S 是字符集，这道题中 S 为全部小写英语字母，∣S∣=26。空间复杂度主要取决于字典树，最坏情况下需要将所有的单词加入字典树。

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
