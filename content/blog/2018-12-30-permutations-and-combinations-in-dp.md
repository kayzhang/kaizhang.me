---
title: 背包问题中的排列与组合
subtitle: 两层循环的嵌套顺序的区别
date: '2018-12-30'
slug: permutations-and-combinations-in-dp
categories:
  - Algorithms
tags:
  - dynamic programming
toc: true
---

本文以背包问题中的硬币/零钱问题来阐述在动态规划中存在的排列与组合问题。

# 1 问题描述
[LeetCode 322. Coin Change](https://leetcode.com/problems/coin-change/)

> You are given coins of different denominations and a total amount of money *amount*. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return `-1`.
> 
> You may assume that you have an infinite number of each kind of coin.

这是最基本的完全背包问题，根据状态定义的不同，可以衍生出两个不同版本的算法。

# 2 第一种状态状态定义方式：Use-It-Or-Lose-It

## 2.1 状态的定义

**此种方式将状态划分为 2 个维度**，即 `dp[i][a]` 表示使用前 `i` 种硬币，总金额为 `a` 时，需要的最少硬币数量。其状态转移方程为：

`$${\rm dp} \left [ i \right ] \left [ a \right ] = {\rm min} \left \{ {\rm dp} \left [ i - 1 \right ] \left [ a - k \cdot {\rm coins} \left [ i \right ] \right ] + k | k \cdot {\rm coins} \left [ i \right ] \leq a , k = 0, 1, 2, ... \right \}$$`

可以把完全背包问题转化为 01 背包问题，用 Use-It-Or-Lose-It 的思路可以得到以下状态转移方程：

`dp[i][a] = min{dp[i - 1][a], dp[i][a - coins[i]] + 1}`

这种定义方式的特点为：**对于同一种硬币的组合，其只会以一个特定的代表顺序(即其在数组 `coins` 中的顺序)被计算一次。最终求的是所有不同组合中用到的最少硬币数。**

## 2.2 两层循环的顺序

此时由于状态是两维的，所以 Tabulation 需要两层外围的循环：硬币面值(`coins`)的循环层，金额(`amount`)的循环层。

**此时，由于 `dp[i][a]` 取决于其上边与左边的值，因此填充 dp table 的顺序既可以一列一列进行，也可以一行一行进行。所以对两层循环的顺序并无要求。**

下表为 dp table，箭头为表项更新顺序，并不代表数据依赖关系。

|            | 0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 | ... | amount |
|:----------:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:---:|:------:|
|      0     | 0 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | ... |   -1   |
|      2     | . |  → |    |    |    |    |    |    |     |        |
|      5     | ↓ |    |    |    |    |    |    |    |     |        |
|      7     |   |    |    |    |    |    |    |    |     |        |
|     ...    |   |    |    |    |    |    |    |    |     |        |
| coins[n-1] |   |    |    |    |    |    |    |    |     |  Dest  |

## 2.3 常规的二维 dp table 实现

初始值的确定：只需要对 `coins` 为空进行初始化即可，`amount == 0` 不需要初始化。

### 2.3.1 `coins` 在外层循环，`amount` 在内层循环

|            | 0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 | ... | amount |
|:----------:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:---:|:------:|
|      0     | 0 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | ... |   -1   |
|      2     | . |  → |  → |  → |  → |  → |  → |  → |  →  |    →   |
|      5     | . |  → |  → |  → |    |    |    |    |     |        |
|      7     |   |    |    |    |    |    |    |    |     |        |
|     ...    |   |    |    |    |    |    |    |    |     |        |
| coins[n-1] |   |    |    |    |    |    |    |    |     |  Dest  |

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        // Tabulation: Combination Version
        // 二维 dp table: coins 循环在外，amount 循环在内
        // O(n * amount) time & O(n * amount) space
        
        int n = coins.size();
        
        // Create dp table
        vector<vector<int> > dp(n + 1, vector<int>(amount + 1));
        
        // Initialization for n = 0
        dp[0][0] = 0;
        for (int j = 1; j <= amount; j++) {
            dp[0][j] = -1;
        }
        
        // Build dp table
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= amount; j++) {
                // Lose it
                dp[i][j] = dp[i - 1][j];
                
                // Use it
                int v = j - coins[i - 1];
                if (v >= 0 && dp[i][v] != -1) {
                    int useIt = dp[i][v] + 1;
                    if (dp[i][j] == -1 || useIt < dp[i][j]) {
                        dp[i][j] = useIt;
                    }
                }
            }
        }
        
        return dp[n][amount];
    }
};
```

### 2.3.2 `amount` 在外层循环，`coins` 在内层循环

|            | 0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 | ... | amount |
|:----------:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:---:|:------:|
|      0     | 0 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | ... |   -1   |
|      2     | . |  . |    |    |    |    |    |    |     |        |
|      5     | ↓ |  ↓ |    |    |    |    |    |    |     |        |
|      7     | ↓ |  ↓ |    |    |    |    |    |    |     |        |
|     ...    | ↓ |  ↓ |    |    |    |    |    |    |     |        |
| coins[n-1] | ↓ |    |    |    |    |    |    |    |     |  Dest  |

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        // Tabulation: Combination Version
        // 二维 dp table: amount 循环在外，coins 循环在内
        // O(n * amount) time & O(n * amount) space
        
        int n = coins.size();
        
        // Create dp table
        vector<vector<int> > dp(n + 1, vector<int>(amount + 1));
        
        // Initialization for n = 0
        dp[0][0] = 0;
        for (int j = 1; j <= amount; j++) {
            dp[0][j] = -1;
        }
        
        // Build dp table
        for (int j = 0; j <= amount; j++) {
            for (int i = 1; i <= n; i++) {
                // Lose it
                dp[i][j] = dp[i - 1][j];
                
                // Use it
                int v = j - coins[i - 1];
                if (v >= 0 && dp[i][v] != -1) {
                    int useIt = dp[i][v] + 1;
                    if (dp[i][j] == -1 || useIt < dp[i][j]) {
                        dp[i][j] = useIt;
                    }
                }
            }
        }
        
        return dp[n][amount];
    }
};
```
### 2.3.3 复杂度分析

