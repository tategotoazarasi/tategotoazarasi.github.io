---
title: "LeetCode 1036. 逃离大迷宫"
date: 2022-01-11T16:08:56+08:00
draft: false
tags: ["leetcode", "困难", "深度优先搜索", "广度优先搜索", "数组", "哈希表"]
math: true
---

在一个 \\(10^6 \times 10^6\\) 的网格中，每个网格上方格的坐标为 `(x, y)` 。

现在从源方格 \\(source = [s_x, s_y]\\) 开始出发，意图赶往目标方格 \\(target = [t_x, t_y]\\) 。数组 `blocked` 是封锁的方格列表，其中每个 \\(blocked[i] = [x_i, y_i]\\) 表示坐标为 \\((x_i, y_i)\\) 的方格是禁止通行的。

每次移动，都可以走到网格中在四个方向上相邻的方格，只要该方格 **不** 在给出的封锁列表 `blocked` 上。同时，不允许走出网格。

只有在可以通过一系列的移动从源方格 `source` 到达目标方格 `target` 时才返回 `true`。否则，返回 `false`。

<!--more-->

**示例 1：**

> **输入：** blocked = [[0,1],[1,0]], source = [0,0], target = [0,2]
>
> **输出：** false
>
> **解释：**
> 从源方格无法到达目标方格，因为我们无法在网格中移动。
> 无法向北或者向东移动是因为方格禁止通行。
> 无法向南或者向西移动是因为不能走出网格。

**示例 2：**

> **输入：** blocked = [], source = [0,0], target = [999999,999999]
>
> **输出：** true
>
> **解释：**
> 因为没有方格被封锁，所以一定可以到达目标方格。

**提示：**

- `0 <= blocked.length <= 200`
- `blocked[i].length == 2`
- \\(0 <= x_i, y_i < 10^6\\)
- `source.length == target.length == 2`
- \\(0 <= s_x, s_y, t_x, t_y < 10^6\\)
- `source != target`
- 题目数据保证 `source` 和 `target` 不在封锁列表内
- 如果我们被卡住了，要么有一个围绕源头的循环，要么有一个围绕目标的循环。
- 如果有一个围绕源头的循环，那么它最多可以有多少个方块？

## 题解

判断能否从 source 走到 target，最简单且直观的方法就是从 source 开始进行广度优先搜索，如果搜索的过程中经过了 target，则说明可以到达。

然而本题中给定的网格规模是 \\(10^6×10^6\\) 的，常规的广度优先搜索会达到 \\(O(10^6×10^6)=O(10^{12})\\) 的时间复杂度，远远超出了时间限制。因此我们必须进行优化。

注意到障碍的个数不超过 200 个，这说明网格中大部分的位置都不是障碍，只有极少一部分的位置是障碍。分析从 source 无法走到 target 的情况，无非就是以下两种：

- source 被障碍完全包围，并且 target 不在包围圈中；

- target 被障碍完全包围，并且 source 不在包围圈中。

在「障碍的个数不超过 200 个前提下」，我们可以猜测「包围圈」的大小也不会很大。也就是说，如果障碍的个数为 n 个，那么我们希望得到「包围圈」中包含的非障碍位置的数量的上界，假设其为 limit，那么：

> 如果我们从 source 开始进行广度优先搜索，搜索到的非障碍位置的数量严格大于 limit，那么说明 source 一定不在任何包围圈中。同时，如果从 target 开始进行广度优先搜索也是如此，那么同样说明 target 一定也不在任何包围圈中。此时我们就可以断定，一定能从 source 走到 target。

因此我们的目标就是正确估计「包围圈」的上界。当障碍的个数固定为 n 个时（我们只考虑 n≥2 的情况，如果 n=0,1，那么无法形成「包围圈」，任意两个位置都互相可达），要想使得非障碍位置的数量最大，这些障碍一定是靠着网格的一个角进行包围的，因为这样可以利用到网格的两条边。

不失一般性，我们假设靠着的是网格的左上角，那么可以证明，最优的包围方法一定是选择：

(0,n−1),(1,n−2),(2,n−3),⋯,(n−3,2),(n−2,1),(n−1,0)

这 n 个位置，此时「包围圈」组成一个等腰直角三角形，如下图所示。

