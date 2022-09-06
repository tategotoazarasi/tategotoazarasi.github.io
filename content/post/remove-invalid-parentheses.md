---
title: "LeetCode 301. 删除无效的括号"
date: 2022-09-06T13:11:21+08:00
draft: false
tags: ["leetcode", "困难", "广度优先搜索","字符串","回溯"]
math: true
---

给你一个由若干括号和字母组成的字符串 `s`，删除最小数量的无效括号，使得输入的字符串有效。

返回所有可能的结果。答案可以按 **任意顺序** 返回。

<!--more-->

**示例 1：**

    输入：s = "()())()"
    输出：["(())()","()()()"]

**示例 2：**

    输入：s = "(a)())()"
    输出：["(a())()","(a)()()"]

**示例 3：**

    输入：s = ")("
    输出：[""]

**提示：**

- `1 <= s.length <= 25`
- `s` 由小写英文字母以及括号 `'('` 和 `')'` 组成
- `s` 中至多含 `20` 个括号

## 题解

#### 背景知识

1. **有效的「括号」**：题目输入的字符串由一系列「左括号」和「右括号」组成，但是有一些额外的括号，使得括号不能正确配对。对于括号配对规则如果还不太清楚的读者，可以先完成问题「[20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)」。

2. **可以一次遍历计算出多余的「左括号」和「右括号」**：
根据括号匹配规则和根据求解「[22. 括号生成](https://leetcode-cn.com/problems/generate-parentheses/)」的经验，我们知道：如果当前遍历到的「左括号」的数目严格小于「右括号」的数目则表达式无效。因此，我们可以遍历一次输入字符串，统计「左括号」和「右括号」出现的次数。

- 当遍历到「左括号」的时候：
  - 「左括号」数量加 \\(1\\)。

- 当遍历到「右括号」的时候：
  - 如果此时「左括号」的数量不为 \\(0\\)，因为「右括号」可以与之前遍历到的「左括号」匹配，此时「左括号」出现的次数 \\(-1\\)；
  - 如果此时「左括号」的数量为 \\(0\\)，「右括号」数量加 \\(1\\)。

通过这样的计数规则，得到的「左括号」和「右括号」的数量就是各自最少应该删除的数量。

#### 方法一：回溯 + 剪枝

**思路与算法**

题目让我们删除括号使得剩下的括号匹配，要求我们删除最少的括号数，并且要求得到所有的结果。我们可以使用回溯算法，尝试遍历所有可能的去掉非法括号的方案。

首先我们利用括号匹配的规则求出该字符串 \\(s\\)中最少需要去掉的左括号的数目 \\(\textit{lremove}\\)和右括号的数目 \\(\textit{rremove}\\)，然后我们尝试在原字符串 \\(s\\)中去掉 \\(\textit{lremove}\\)个左括号和 \\(\textit{rremove}\\)个右括号，然后检测剩余的字符串是否合法匹配，如果合法匹配则我们则认为该字符串为可能的结果，我们利用回溯算法来尝试搜索所有可能的去除括号的方案。

在进行回溯时可以利用以下的剪枝技巧来增加搜索的效率：

- 我们从字符串中每去掉一个括号，则更新 \\(\textit{lremove}\\)或者 \\(\textit{rremove}\\)，当我们发现剩余未尝试的字符串的长度小于 \\(\textit{lremove} + \textit{rremove}\\)时，则停止本次搜索。
- 当 \\(\textit{lremove}\\)和 \\(\textit{rremove}\\)同时为 \\(0\\)时，则我们检测当前的字符串是否合法匹配，如果合法匹配则我们将其记录下来。

由于记录的字符串可能存在重复，因此需要对重复的结果进行去重，去重的办法有如下两种：

- 利用哈希表对最终生成的字符串去重。
- 我们在每次进行搜索时，如果遇到连续相同的括号我们只需要搜索一次即可，比如当前遇到的字符串为 \\(\texttt{"(((())"}\\)，去掉前四个左括号中的任意一个，生成的字符串是一样的，均为 \\(\texttt{"((())"}\\)，因此我们在尝试搜索时，只需去掉一个左括号进行下一轮搜索，不需要将前四个左括号都尝试一遍。

**代码**

```cpp
class Solution {
public:
    vector<string> res;
    vector<string> removeInvalidParentheses(string s) {
        int lremove = 0;
        int rremove = 0;

        for (char c : s) {
            if (c == '(') {
                lremove++;
            } else if (c == ')') {
                if (lremove == 0) {
                    rremove++;
                } else {
                    lremove--;
                }
            }
        }
        helper(s, 0, lremove, rremove);
        return res;
    }

    void helper(string str, int start, int lremove, int rremove) {
        if (lremove == 0 && rremove == 0) {
            if (isValid(str)) {
                res.push_back(str);
            }
            return;
        }
        for (int i = start; i < str.size(); i++) {
            if (i != start && str[i] == str[i - 1]) {
                continue;
            }
            // 如果剩余的字符无法满足去掉的数量要求，直接返回
            if (lremove + rremove > str.size() - i) {
                return;
            } 
            // 尝试去掉一个左括号
            if (lremove > 0 && str[i] == '(') {
                helper(str.substr(0, i) + str.substr(i + 1), i, lremove - 1, rremove);
            }
            // 尝试去掉一个右括号
            if (rremove > 0 && str[i] == ')') {
                helper(str.substr(0, i) + str.substr(i + 1), i, lremove, rremove - 1);
            }
        }
    }

    inline bool isValid(const string & str) {
        int cnt = 0;

        for (int i = 0; i < str.size(); i++) {
            if (str[i] == '(') {
                cnt++;
            } else if (str[i] == ')') {
                cnt--;
                if (cnt < 0) {
                    return false;
                }
            }
        }

        return cnt == 0;
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(n \times 2^n)\\)，其中 \\(n\\)为字符串的长度。考虑到一个字符串最多可能有 \\(2^n\\)个子序列，每个子序列可能需要进行一次合法性检测，因此时间复杂度为 \\(O(n \times 2^n)\\)。

- 空间复杂度：\\(O(n^2)\\)，其中 \\(n\\)为字符串的长度。返回结果不计入空间复杂度，考虑到递归调用栈的深度，并且每次递归调用时需要复制字符串一次，因此空间复杂度为 \\(O(n^2)\\)。

#### 方法二：广度优先搜索

**思路与算法**

注意到题目中要求最少删除，这样的描述正是广度优先搜索算法应用的场景，并且题目也要求我们输出所有的结果。我们在进行广度优先搜索时每一轮删除字符串中的 \\(1\\)个括号，直到出现合法匹配的字符串为止，此时进行轮转的次数即为最少需要删除括号的个数。

我们进行广度优先搜索时，每次保存上一轮搜索的结果，然后对上一轮已经保存的结果中的每一个字符串尝试所有可能的删除一个括号的方法，然后将保存的结果进行下一轮搜索。在保存结果时，我们可以利用哈希表对上一轮生成的结果去重，从而提高效率。

**代码**

```cpp
class Solution {
public:
    bool isValid(string str) {
        int count = 0;

        for (char c : str) {
            if (c == '(') {
                count++;
            } else if (c == ')') {
                count--;
                if (count < 0) {
                    return false;
                }
            }
        }

        return count == 0;
    }

    vector<string> removeInvalidParentheses(string s) {
        vector<string> ans;
        unordered_set<string> currSet;

        currSet.insert(s);
        while (true) {
            for (auto & str : currSet) {
                if (isValid(str))
                    ans.emplace_back(str);
            }
            if (ans.size() > 0) {
                return ans;
            }
            unordered_set<string> nextSet;
            for (auto & str : currSet) {
                for (int i = 0; i < str.size(); i++) {
                    if (i > 0 && str[i] == str[i - 1]) {
                        continue;
                    }
                    if (str[i] == '(' || str[i] == ')') {
                        nextSet.insert(str.substr(0, i) + str.substr(i + 1, str.size()));
                    }
                }
            }
            currSet = nextSet;
        }
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(n \times 2^n)\\)，其中 \\(n\\)为字符串的长度。考虑到一个字符串最多可能有 \\(2^n\\)个子序列，因此时间复杂度为 \\(O(n \times 2^n)\\)。

- 空间复杂度：\\(O(n \times C_n^\frac{n}{2})\\)，其中 \\(n\\)为字符串的长度。我们在进行第 \\(i\\)轮迭代时，会从原始字符串中删除 \\(i\\)个括号，因此第 \\(i\\)轮迭代产生的字符串最多有 \\(C_n^i\\)个，当 \\(i = \frac{n}{2}\\)时组合数最大，此时迭代生成的字符串个数最多，因此空间复杂度为 \\(O(n \times C_n^\frac{n}{2})\\)。

#### 方法三：枚举状态子集

**思路与算法**

首先我们利用括号匹配的规则求出该字符串 \\(s\\)中最少需要去掉的左括号的数目 \\(\textit{lremove}\\)和右括号的数目 \\(\textit{rremove}\\)，然后我们利用状态子集求出字符串 \\(s\\)中所有的左括号去掉 \\(\textit{lremove}\\)的左括号的子集，和所有的右括号去掉 \\(\textit{rremove}\\)个右括号的子集，依次枚举这两种子集的组合，检测组合后的字符串是否合法匹配，如果字符串合法则记录，最后我们利用哈希表对结果进行去重。

**代码**

```cpp
class Solution {
public:
    bool checkValid(const string & str, int lmask, vector<int> & left, int rmask, vector<int> & right) {
        int pos1 = 0;
        int pos2 = 0;
        int cnt = 0;

        for (int i = 0; i < str.size(); i++) {
            if (pos1 < left.size() && i == left[pos1]) {
                if (!(lmask & (1 << pos1))) {
                    cnt++;
                }
                pos1++;
            } else if (pos2 < right.size() && i == right[pos2]) {
                if (!(rmask & (1 << pos2))) {
                    cnt--;
                    if (cnt < 0) {
                        return false;
                    }
                }
                pos2++;
            }
        }

        return cnt == 0;
    }

    string recoverStr(const string & str, int lmask, vector<int> & left, int rmask, vector<int> & right){
        string ans;
        int pos1 = 0;
        int pos2 = 0;

        for (int i = 0; i < str.size(); i++) {
            if (pos1 < left.size() && i == left[pos1]) {
                if (!(lmask & (1 << pos1))){
                    ans.push_back(str[i]);
                }
                pos1++;
            } else if (pos2 < right.size() && i == right[pos2]) {
                if (!(rmask & (1 << pos2))) {
                    ans.push_back(str[i]);
                }
                pos2++;
            } else {
                ans.push_back(str[i]);
            }
        }

        return ans;
    }

    vector<string> removeInvalidParentheses(string s) {
        int lremove = 0;
        int rremove = 0;
        vector<int> left;
        vector<int> right;
        vector<string> ans;
        unordered_set<string> cnt;

        for (int i = 0; i < s.size(); i++) {
            if (s[i] == '(') {
                left.push_back(i);
                lremove++;
            } else if (s[i] == ')') {
                right.push_back(i);
                if (lremove == 0) {
                    rremove++;
                } else {
                    lremove--;
                }
            }
        }

        int m = left.size();
        int n = right.size();
        vector<int> maskArr1;
        vector<int> maskArr2;
        for (int i = 0; i < (1 << m); i++) {
            if (__builtin_popcount(i) != lremove) {
                continue;
            }
            maskArr1.push_back(i);
        }
        for (int j = 0; j < (1 << n); j++) {
            if (__builtin_popcount(j) != rremove) {
                continue;
            }
            maskArr2.push_back(j);
        }
        for (auto mask1 : maskArr1) {
            for (auto mask2 : maskArr2) {
                if (checkValid(s, mask1, left, mask2, right)) {
                    cnt.insert(recoverStr(s, mask1, left, mask2, right));
                }
            }
        }
        for (auto v : cnt) {
            ans.emplace_back(v);
        }

        return ans;
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(n \times 2^n)\\)，其中 \\(n\\)为字符串的长度。考虑到一个字符串最多可能有 \\(2^n\\)个子序列，每个子序列可能需要进行一次合法性检测，因此时间复杂度为 \\(O(n \times 2^n)\\)。

- 空间复杂度：\\(O(n \times C_n^m)\\)，其中 \\(n\\)为字符串的长度，\\(m\\)为字符串中非法括号的数目。空间复杂度主要为集合 \\(\texttt{cnt}\\)占用的空间。