一共 `(n + 1) * (amount + 1)` 个表项，每个表项为 O(1) time，因此：

O(n * amount) time & O(n * amount) space

## 2.4 空间复杂度的优化：只维护 dp table 的一部分 Sliding Window

本题的 dp table 的依赖关系存在一个特点：

即每个表项 `dp[i][a]` 只依赖于 `dp[i - 1][a]` 与 `dp[i][a - coins[i]]`，由此引入一个优化空间的方法。

**当一个表项 `dp[i]` 最远的依赖为 `dp[i - k]` 时，我们只需维护一个长度为 `k` 的 Sliding Window，将 `dp[i]` 写到 `dp[i - k]` 所在的位置对其进行覆盖即可。空间复杂度由 O(n) 降为 O(k)，时间复杂度不变。**

|            | 0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 | ... | amount |
|:----------:|:-:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:---:|:------:|
|      0     | 0 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | ... |   -1   |
|      2     |   |    |    |    |    |    |    |    |     |        |
|      5     |   |    |    |    |    |    | d1 |    |     |        |
|      7     |   |    |    | d2 |    |    |  x |    |     |        |
|     ...    |   |    |    |    |    |    |    |    |     |        |
| coins[n-1] |   |    |    |    |    |    |    |    |     |  Dest  |

此表中，`x` 依赖于上边相邻行的 `d1` 与左边非相邻列的 `d2`。

### 2.4.1 `coins` 在外层循环，`amount` 在内层循环

若采用此种嵌套方式，则相当于一行一行地从左到右填充 dp table。因此，此时只需维护一个只有一行的一维 dp table 即可。

初始值如下，对应于 `coins` 为空的一行：

| 0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 | ... | amount |
|:-:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:---:|:------:|
| 0 | -1 | -1 | -1 | -1 | -1 | -1 | -1 | ... |   -1   |

