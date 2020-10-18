---
title: LeetCode 2019 秋季编程大赛总结
key: leetcode-autumn-contest-2019
tags:
- Original
- Algorithm
toc: true
---

[2019 力扣杯全国秋季编程大赛](https://leetcode-cn.com/contest/season/2019-fall/) 算是我大学以来第一次参加的算法编程比赛。由于并没有接触过算法竞赛方面的知识，总体感觉是五道题难度跨越特别大，前三题花了不到半个小时，后面的时间几乎都用在最后一题上。

<!--more-->

## 题解

### 1. 猜数字

> 小A 和 小B 在玩猜数字。小B 每次从 1, 2, 3 中随机选择一个，小A 每次也从 1, 2, 3 中选择一个猜。他们一共进行三次这个游戏，请返回 小A 猜对了几次？
>
> 输入的`guess`数组为 小A 每次的猜测，`answer`数组为 小B 每次的选择。`guess`和`answer`的长度都等于3。

送分题，代码如下：

```java
class Solution {
    public int game(int[] guess, int[] answer) {
        int res = 0;
        for (int i = 0; i < 3; i++)
            if (guess[i] == answer[i])
                res++;
        return res;
    }
}
```

### 2. 分式化简

> 化简如下分式：
>
> $$
> a_0+\frac{1}{a_1+\frac{1}{a_2+\frac{1}{\cdots}}}
> $$
>
> 输入的`cont`代表连分数的系数（`cont[0]`代表上式的`a0`，以此类推）。返回一个长度为2的数组`[n, m]`，使得连分数的值等于`n / m`，且`n, m`最大公约数为1。

模拟题，从最里面的分式开始计算，题目中说明了`cont[i] >= 0`，因此不用考虑负数情况（当时在比赛时没有注意这一点，因此为考虑负数多花了些时间），最后分子分母同时除以最大公因数。

```java
class Solution {
    private int gcd(int a, int b) {
        return b == 0 ? a : gcd(b, a % b);
    }
    
    public int[] fraction(int[] cont) {
        int up = 1, down = cont[cont.length - 1];
        for (int i = cont.length - 2; i >= 0; i--) {
            int a = cont[i];
            up += a * down;
            
            int tmp = up;
            up = down;
            down = tmp;
        }
        int gcd = gcd(up, down);
        return new int[]{down / gcd, up / gcd};
    }
}
```


### 3. 机器人大冒险

> 力扣团队买了一个可编程机器人，机器人初始位置在原点(0, 0)。小伙伴事先给机器人输入一串指令command，机器人就会无限循环这条指令的步骤进行移动。指令有两种：
>
> U: 向y轴正方向移动一格
>
> R: 向x轴正方向移动一格。
>
> 不幸的是，在 xy 平面上还有一些障碍物，他们的坐标用obstacles表示。机器人一旦碰到障碍物就会被损毁。
>
> 给定终点坐标(x, y)，返回机器人能否完好地到达终点。如果能，返回true；否则返回false。

模拟题，机器人若能走到终点，那么需要的指令数一定恰好等于 x+y，模拟 x+y 次后判断是否处于终点位置，如果中途遇到障碍或者走出 (x, y) 范围则提前结束。

一开始将所有障碍坐标保存在 Map 中，用于加速判断当前坐标是否处在障碍上。

比赛中的一次罚时是因为最后直接 return true 没有判断是否位于终点... 做简单题也一定要细心不能大意。

```java
class Solution {
    public boolean robot(String command, int[][] obstacles, int x, int y) {
        Map<Integer, Set<Integer>> obsMap = new HashMap<>();
        for (int i = 0; i < obstacles.length; i++) {
            int x1 = obstacles[i][0];
            int y1 = obstacles[i][1];
            if (!obsMap.containsKey(x1)) {
                obsMap.put(x1, new HashSet<>());
            }
            obsMap.get(x1).add(y1);
        }
        int curX = 0, curY = 0;
        for (int i = 1; i <= x + y; i++) {
            char step = command.charAt((i - 1) % command.length());
            if (step == 'U')
                curY += 1;
            else
                curX += 1;
            if (obsMap.containsKey(curX) && obsMap.get(curX).contains(curY))
                return false;
            if (curX > x || curY > y)
                return false;
        }
        return curX == x && curY == y;
    }
}
```

### 4. 覆盖

> 你有一块棋盘，棋盘上有一些格子已经坏掉了。你还有无穷块大小为`1 * 2`的多米诺骨牌，你想把这些骨牌**不重叠**地覆盖在**完好**的格子上，请找出你最多能在棋盘上放多少块骨牌？这些骨牌可以横着或者竖着放。
>
> 输入：`n, m`代表棋盘的大小；`broken`是一个`b * 2`的二维数组，其中每个元素代表棋盘上每一个坏掉的格子的位置。
>
> 输出：一个整数，代表最多能在棋盘上放的骨牌数。
> 
> 限制：
>
> 1 <= n <= 8
>
> 1 <= m <= 8
>
> 0 <= b <= n * m

这题在比赛时并没有什么思路，后来看题解可以用状态压缩DP或者二分图匹配来做，二分图匹配的方法很巧妙，之后有空会把代码补上。


### 5. 发 LeetCoin

题目描述很长，可以看[此链接](https://leetcode-cn.com/problems/coin-bonus/)。

大意是说有一棵树（不一定是二叉树），树上的每个节点代表一个员工，树的层次就代表上下属关系。现在可以进行三种操作：

1. 给某个节点（员工）发一定数量硬币。
2. 给某棵子树的所有节点都发一定数量硬币。
3. 查询某棵子树所有节点的硬币之和。

程序需要尽可能高效地完成这三种操作。



在执行具体操作前，我建立的数据结构包括：

- `all` 数组：来记录每个节点的硬币数，初始均为0
- `father` 数组：记录每个节点的父节点编号（类似并查集中的数组）
- `sons` Map：记录每个节点的所有直接子节点集合
- `totalSons` 数组：记录以每个节点为根的子树一共有多少节点。这里会用 `countSons()` 函数递归计算。

当执行操作1时，步骤如下：

1. 将 `all` 中对应节点硬币数增加；
2. 在 `all` 数组中依次更新当前节点父节点的硬币数，直到到达根节点，这里会用到 `father` 数组。

当执行操作2时，步骤如下：

1. 使用 `update()` 函数递归更新当前节点子树中所有节点硬币数，在更新时，使用 `totalSons` 数组可以直接计算出当前节点硬币数应该增加多少。
2. 与操作1类似，依次更新父节点硬币数，不过这里更新时增加的值（`addFatherCoin`）与1中有所不同。

具体实现代码如下：

```java
class Solution {
    private int countSons(int k, Map<Integer, Set<Integer>> sons) {
        if (!sons.containsKey(k) || sons.get(k) == null || sons.get(k).isEmpty())
            return 0;
        Set<Integer> sonSet = sons.get(k);
        int res = sonSet.size();
        for (Integer son : sonSet) {
            res += countSons(son, sons);
        }
        return res;
    }

    private void update(int num, int coin, int[] totalSons, int[] all, Map<Integer, Set<Integer>> sons) {
        all[num] = (all[num] + coin) % 1000000007;
        if (totalSons[num] == 0)
            return;
        all[num] = (all[num] + totalSons[num] * coin % 1000000007);
        for (int son : sons.get(num)) {
            update(son, coin, totalSons, all, sons);
        }
    }


    public int[] bonus(int n, int[][] leadership, int[][] operations) {
        int[] all = new int[n + 1];
        int[] father = new int[n + 1];
        Map<Integer, Set<Integer>> sons = new HashMap<>();
        father[1] = -1;
        for (int i = 0; i < leadership.length; i++) {
            int a = leadership[i][0];
            int b = leadership[i][1];
            father[b] = a;

            if (!sons.containsKey(a))
                sons.put(a, new HashSet<>());
            sons.get(a).add(b);
        }
        int[] totalSons = new int[n + 1];
        for (int i = 0; i < n; i++) {
            totalSons[i + 1] = countSons(i + 1, sons);
        }

        List<Integer> res = new ArrayList<>();
        for (int i = 0; i < operations.length; i++) {
            int op = operations[i][0];
            int num = operations[i][1];
            if (op == 1) {
                int coin = operations[i][2];
                all[num] = (all[num] + coin) % 1000000007;
                int cur = father[num];
                while (cur != -1) {
                    all[cur] = (all[cur] + coin) % 1000000007;
                    cur = father[cur];
                }
            } else if (op == 2) {
                int coin = operations[i][2];
                update(num, coin, totalSons, all, sons);
                int addFatherCoin = coin * (totalSons[num] + 1) % 1000000007;
                int cur = father[num];
                while (cur != -1) {
                    all[cur] = (all[cur] + addFatherCoin) % 1000000007;
                    cur = father[cur];
                }
            } else {
                res.add(all[num]);
            }
        }
        int[] resArr = new int[res.size()];
        for (int i = 0; i < res.size(); i++)
            resArr[i] = res.get(i);
        return resArr;
    }
}
```


## 后续

可能由于这次是 LeetCode 第一次举办的全国大赛所以大佬们并没有关注的原因，我最后竟然正好是第50名（能拿到奖品的最后一名 :joy:） 。

> 第 1 名选手获得《计算机程序设计艺术》卷 1 - 卷 4A ；第 2-5 名选手，优先随机挑选以下图书，第 6-50 再做选择，数量有限，先选先得，图书包含：《程序员面试金典 · 第六版》、《算法 4》、《挑战程序设计 2》、《Python 深度学习》、《C++ Primer中文版（第5版）》、《Python机器学习手册：从数据预处理到深度学习》、《Effective C++：改善程序与设计的55个具体做法（第三版）》、《Python编程之美：最佳实践指南》、《Code：隐匿在计算机软硬件背后的语言（英文版）》。

![](/assets/images/posts/leetcode-autumn-contest-2019/prize-email.png){:width="600px"}

虽然只剩下最后两本书可以选了，不过还是挺开心的 :joy:。选了一本《程序员面试金典》，很快就收到了书，厚厚的一大本，之后找工作的时候有东西看了~

