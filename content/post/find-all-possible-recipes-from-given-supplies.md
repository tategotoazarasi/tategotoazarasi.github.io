---
title: "LeetCode 5947. 从给定原材料中找到所有可以做出的菜"
date: 2021-12-26T09:22:40+08:00
draft: false
tags: ["leetcode", "中等"]
math: true
---

你有 `n`  道不同菜的信息。给你一个字符串数组  `recipes`  和一个二维字符串数组  `ingredients` 。第  `i`  道菜的名字为  `recipes[i]` ，如果你有它  **所有**  的原材料  `ingredients[i]` ，那么你可以  **做出**  这道菜。一道菜的原材料可能是  **另一道**  菜，也就是说  `ingredients[i]`  可能包含  `recipes`  中另一个字符串。

同时给你一个字符串数组  `supplies` ，它包含你初始时拥有的所有原材料，每一种原材料你都有无限多。

请你返回你可以做出的所有菜。你可以以 **任意顺序**  返回它们。

注意两道菜在它们的原材料中可能互相包含。

<!--more-->

**示例 1：**

**输入：** recipes = ["bread"], ingredients = [["yeast","flour"]], supplies = ["yeast","flour","corn"]
**输出：** ["bread"]
**解释：**
我们可以做出 "bread" ，因为我们有原材料 "yeast" 和 "flour" 。

**示例 2：**

**输入：** recipes = ["bread","sandwich"], ingredients = [["yeast","flour"],["bread","meat"]], supplies = ["yeast","flour","meat"]
**输出：** ["bread","sandwich"]
**解释：**
我们可以做出 "bread" ，因为我们有原材料 "yeast" 和 "flour" 。
我们可以做出 "sandwich" ，因为我们有原材料 "meat" 且可以做出原材料 "bread" 。

**示例 3：**

**输入：** recipes = ["bread","sandwich","burger"], ingredients = [["yeast","flour"],["bread","meat"],["sandwich","meat","bread"]], supplies = ["yeast","flour","meat"]
**输出：** ["bread","sandwich","burger"]
**解释：**
我们可以做出 "bread" ，因为我们有原材料 "yeast" 和 "flour" 。
我们可以做出 "sandwich" ，因为我们有原材料 "meat" 且可以做出原材料 "bread" 。
我们可以做出 "burger" ，因为我们有原材料 "meat" 且可以做出原材料 "bread" 和 "sandwich" 。

**示例 4：**

**输入：** recipes = ["bread"], ingredients = [["yeast","flour"]], supplies = ["yeast"]
**输出：** []
**解释：**
我们没法做出任何菜，因为我们只有原材料 "yeast" 。

**提示：**

- `n == recipes.length == ingredients.length`
- `1 <= n <= 100`
- `1 <= ingredients[i].length, supplies.length <= 100`
- `1 <= recipes[i].length, ingredients[i][j].length, supplies[k].length <= 10`
- `recipes[i], ingredients[i][j]`  和  `supplies[k]`  只包含小写英文字母。
- 所有  `recipes` 和  `supplies`  中的值互不相同。
- `ingredients[i]`  中的字符串互不相同。
- 我们能否用一个数据结构来快速查询我们是否有某种成分？
- 一旦我们验证了我们可以制作一个配方，我们就可以把它添加到我们的配料数据结构中。然后我们可以检查我们是否可以因此而制作更多的食谱。

```java
import java.util.HashMap;
import java.util.LinkedList;
import java.util.List;
import java.util.Vector;

class Solution {
    class Node {
        public String name;
        public Vector<Node> next;
        public boolean flag;
        public Node(String name){
            this.name = name;
            this.flag = false;
            this.next = new Vector<>();
        }
    }

    public List<String> findAllRecipes(String[] recipes, List<List<String>> ingredients, String[] supplies) {
        HashMap<String, Node> nodes = new HashMap<>();
        for (int i = 0; i < recipes.length; i++) {
            if (!nodes.containsKey(recipes[i])) {
                Node node = new Node(recipes[i]);
                nodes.put(recipes[i], node);
            }
            Node node = nodes.get(recipes[i]);
            for (String ingredient : ingredients.get(i)) {
                if (!nodes.containsKey(ingredient)) {
                    Node node2 = new Node(ingredient);
                    nodes.put(ingredient, node2);
                    node.next.add(node2);
                } else {
                    Node node2 = nodes.get(ingredient);
                    node.next.add(node2);
                }
            }
        }
        for (String supply : supplies) {
            if (nodes.containsKey(supply)) {
                nodes.get(supply).flag = true;
            }
        }
        boolean flag = true;
        while (flag) {
            flag = false;
            for (String recipe : recipes) {
                boolean f2 = true;
                Node curnode = nodes.get(recipe);
                for (Node node : curnode.next) {
                    if (!node.flag) {
                        f2 = false;
                        break;
                    }
                }
                if (f2 && !curnode.flag) {
                    curnode.flag = true;
                    flag = true;
                }
            }
        }
        List<String> list = new LinkedList<>();
        for (String recipe : recipes){
            Node curnode = nodes.get(recipe);
            if(curnode.flag){
                list.add(curnode.name);
            }
        }
        return list;
    }
}
```
