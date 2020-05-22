---
title: 动态规划的相关细节探讨
subtitle: DAG 上的动态规划
date: '2018-12-28'
slug: more-dynamic-programming
categories:
  - Algorithms
tags:
  - dynamic programming
toc: true
---

注：本文为刘汝佳的《算法竞赛入门经典（第2版）》的读书笔记，大部分内容均摘自此书。

为了对动态规划有更加深入的了解，本文对动态规划中的诸多细节问题进行了探究。

在应用动态规划时，除了常规的直接从确定状态与状态转移方程出发的思路之外，还有另外一种思路，即将动态规划和 DAG 联系起来，进而进行状态与转移方程的确定。下面主要对第二种思路进行阐述。

# 1. 状态和状态转移方程
状态和状态转移方程是动态规划的核心，二者一起完整地描述了具体的算法。（注：此处的“算法”指的是特定问题的具体实现算法。动态规划是一种用途很广的问题求解方法，它本身并不是一个特定的算法，而是一种思想，一种手段。）

动态规划具体的实现方式分为 2 种：

1. Tabulation: Bottom-up 的迭代算法（递推）
2. Memoization: Top-down 的递归算法（记忆化搜索）

# 2. DAG 上的动态规划
有向无环图上的动态规划是学习动态规划的基础。很多问题都可以转化为 DAG 上的最长路径、最短路径或路径计数问题。

## 2.1 最长路径及其字典序（不限起点和终点）
**嵌套矩形问题**：有 *n* 个矩形，每个矩形可以用两个整数 *a*、*b* 描述，表示它的长和宽。矩形 `$X \left ( a, b \right )$` 可以嵌套在矩形 `$Y \left ( c, d \right )$` 中，当且仅当 `$a < c$`，`$b < d$`，或者 `$b < c$`，`$a < d$` （相当于把矩形 *X* 旋转 90 度）。你的任务是选出尽量多的矩形排成一行，使得除了最后一个之外，每一个矩形都可以嵌套在下一个矩形内。如果有多解，矩形编号的字典序应尽量小。

**建立 DAG 模型**：矩形之间的“可嵌套”关系是一个典型的二元关系，二元关系可以用图来建模。如果矩形 *X* 可以嵌套在矩形 *Y* 里，就从 *X* 到 *Y* 连一条有向边。这个有向图是无环的，因为一个矩形无法直接或间接地嵌套在自己内部。换句话说，它是一个 DAG。这样，所要求的便是 DAG 上的最长路径。

注意：对于不确定路径起点和终点的问题，“最短路径”显然是空（如果不允许空，不管怎样都是平凡的）。

**求解最长路径**：如何求 DAG 中不固定起点和终点的最长路径呢？回到状态与状态方程这两个核心要素上来，设 `$d \left ( i \right )$` 表示从结点 *i* 出发的最长路径的长度，应该如何写状态转移方程呢？第一步只能走到它的相邻点，因此：

`$$d \left ( i \right ) = {\rm max} \left \{ d \left ( j \right ) + 1 | \left ( i, j \right ) \in E \right \}$$`

其中，*E* 为边集。最终答案是所有 `$d \left ( i \right )$` 中的最大值。具体算法可以使用 Bottom-up 的 Tabulation 迭代法或者 Top-down 的 Memoization 递归法。不管怎样，都需要先把图建立起来，假设用 Adjacent Matrix 保存在矩阵 *G* 中（在编写主程序之前需测试和调试程序，以确保建图过程正确无误）。接下来编写记忆化搜索程序（调用前需初始化 d 数组的所有值为 0，即为 MIN = x - 1 = 1 - 1 = 0，具体参考第三节）：

注意，此题永远有解，并且解的初始值为 1。

```cpp
// Initialize all d[i] to 0. (MIN = x - 1 = 1 - 1 = 0)

int dp(int i) {
    int & ans = d[i];
    if (ans != 0) {
        return ans;
    }

    ans = 1; // The solution always exists. And the initial solution is 1.
    for (int j = 1; j <= n; j++) {
        if (G[i][j]) {
            ans = max(ans, dp(j) + 1);
        }
    }

    return ans;
}
```