之后还需要对此行更新 `n` 次，可以把 `1` 到 `n` 简化为 `0` 到 `n - 1`。

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        // Tabulation: Combination Version
        // 空间优化：一维 dp table: coins 循环在外，amount 循环在内
        // O(n * amount) time & O(n * amount) space
        
        int n = coins.size();
        
        // Create dp table
        vector<int> dp(amount + 1);
        
        // Initialization for n = 0
        dp[0] = 0;
        for (int j = 1; j <= amount; j++) {
            dp[j] = -1;
        }
        
        // Build dp table
        for (int i = 0; i < n; i++) { // Update the row n times
            for (int j = 1; j <= amount; j++) {
                // Lose it: do nothing
                
                // Use it
                int v = j - coins[i];
                if (v >= 0 && dp[v] != -1) {
                    int useIt = dp[v] + 1;
                    if (dp[j] == -1 || useIt < dp[j]) {
                        dp[j] = useIt;
                    }
                }
            }
        }
        
        return dp[amount];
    }
};
```

### 2.4.2 `amount` 在外层循环，`coins` 在内层循环

若采用此种嵌套方式，则相当于一列一列地从左到右填充 dp table。因此，此时只需维护一个 `(n + 1) * k` 的 Sliding Window，其中 `k` 为最大的硬币面值。

代码实现有点复杂，此处不赘述。

# 3 第二种状态定义方式：DAG 上的动态规划

参考上一篇文章：[动态规划的相关细节探讨 -- DAG 上的动态规划](https://kaizhang.me/note/2018/12/more-dynamic-programming/)

**此种方式将状态只划分为 1 个维度**，即 `dp[a]` 表示总面值为 `a`，并且使用所有硬币的情况下，最少需要用到的硬币个数。其状态转移方程为：

`$${\rm dp} \left [ a \right ] = {\rm min} \left \{ {\rm dp} \left [ a - {\rm coins} \left [ i \right ] \right ] + 1 | {\rm coins} \left [ i \right ] \leq a , i = 0, 1, ..., N - 1 \right \}$$`

这种定义方式的特点为：**每一步决策时所有的硬币均可选，硬币选择的顺序是不确定的，因此同一硬币组合方式的每一种排列方式均被计算了一次。最终求的是所有不同排列中用到的最少硬币数。**如 1 3 5 和 5 3 1 被视为两种不同的方案。

其在具体代码的实现上的特征为：**`amount` 的循环在外层，`coins` 的循环在内层，表示当前考虑的下一枚硬币。并且此时，只有外层循环对应于 dp table 的表项的位置，内层循环为同一个表项的多种不同决策。**

## 3.1 代码实现

初始值如下：

| 0 | 1 | 2 |  3 | 4 | 5 |  6 | 7 | ... | amount |
|:-:|:-:|:-:|:--:|:-:|:-:|:--:|:-:|:---:|:------:|
| 0 |   |   |    |   |   |    |   |     |        |

由于此时更新 `dp[i]` 时，可能 `i` 比最小的面值都小，导致其每一种硬币都无法用，进而出现了边界情况。因此，在更新 `dp[1] ... dp[amount]` 每个表项时，要处理边界情况，即将该表项的值首先置位无解时的特征值，如本题的 `-1`。

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        // Tabulation: Permutation Version
        // 一维 dp table: amount 循环在外，coins 循环在内
        // O(n * amount) time & O(n * amount) space
        
        int n = coins.size();
        
        // Create dp table
        vector<int> dp(amount + 1);
        
        // Initialization for amount = 0
        dp[0] = 0;
        
        // Build dp table
        for (int j = 1; j <= amount; j++) {
            // 边界情况的处理：首先将 dp[j] 置位无解标志值 -1
            // 当然也可以用一个 bool isValid = false; 来记录
            dp[j] = -1;
            
            // 决定此步选择哪一枚硬币：此层循环非 dp table 表项选择的循环！
            for (int i = 0; i < n; i++) {
                int v = j - coins[i];
                if (v >= 0 && dp[v] != -1) {
                    int useIt = dp[v] + 1;
                    if (dp[j] == -1 || useIt < dp[j]) {
                        dp[j] = useIt;
                    }
                }
            }
        }
        
        return dp[amount];
    }
};
```

## 3.2 复杂度分析

dp table 为 `1 * (amount + 1)`，因此空间复杂度为 O(amount)，而更新每个表项的部分用到了 Use-Which 的循环来遍历 `coins`。

O(n * amount) time & O(amount) space

# 4 两种状态定义方式的对比

## 4.1 代码形式与逻辑的对比

在形式上二者非常相似，尤其是 2.4.1 节中将二维 dp table 优化为一维 dp table 的 `coins` 在外层循环，`amount` 在内层循环的 Combination 状态定义版本，与第二种 Permutation 的状态定义版本的代码非常地相似，唯一的明显区别为二者的循环嵌套关系是相反的。

