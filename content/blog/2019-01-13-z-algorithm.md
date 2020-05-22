---
title: 线性字符串匹配算法：Z 算法
subtitle: Linear String Matching Algorithm
date: '2019-01-13'
slug: z-algorithm
categories:
  - Algorithms
tags:
  - pattern searching
  - pattern matching
toc: true
---

# 1 字符串匹配算法的描述：String-searching algorithm

> In computer science, [string-searching algorithms](https://en.wikipedia.org/wiki/String-searching_algorithm), sometimes called string-matching algorithms, are an important class of string algorithms that try to find a place where one or several strings (also called patterns) are found within a larger string or text.

字符串匹配问题的具体形式可能发生变化，如查找第一个匹配位置，或者查找所有匹配位置等等，但其无本质区别。

# 2 Z 算法模式匹配程度指标的引入
注：下文中 `txt` 表示长度为 n 的原始字符串，`pat` 表示长度为 m 的模式字符串。

Z 算法对于 `txt` 的任一位置 `txt[i]` 引入了一个衡量模式匹配程度的指标，本文用 `rightMatchLen` 表示，其意义为以 `pat[0]` 开头的字符串（prefix）和以 `txt[i]` 开头的字符串（prefix）能够匹配的最大长度。当 `rightMatchLen == m` 时，表示模式匹配成功。

那么问题转化为了：**如何在线性时间内遍历 `txt` 并对每一个位置 `txt[i]` 求出其相应的指标值呢？**

# 3 指标值 `rightMatchLen` 的线性计算
## 3.1 计算流程
首先类似于求解 Longest palindromic substring 的 [Manacher's Algorithm](https://www.geeksforgeeks.org/manachers-algorithm-linear-time-longest-palindromic-substring-part-1/) 算法，Z 算法在遍历 `txt[i]` 求解指标值的时候，保持记录 2 个值：L、R。其中 R 表示的是目前为止能够匹配到的最右侧的位置，L 为 R 对应的起始位置，即 `txt[L...R]` 与 `pat[0...R-L]` 是完全一致的。

与 [KMP 算法](https://kaizhang.me/note/2019/01/kmp-algorithm/) 类似，Z 算法的流程如下：

* 初始化：`rightMatchLen = 0` 和 `i = 0`，并初始化 `L = 0， R = 0`。
* 遍历 `txt[i], i = 0...n-1`，求解相应的 `rightMatchLen` 值。
    * 如果 `i <= R`，那么 `txt[i...R]` 和 `pat[i-L...R-L]` 是完全一致的，因此可以建立 `pat` 相对于自己的 `rightMatchLen` 数组 Z，并通过查表得到 `Z[i-L]`。
        * 如果 `i + Z[i-L] - 1  < R`，那么说明 `txt[i]` 处向右最大匹配在 `txt[i...R]` 之内，那么 `rightMatchLen = Z[k] < pat.size()`，无需更新任何值，直接 `i++` 进入下一循环。
        * 如果 `i + Z[i-L] - 1  >= R`，那么说明 `txt[i]` 处向右最大匹配最短可以到 `txt[i...R]`，并且有可能继续向 R 的右侧延伸，此时需要从 R 的位置开始对 `pat` 的相应位置进行继续匹配。
    * 如果 `i > R`，那么直接从 `i` 的位置开始匹配即可。

与 KMP 算法类似，求解 `txt[i]` 处的向右最大匹配长度 `rightMatchLen` 与求解 `pat[i]` 相对于 `pat` 自身的向右最大匹配长度时存在一个小小的区别，即二者的循环初始位置不同：

* 前者从 `i = 0` 开始遍历求解所有位置 `txt[i]` 对应的指标值 `rightMatchLen`。
* 后者从 `i = 1` 开始遍历求解所有位置 `txt[i]` 对应的指标值 `rightMatchLen`。

原因在于，后者要排除掉 `pat[0]` 处始终可以向右完全匹配这一情况，因为其会将 R 更新为 `m-1`以对后续的求解造成错误的影响。

但与 KMP 不同的是，KMP 算法必须初始化 `lps[0] = 0`，这是因为在求解 `leftMatchLen` 时会用到 `lps[0]`，（`leftMatchLen = lps[leftMatchLen - 1]` 在 `leftMatchLen == 1` 时会用到 `lps[0]`）；但是 Z 算法中的 `Z[0]` 在求解 `rightMatchLen` 时永远不会被用到，这是因为 `Z[i - L]` 中，`i` 始终是比 L 大的，`i - L > 0`，所以可以不必初始化 `Z[0]`。

# 4 算法版本一：预处理 pattern 字符串，建立 Z table
## 4.1 代码
根据上述分析，Z 算法的一种实现方式为：

首先预处理 `pat` 字符串，并建立与其长度相等的向右最大匹配数组 Z table。

然后遍历 `txt` 字符串，依次求解 `txt[i]` 处的向右最大匹配长度 `rightMatchLen`。

```cpp
/* Searches for the given pattern string @pat in the given text
   string @txt using the Z string matching algorithm.
   If the pattern is found, return the index of the start of the
   earliest match. Otherwise returned -1. If @pat is empty, return 0. */

int ZSearch(const string & txt, const string & pat) {
    if (pat.empty()) {
        return 0;  // Immediate match
    }

    // Preprocess the pattern: build Z table
    vector<size_t> Z = computeZTable(pat);

    // txt[L...R] matches pat[0...R-L] with the largest R yet.
    int L = 0;
    int R = -1;  // Here we use int instead of size_t, since -1 is used.

    // Walk through txt to calculate every rightMatchLen (or R - L + 1)
    // Note, i start from 0
    for (int i = 0; i < txt.size(); i++) {
        // i > R, or txt[i...R] == pat[0...R-i]
        if (i > R || i + Z[i - L] - 1 >= R) {
            L = i;

            if (i > R) {
                // Need to continue matching from i
                R = i;
            }

            while (R < txt.size() && txt[R] == pat[R - i]) {
                // Check if found
                if (R - i == pat.size() - 1) {
                    return i;
                }
                R++;
            }
            R--;  // Decrement R to the last index which matches
        }
    }

    return -1;  // Not found
}

/* Preprocess @pat and build the Z table */
vector<size_t> computeZTable(const string & pat) {
    vector<size_t> Z(pat.size());

    // pat[L...R] matches pat[0...R-L] with the largest R yet.
    size_t L = 0;
    size_t R = 0;

    // Walk through pat to calculate every rightMatchLen (or R - L + 1)
    // Note, here i should start from 1
    for (size_t i = 1; i < pat.size(); i++) {
        // i >= R, or pat[i...R] == pat[0...R-i]
        if (i > R || i + Z[i - L] - 1 >= R) {
            L = i;

            if (i > R) {
                // Need to continue matching from i
                R = i;
            }

            while (R < pat.size() && pat[R] == pat[R - i]) {
                R++;
            }
            R--;  // Decrement R to the last index which matches

            Z[i] = R - i + 1;
        }
    }

    return Z;
}
```

## 4.2 复杂度
预处理构建 pattern 的 Z table 的复杂度为：O(m) time & O(m) space

遍历求解 `txt[i]` 处的向右最大匹配长度的复杂度为 O(n) time & O(1) space

因此，总的复杂度为：O(m + n) time & O(m) space

# 5 算法版本二：构建合成字符串 `pat$txt`
## 5.1 代码
除了上述预处理 `pat` 构建 Z table 的方式之外，Z 算法还有另外一种实现形式，即创建一个新的字符串 `pat$txt`，其中 `$` 不能在 `pat` 或着 `txt` 中出现，然后求解这个新的字符串对其自己的向右最大匹配数组 Z table 即可。注意，应从 index `i = 1` 开始遍历。

```cpp
/* Searches for the given pattern string @pat in the given text
   string @txt using the Z string matching algorithm.
   If the pattern is found, return the index of the start of the
   earliest match. Otherwise returned -1. If @pat is empty, return 0. */

int ZSearch(const string & txt, const string & pat) {
    if (pat.empty()) {
        return 0;  // Immediate match
    }

    // Create concatenated string "pat$txt"
    string concat = pat  + "$" + txt;

    // Create Z table
    vector<size_t> Z(concat.size());

    // pat[L...R] matches pat[0...R-L] with the largest R yet.
    size_t L = 0;
    size_t R = 0;

    // Walk through concat to calculate every rightMatchLen (or R - L + 1)
    // Note, here i should start from 1
    for (size_t i = 1; i < concat.size(); i++) {
        // i >= R, or concat[i...R] == concat[0...R-i]
        if (i > R || i + Z[i - L] - 1 >= R) {
            L = i;

            if (i > R) {
                // Need to continue matching from i
                R = i;
            }

            while (R < concat.size() && concat[R] == concat[R - i]) {
                // Check if found
                if (R - i == pat.size() - 1) {
                    return i - pat.size() - 1;
                }
                R++;
            }
            R--;  // Decrement R to the last index which matches

            Z[i] = R - i + 1;
        }
    }

    return Z;
}
```

## 5.2 复杂度
总的复杂度为：O(m + n) time & O(m + n) space

因此相对于版本一，时间复杂度一致，空间复杂度较差，但是代码更加简洁。