小技巧：为表项 d[i] 声明一个引用 ans。这样，任何对 ans 的读写实际上都是在对 d[i] 进行。当 d[i] 换成 d[i][j][k][l][m][n] 这样很长的名字时，该技巧的优势就会很明显。

提示：在记忆化搜索中，可以为正在处理的表项声明一个引用，简化对它的读写操作。

**重建字典序最小的最长路径**：原题还有一个要求，即如果有多个最优解，矩形编号的字典序应最小。解决方法为：将所有 *d* 值计算出来以后，选择最大的 d[i] 所对应的 *i*。如果有多个 *i*，则选择最小的 *i*，这样才能保证字典序最小。接下来可以选择 `$d \left ( i \right ) = d \left ( j \right ) + 1$` 且 `$\left ( i, j \right ) \in E$` 的任何一个 *j*。为了让方案的字典序最小，应选择其中最小的 *j*。程序如下：

```cpp
void print_ans(int i) {
    printf("%d ", i);
    for (int j = 1; i <= n; j++) {
        if (G[i][j] && d[i] == d[j] + 1) {
            print_ans(j);
            break;
        }
    }
}
```

提示：当用递推法计算出各个指标之后，可以用与记忆化搜索完全相同的方式打印方案。

提示：根据各个状态的指标值可以依次确定各个最优决策，从而构造出完整方案。由于决策是从起点开始依次确定的，所以很容易打印出字典序最小的最优方案。

**重建所有最长路径**：注意，当找到一个满足 d[i] == d[j] + 1 的节点 *j* 后就应立即递归打印从 *j* 开始的路径，并在递归返回后退出循环。如果要打印所有方案，只把 break 语句删除是不够的（这是因为，单纯将 break 删除后，上述代码将按照 pre-order 遍历打印所有节点）。正确的方法是记录路径上的所有点，如使用 stack，然后在递归结束时一次性输出整条路径。

**状态定义的重要性**：有趣的是，如果把状态定义成“`$d \left ( i \right )$` 表示以结点 *i* 为终点的最长路径长度”，也能顺利求出最优值，却难以打印出字典序最小的方案。原因在于字典序是由起点出发，依次向后比较的。

## 2.2 固定起点和终点的最长路径和最短路径
**硬币问题/零钱问题**：有 *n* 种硬币，面值分别为 `$V_{1}, V_{2}, ..., V_{n}$`，每种都有无限多。给定非负整数 *S*，可以选用多少个硬币，使得面值之和恰好为 *S*？输出硬币数目的最小值和最大值。其中 `$1 \leq n \leq 100$`，`$0 \leq S \leq 10000$`，`$1 \leq V_{i} \leq S$`。

**建立 DAG 模型**：此问题尽管看上去和嵌套矩形问题很不一样，但问题的本质也是 DAG 上的路径问题。将每种面值看作一个点，表示“还需要凑足的面值”，则初始状态为 *S*，目标状态为 0。若当前状态为 *i*，每使用一个硬币 *j*，状态便转移到 `$i - V_{j}$`。

这个模型与嵌套矩形问题类似，但也有一些明显的不同：嵌套矩形问题并没有确定路径的起点和终点（可以把任意的矩形放在第一个和最后一个），而本题的起点必须为 *S*，终点必须为 0；点固定之后“最短路径”才是有意义的。在嵌套矩形问题中，最短路径显然是空（如果不允许空，就是单个矩形，不管怎样都是平凡的），而本题的最短路径却不容易确定。

**求解最长路径和最短路径**：最长路径（所用硬币最多）和最短路径（所用硬币最少）的求法是类似的，下面只考虑最长路径。由于终点固定，`$d \left ( i \right )$` 的确切含义是“从结点 *i* 出发到结点 0 的最长路径长度”。下面是求最长路径的代码：

注：下面代码实际功能为返回总面值**不超过** *S* 的零钱的最大张数，因此不存在无解的问题，并且解的默认初始值为 0。对于零钱总面值必须等于 *S* 的情况，在第三节进行讨论。

```cpp
int dp(int S) {
    // Initialize all d[i] to -1. (MIN = x - 1 = 0 - 1 = -1)

    int & ans = d[S];
    if (ans != -1) {
        return ans;
    }

    ans = 0; // The solution always exists. And the initial solution is 0.
    for (int i = 1; i <= n; i++) {
        if (S >= V[i]) {
            ans = max(ans, dp(S - V[i]) + 1);
        }
    }

    return ans;
}
```