![fig1](https://tategotoazarasi.github.io/leetcode1036_1.png)

其中包含的非障碍位置的数量为：

\\(\frac{n(n-1)}{2}\\)

最优性可以通过构造法证明。可以发现，上面给出的包围方法保证了每一行和每一列不会有超过 1 个障碍。对于其它的包围方法，不失一般性，设某一行有 2 个或以上的障碍，此时有两种情况：

- 如果这两个障碍不八连通，那么其中一个障碍的设置是无意义的，因为它不和「包围圈」的外界连通，放置该障碍只会减少「包围圈」中非障碍位置的数量；

- 如果这两个障碍八连通，如下图所示，我们可以将从第一个障碍开始，到离第二个障碍最近的并且行坐标小于等于它为止的所有障碍的行坐标增加 1，这样就可以构造出一个包含非障碍位置数量更多的「包围圈」。

![fig2](https://tategotoazarasi.github.io/leetcode1036_2.png)

因此，最优的包围方法一定保证每一行和每一列不会有超过 1 个障碍。由于我们需要包围住左上角，因此列坐标的必须取 0,1,2,⋯,n−1 这 n 个位置（如果取 ≥n，那么该障碍就无法与「包围圈」连通，因为它至少还需要借助 n 个障碍来连接到左边界）。同时，每一行只有一个障碍，因此每个障碍右侧的所有位置是不可能被包围的，只有左侧的位置才可能被包围。而左侧位置的总数为：

\\(0 + 1 + 2 + \cdots + (n-1) = \frac{n(n-1)}{2}\\)

因此最多包含非障碍位置的数量为 \\(\frac{n(n-1)}{2}\\)。

**算法**

通过上述的证明，我们就可以设计出如下的算法：

- 我们从 source 开始进行广度优先搜索。如果经过了不超过 \\(\frac{n(n-1)}{2}\\) 个非障碍位置就已经结束搜索，说明 source 在「包围圈」中。但如果我们在过程中经过了 target，那么说明它们是可达的，否则一定不可达。

- 我们再从 target 开始进行广度优先搜索。如果经过了不超过 \\(\frac{n(n-1)}{2}\\) 个非障碍位置就已经结束搜索，说明 target 在「包围圈」中。否则说明 source 和 target 均不在「包围圈」中，此时一定可达。

搜索的过程中需要借助哈希表来标记每个位置是否已经搜索过。

```cpp
struct point {
    unsigned int x;
    unsigned int y;
    unsigned int distance;
    const point *target;
    point(): x(0), y(0), distance(0), target(nullptr) {};
    point(unsigned int x, unsigned int y, int distance, point *target): x(x), y(y), distance(distance), target(target) {};
    bool operator<(const point &p) const {
        return ((this->distance + (abs(int(this->x - target->x)) + abs(int(this->y - target->y)))) < (p.distance + (abs(int(p.x - target->x)) + abs(int(p.y - target->y)))));
    }
    bool operator==(const point &p) const {
        return this->x == p.x && this->y == p.y;
    }
};

struct point_hash {
    size_t operator()(const point &p) const {
        return p.x * 1000000 + p.y;
    }
};

class Solution {
public:
    /// \brief 在迷宫中以source为起点搜索
    /// \param block_set 阻塞点集
    /// \param source 起点
    /// \param target 终点
    /// \param limit 搜索步数上限
    /// \return 0表示无法抵达 1表示可以抵达 2表示达到搜索步数上限
    unsigned int search(unordered_set<point, point_hash> &block_set, point *source, point *target, unsigned int limit) {
        priority_queue<point> pq = priority_queue<point>();
        pq.push(point(source->x, source->y, 0, target));
        int count = 0;
        while(!pq.empty()) {
            if(count > limit || pq.size() > limit) {
                return 2;//不包围
            }
            count++;
            auto p = pq.top();
            pq.pop();
            point nexts[] = {point(p.x + 1, p.y, p.distance + 1, target), point(p.x - 1, p.y, p.distance + 1, target), point(p.x, p.y + 1, p.distance + 1, target), point(p.x, p.y - 1, p.distance + 1, target)};
            for(auto next: nexts) {
                if(0 <= next.x && next.x < 1000000 && 0 <= next.y && next.y < 1000000 && block_set.count(next) == 0) {
                    if(next.x == target->x && next.y == target->y) {
                        return 1;//连通
                    }
                    pq.push(next);
                    block_set.insert(next);
                }
            }
        }
        return 0;//不连通
    }

    bool isEscapePossible(vector<vector<int>>& blocked, vector<int>& source, vector<int>& target) {
        int limit = blocked.size() *(blocked.size()-1) / 2;
        if(blocked.empty()) {
            return true;
        }
        auto *p_source                                      = new point(source[0], source[1], 0, nullptr);
        auto *p_target                                      = new point(target[0], target[1], 0, nullptr);
        unordered_set<point, point_hash> blocked_set_source = unordered_set<point, point_hash>();
        unordered_set<point, point_hash> blocked_set_target = unordered_set<point, point_hash>();
        for(auto block: blocked) {
            blocked_set_source.insert(point(block[0], block[1], 0, nullptr));
            blocked_set_target.insert(point(block[0], block[1], 0, nullptr));
        }
        unsigned int source_status = search(blocked_set_source, p_source, p_target, limit);
        if(source_status == 0) {
            return false;
        } else if(source_status == 1) {
            return true;
        } else {
            unsigned int target_status = search(blocked_set_target, p_target, p_source, limit);
            if(target_status == 0) {
                return false;
            } else {
                return true;
            }
        }
    }
};
```
