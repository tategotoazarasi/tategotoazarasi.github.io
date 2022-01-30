---
title: "LeetCode 5995. 字符串分组"
date: 2022-01-30T16:22:21+08:00
draft: false
tags: ["leetcode", "困难", "并查集", "状态压缩", "位运算"]
math: true
---

给你一个下标从  **0** 开始的字符串数组  `words` 。每个字符串都只包含 **小写英文字母** 。`words`  中任意一个子串中，每个字母都至多只出现一次。

如果通过以下操作之一，我们可以从 `s1`  的字母集合得到 `s2`  的字母集合，那么我们称这两个字符串为 **关联的** ：

- 往  `s1`  的字母集合中添加一个字母。
- 从  `s1`  的字母集合中删去一个字母。
- 将 `s1`  中的一个字母替换成另外任意一个字母（也可以替换为这个字母本身）。

数组  `words`  可以分为一个或者多个无交集的 **组** 。一个字符串与一个组如果满足以下 **任一**  条件，它就属于这个组：

- 它与组内 **至少**  一个其他字符串关联。
- 它是这个组中 **唯一**  的字符串。

注意，你需要确保分好组后，一个组内的任一字符串与其他组的字符串都不关联。可以证明在这个条件下，分组方案是唯一的。

请你返回一个长度为 `2`  的数组  `ans` ：

- `ans[0]`  是  `words`  分组后的  **总组数** 。
- `ans[1]`  是字符串数目最多的组所包含的字符串数目。

<!--more-->

**示例 1：**

> **输入：** words = ["a","b","ab","cde"]
>
> **输出：** [2,3]
>
> **解释：**

- words[0] 可以得到 words[1] （将 'a' 替换为 'b'）和 words[2] （添加 'b'）。所以 words[0] 与 words[1] 和 words[2] 关联。
- words[1] 可以得到 words[0] （将 'b' 替换为 'a'）和 words[2] （添加 'a'）。所以 words[1] 与 words[0] 和 words[2] 关联。
- words[2] 可以得到 words[0] （删去 'b'）和 words[1] （删去 'a'）。所以 words[2] 与 words[0] 和 words[1] 关联。
- words[3] 与 words 中其他字符串都不关联。
  所以，words 可以分成 2 个组 ["a","b","ab"] 和 ["cde"] 。最大的组大小为 3 。

**示例 2：**

> **输入：** words = ["a","ab","abc"]
>
> **输出：** [1,3]
>
> **解释：**

- words[0] 与 words[1] 关联。
- words[1] 与 words[0] 和 words[2] 关联。
- words[2] 与 words[1] 关联。
  由于所有字符串与其他字符串都关联，所以它们全部在同一个组内。
  所以最大的组大小为 3 。

**提示：**

- \\(1 <= words.length <= 2 \* 10^4\\)
- `1 <= words[i].length <= 26`
- `words[i]`  只包含小写英文字母。
- `words[i]` 中每个字母最多只出现一次。

```cpp
class Solution {
    int groups;
    unsigned int max_size;
    unordered_map<unsigned int, unsigned int> parent;
    unordered_map<unsigned int, unsigned int> rank;
    unordered_map<unsigned int, unsigned int> size;

public:
    Solution()
        : groups(0), max_size(1), parent(unordered_map<unsigned int, unsigned int>()), rank(unordered_map<unsigned int, unsigned int>()) {};
    vector<int> groupStrings(vector<string> &words) {
        auto ans  = vector<int>();
        auto nums = vector<unsigned int>();
        for(const auto &word: words) {
            auto num = compress(word);
            nums.push_back(num);
            insert(num);
        }
        for(const auto num: nums) {
            for(int i = 0; i < 26; i++) {
                auto next = num ^ 1 << i;//添加或删除字符 i
                if(parent.count(next)==1) {
                    to_union(num, next);
                }
                if((num >> i & 1) == 1) {
                    //存在字符i
                    for(int j = 0; j < 26; j++) {
                        if((num >> j & 1) == 0) {
                            //不存在字符j
                            auto next = num ^ 1 << i | 1 << j;
                            if(parent.count(next)==1) {
                                to_union(num, num ^ 1 << i | 1 << j);// 替换字符 i 为 j
                            }
                        }
                    }
                }
            }
        }
        ans.push_back(groups);
        ans.push_back(max_size);
        return ans;
    }
    static unsigned int compress(const string &word) {
        unsigned int ans = 0;
        for(const char ch: word) {
            ans |= 1 << (ch - 'a');
        }
        return ans;
    }
    void insert(unsigned int num) {
        parent.insert(pair(num, num));
        rank.insert(pair(num, 0));
        if(size.count(num)==0) {
            size.insert(pair(num, 1));
            groups++;
        } else {
            size[num]++;
            max_size = size[num];
        }
    }
    unsigned int find(unsigned int x) {
        return x == parent[x] ? x : parent[x] = find(parent[x]);
    }
    void to_union(unsigned int x1, unsigned int x2) {
        const int f1 = find(x1);
        const int f2 = find(x2);
        if(f1 == f2) {
            return;
        }
        groups--;
        if(rank[f1] > rank[f2]) {
            parent[f2] = f1;
            size[f1] += size[f2];
            max_size = max(max_size, size[f1]);
        } else {
            parent[f1] = f2;
            size[f2] += size[f1];
            max_size = max(max_size, size[f2]);
            if(rank[f1] == rank[f2]) {
                ++rank[f2];
            }
        }
    }
};
```