但是一定要注意，二者虽然在形式上非常相似，但是内在的逻辑是截然不同的。

* **Combination 版本，虽然形式上 dp table 是一维的，但是在物理意义上其 dp table 还是二维的，因此两层循环都是在确定当前待更新表项的位置。两层循环之内是 Use-It-Or-Lose-It 的决策部分。**
* **Permutation 版本，其 dp table 在物理意义上就是一维的，只有最外层循环是在确定当前待更新表项的位置。在最外层循环之内是用一个循环完成的 Use-Which 的决策部分。**

## 4.2 时间复杂度的对比

二者的时间复杂度都是 O(n * amount)，由此不禁引起一个疑问：

> 这两种状态定义方式分别为 Combination 和 Permutation，为什么时间复杂度是一样的呢？ 

这是因为：

对于每一个表项而言，我们考虑的硬币组合情况是小于硬币排列情况的，但是组合版本实际上把原问题分成了更多的子问题：

* 组合版本将原问题划分为了 O(n * amount) 个不同的子问题，每个问题的时间复杂度为 O(1)，因此总的时间复杂度为 O(n * amount)
* 排列版本将原问题划分为了 O(amount) 个不同的子问题，但是每个问题的时间复杂度为 O(n)，因此总的时间复杂度也为 O(n * amount)

# 5 对于方案计数问题要分清楚问的组合计数还是排列计数

[LeetCode 518. Coin Change 2](https://leetcode.com/problems/coin-change-2/)

问题描述：

> You are given coins of different denominations and a total amount of money. Write a function to compute the number of combinations that make up that amount. You may assume that you have infinite number of each kind of coin.

对于求最优解的问题，采用前述的组合或排列的方式是没有区别的，结果是一致的，时间、空间复杂度也是一致的。但是对于方案计数问题，需要分清楚题目问的是组合还是排列的种类个数。

对于本题而言，对于同一种硬币组成而言，其排列顺序我们是不关心的，因此是一种组合计数问题，需要采用 Use-It-Or-Lose-It 的状态定义方式。

此种方式将状态划分为 2 个维度，即 `dp[i][a]` 表示使用前 `i` 种硬币，总金额为 `a` 时，可能存在的组合总数。“无法到达”的标志值为 0。其状态转移方程为：

`dp[i][a] = dp[i][a - coins[i]] + dp[i - 1][a]`

## 5.1 常规的二维 dp table 实现

如前所述，此时循环嵌套的顺序无所谓。

```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        // Tabulation: Combination Version
        // 二维 dp table: coins, amount 循环嵌套关系无所谓
        // O(n * amount) time & O(n * amount) space
        
        int n = coins.size();
        
        // Create dp table
        vector<vector<int> > dp(n + 1, vector<int>(amount + 1));
        
        // Initialization for n = 0
        dp[0][0] = 1;
        for (int j = 1; j <= amount; j++) {
            dp[0][j] = 0;
        }
        
        // Build dp table: 循环嵌套关系无所谓
        
        /*
        for (int j = 0; j <= amount; j++) {
            for (int i = 1; i <= n; i++) {
        */
        
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= amount; j++) {
                // Lose it
                dp[i][j] = dp[i - 1][j];
                
                // Use it
                if (j >= coins[i - 1]) {
                    dp[i][j] += dp[i][j - coins[i - 1]];
                }
            }
        }
        
        return dp[n][amount];
    }
};
```

## 5.2 空间优化为一维 dp table 的实现

如前所述，当 `coins` 在外层循环，`amount` 在内层循环时，可以将 dp table 优化为一维的。

```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        // Tabulation: Combination Version
        // 空间优化：一维 dp table: coins 循环在外，amount 循环在内
        // O(n * amount) time & O(n * amount) space
        
        int n = coins.size();
        
        // Create dp table
        vector<int> dp(amount + 1);
        
        // Initialization for n = 0
        dp[0] = 1;
        for (int j = 1; j <= amount; j++) {
            dp[j] = 0;
        }
        
        // Build dp table
        for (int i = 0; i < n; i++) { // Update the row n times
            for (int j = 1; j <= amount; j++) {
                // Lose it: do nothing
                
                // Use it
                if (j >= coins[i]) {
                    dp[j] += dp[j - coins[i]];
                }
            }
        }
        
        return dp[amount];
    }
};
```