**矩形嵌套问题和零钱问题的一个区别**：嵌套矩形中关心的是最长路径中的结点个数（因此解的最小值为 x = 1），因此可以初始化为 MIN = x - 1 = 0。而零钱问题中关心的是最长路径中的边的个数（因此解的最小值为 x = 0），因此可以初始化为 MIN = x - 1 = -1。由此导致，二者在实现上有 2 个区别：（1）“没有算过”的表示，即 MIN 值的不同；（2）路径计算时的初始值，即 x 的不同。

提示：当程序中需要用到特殊值时，应确保该值在正常情况下不会被取到。这不仅意味着特殊值不能有“正常的理解方式”，而且也不能在正常运算中“意外得到”。


## 3. 对 DP table 的标志值与初始值的处理方式
由上述零钱问题的分析可以看出，在 DP table 的建立过程中，有 3 个关键的标志值和一个初始值：

1. 第一个为表示该表项“还没算过”的标志值，该值应不在有效值区间内；
2. 第二个为表示该表项“无法到达”的标志值，该值应不在有效值区间内；
3. 第三个为查询“无法到达”表项时的返回值，该值应不在有效值区间内，通常为 -1。注意，查询“不能到达”的表项应该返回的值并不意味着一定是该表项应该存储的值，如下面要讲到的 Bottom-up 的递推方法中“无法到达”的表项内最终存的值就不是 -1。另外，并不是所有的题目均存在“无法到达”的无解情况，如上述矩形嵌套问题永远有解。
4. DP table 的初始值，该值应不在有效值区间内，其是一个同上边三个标志值不同的概念，上边三个为功能值，而初始值为实际初始化的值，初始值可能起到表征“还没算过”的功能，同时也可能起到表征“无法到达”的功能，需根据具体的算法需求及设计来确定。（此处所说的初始值为 DP table 创建时的初始值，而非前述代码中恒有解问题的初始解。）
5. DP table 的初始值是有一定要求的，因为其要参与到后续的状态转移方程中去。初始值的一般选取原则为：

> 假设 DP table 的有效取值范围为 [x, y]，(一般 x 为 0 或 1，)，则 DP table 的初始值可以选取 [x, y] 之外的值。但其要受到一定条件的限制：
>
> 因为初始值要参与到状态转移方程中，因此其取值要不影响正常的状态更新。因此在最短路径问题中，由于状态转移方程用到 min 函数，因此初始值要选取一个比 y 大的数，通常选为 MAX = y + 1；而在最长路径问题中，由于状态转移方程用到 max 函数，因此初始值要选取一个比 x 小的数，通常选作 MIN = x - 1。

提示：在记忆化搜索（递归）中，如果用特殊值表示“还没算过”，则必须将其和其他特殊值（如无解）区别开。

下面对比分析 Bottom-up 递推和记忆化搜索（memoization）对这 4 个值的不同的处理方式。

注：其中的代码可以参考 [LeetCode 332. Coin Change](https://leetcode.com/problems/coin-change/).

### 3.1 递推法对标志值与初始值的处理方法
递推与递归的本质区别在于递推是按照 DP table 实际建立的顺序来更新表项的，因此在更新一个新的表项时，其需要访问的所依赖的表项均为已经算过的，并不需要一个标志值来表示“还没算过”，因此初始值可以选择比 y 大 （最短路径）或者比 x 小（最长路径）的任意一个值。如果 -1 在初始值的选择范围内，那么选择 -1 作为初始值无疑是最恰当的，其既在功能上充当了“无法到达”的标志值又可以直接作为查询的返回值。如果 -1 不在初始值的选择范围内，则按照上节中的常规 MAX = y + 1 或者 MIN = x - 1 的原则进行选择即可。

最短路径问题：

```cpp
int coinChange(vector<int> & coins, int amount) {
    // Dynamic Programming
    // O(n * amount) time and O(amount) space
    int MAX = amount + 1; // MAX means no valid solution
    vector<int> dp(amount + 1, MAX);
    // Base case, amount = 0
    dp[0] = 0;
    // iterately fill in dp table
    // Note, we can use one-dimensional array because dp[k] only depends on dp[m] where m < k.
    for (int i = 1; i <= amount; i++) {
        for (int j = 0; j < coins.size(); j++) {
            if (coins[j] <= i) {
                dp[i] = min(dp[i], dp[i - coins[j]] + 1);
            }
        }
    }
    return dp[amount] == MAX ? -1 : dp[amount];
}
```

最长路径问题：

```cpp
int coinChange(vector<int> & coins, int amount) {
    // Dynamic Programming
    // O(n * amount) time and O(amount) space
    int MIN = -1; // MIN means no valid solution
    vector<int> dp(amount + 1, MIN);
    // Base case, amount = 0
    dp[0] = 0;
    // iterately fill in dp table
    // Note, we can use one-dimensional array because dp[k] only depends on dp[m] where m < k.
    for (int i = 1; i <= amount; i++) {
        for (int j = 0; j < coins.size(); j++) {
            // If dp[i - coins[j]] == MIN, then it cannot update dp[i].
            if (coins[j] <= i && dp[i - coins[j]] != MIN) {
                dp[i] = max(dp[i], dp[i - coins[j]] + 1);
            }
        }
    }
    return dp[amount];
}
```

注意这两段代码的几个区别：

1. 首先注意到状态转移方程中的其中一项为 dp[i - coins[j]] + 1，即不管是 min 还是 max 均用的是 +1。因此最短路径的状态转移方程不需要对 dp[i - coins[j]] == MAX 进行单独的考虑，因为 dp[i] = min(dp[i], MAX + 1) 仍为 dp[i] 不变。而最长路径的状态转移方程需要对 dp[i - coins[j]] == MIN 的情况进行单独的考虑，否则 dp[i] = max(dp[i], MIN + 1); 在 dp[i] == MIN 时会将其更新为错误的 MIN + 1。
2. 因为前者将初始值 MAX 直接作为了“没有到达”的标志值，因此在返回时要将其转为 -1，即：return dp[amount] == MAX ? -1 : dp[amount]; 而后者由于直接用 -1 作为初始值来表征“没有到达”，因此可以直接返回实际的表项数据。

### 3.2 记忆搜索法（递归法）对标志值与初始值的处理方法
与递推法不同，递归法在更新一个表项时，访问的其所依赖的表项事先并不知道其是否已经算过，因此需要一个单独的“没有算过”的标志值，此标志既要在有效值区间 [x, y] 之外，又要区别于"无法到达"标志值。榆次同时，此初始值就是此“没有算过”标志值，因此初始值不能与“没有到达”标志值相等。一般情况下，可以选择 MAX = y + 1 或 Min = x - 1 （并且避开 -1）作为初始值和“没有算过”标志值，而 -1 作为“无法到达”标志值和相应的返回值。

最短路径问题：

```cpp
int coinChange(vector<int>& coins, int amount) {
    // Dynamic Programming: Recursion with Memoization
    // O(n * amount) time and O(amount) space
    int MAX = amount + 1; // MAX means no valid solution
    vector<int> dp(amount + 1, MAX);
    return helper(coins, amount, dp, MAX);
}

int helper(vector<int> & coins, int amount, vector<int> & dp, int MAX) {
    int ans & = dp[amount];

    // Base case, amount = 0
    if (amount == 0) {
        return ans = 0;
    }
    
    // The item has already been updated: just return it.
    if (ans != MAX) {
        return ans;
    }
    
    // The item has not been updated: update it.
    for (size_t i = 0; i < coins.size(); i++) {
        if (coins[i] <= amount && helper(coins, amount - coins[i], dp, MAX) != -1) {
            ans = min(ans, dp[amount - coins[i]] + 1);
        }
    }

    // No valid solution: return -1.
    if (ans == MAX) {
        ans = -1;
    }

    return ans;
}
```

最长路径问题：

```cpp
int coinChange(vector<int>& coins, int amount) {
    // Dynamic Programming: Recursion with Memoization
    // O(n * amount) time and O(amount) space
    int MIN = -2; // MIN means no valid solution (it should not equal to -1)
    vector<int> dp(amount + 1, MIN);
    return helper(coins, amount, dp, MIN);
}

int helper(vector<int> & coins, int amount, vector<int> & dp, int MIN) {
    int ans & = dp[amount];

	 // Base case, amount = 0
    if (amount == 0) {
        return ans = 0;
    }
    
    // The item has already been updated: just return it.
    if (ans != MIN) {
        return ans;
    }

    // The item has not been updated: update it.
    for (size_t i = 0; i < coins.size(); i++) {
        if (coins[i] <= amount && helper(coins, amount - coins[i], dp, MIN) != -1) {
            ans = max(ans, dp[amount - coins[i]] + 1);
        }
    }

    // No valid solution: return -1.
    if (ans == MIN) {
        ans = -1;
    }

    return ans;
}
```

注意，此时在进行状态更新时，均需要判断其依赖表项是否为“无法到达”对应的 -1，不再有类似递推法解决最短路径时的可以不考虑此 corner case 的特殊情况。

在递归算法中，引用所有依赖表项时都需要用递归函数对其进行调用，如 helper(coins, amount - coins[i], dp, MIN)，但是如果是同一 scope 下的第二次（及其以后）引用该表项，可以直接从表中取得，因为在第一次调用该函数时已经保证了该表项数据已经被更新过了。如：

```cpp
if (coins[i] <= amount && helper(coins, amount - coins[i], dp, MIN) != -1) {
    ans = max(ans, dp[amount - coins[i]] + 1);
}
```

# 4. 小节：对称的状态定义方式及“填表法”与“刷表法”的区别

上文介绍了动态规划的经典应用：DAG 中的最长路径和最短路径。DAG的最长路径和最短路径都可以用递推和记忆化搜索两种实现方式。打印解时既可以根据 *d* 值重新计算出每一步的最优决策，也可以在动态规划时“顺便”记录下每步的最优决 策（即“用空间换时间”）。

由于 DAG 最长（短）路径的特殊性，有两种“对称”的状态定义方式。

状态1：设 `$d \left ( i \right )$` 为从 *i* 出发的最长路径，则 `$d \left ( i \right ) = {\rm max} \left \{ d \left ( j \right ) + 1 | \left ( i, j \right ) \in E \right \}$`

状态2：设 `$d \left ( i \right )$` 为以 *i* 结束的最长路径，则 `$d \left ( i \right ) = {\rm max} \left \{ d \left ( j \right ) + 1 | \left ( j, i \right ) \in E \right \}$`

如果使用状态 2，“硬币问题”就变得和“嵌套矩形问题”几乎一样了（唯一的区别是：“嵌套矩形问题”还需要取所有 `$d \left ( i \right )$` 的最大值）！上文有意介绍了比较麻烦的状态 1，主要是为了展示一些常见技巧和陷阱，实际笔试中不推荐使用。（注：在需要考虑字典序的情况下只能使用从头开始进行记忆化搜索的状态 1。）

使用状态 2 时，有时还会遇到一个问题：状态转移方程可能不好计算，因为在很多时候，可以方便地枚举从某个结点 *i* 出发的所有边 `$\left ( i, j \right )$`，却不方便“反着”枚举 `$\left ( j, i \right )$`。特别是在有 些题目中，这些边具有明显的实际背景，对应的过程不可逆。

这时需要用“刷表法”。什么是“刷表法”呢？传统的递推法可以表示成“对于每个状态 *i*，计算 `$d \left ( i \right )$`”，或者称为“填表法”。这需要对于每个状态 *i*，找到 `$d \left ( i \right )$` 依赖的所有状态，在某些情况下并不方便。另一种方法是“对于每个状态 *i*，更新 `$d \left ( i \right )$` 所影响到的状态”，或者称为“刷表法”。对应到 DAG 最长路径的问题中，就相当于按照拓扑序枚举 *i*，对于每个 *i*，枚举边 `$\left ( i, j \right )$`，然后更新 d[j] = max(d[j], d[i] ＋ 1)。注意，一般不把这个式子叫做“状态转移方程”，因为它不是一个可以直接计算 d[j] 的方程，而只是一个更新公式。

提示：“刷表法”有时比填表法方便。但需要注意的是，**只有当每个状态所依赖的状态对它的影响相互独立时才能用刷表法**。