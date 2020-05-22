---
title: 线性字符串匹配算法：KMP 算法
subtitle: Linear String Matching Algorithm
date: '2019-01-13'
slug: kmp-algorithm
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

# 2 KMP 算法模式匹配程度指标的引入
注：下文中 `txt` 表示长度为 n 的原始字符串，`pat` 表示长度为 m 的模式字符串。

KMP 算法对于 `txt` 的任一位置 `txt[i]` 引入了一个衡量模式匹配程度的指标，本文用 `leftMatchLen` 表示，其意义为以 `pat[0]` 开头的字符串（prefix）和以 `txt[i]` 结尾的字符串（suffix）能够匹配的最大长度。当 `leftMatchLen == m` 时，表示模式匹配成功。

那么问题转化为了：**如何在线性时间内遍历 `txt` 并对每一个位置 `txt[i]` 求出其相应的指标值呢？**

# 3 指标值 `leftMatchLen` 的线性计算
## 3.1 计算流程（第二匹配长度的引入）
* 初始化：`leftMatchLen = 0` 和 `i = 0`。
* 遍历 `txt[i], i = 0...n-1`，求解相应的 `leftMatchLen` 值。
    * 如果继承之前的最大匹配长度 `leftMatchLen` 匹配失败，即 `txt[i] != pat[leftMatchLen]`，但是 `leftMatchLen > 0` 的话，则存在能够继承 `txt[i]` 之前的**第二匹配长度**的可能性，因此需要逐步将 `leftMatchLen` 逐步重置为其对应的第二匹配长度，直到发现继续匹配成功 `txt[i] == pat[leftMatchLen]` 或者 `leftMatchLen` 已经递减到 0 为止。
    * 在上述过程停止后，`leftMatchLen == 0` 或者为能够成功继承的匹配长度（可能是第一匹配长度或第二匹配长度），此时只需判断当前位置是否匹配，若匹配，`leftMatchLen++`，否则保持其不变。截至此时，`txt[i]` 处的 `leftMatchLen` 已求解完成，判断其是否等于 `pat` 的长度即可。

那么怎么求第二匹配长度呢？

## 3.2 最大匹配长度 `leftMatchLen` 对应的第二匹配长度的求解
由于第二匹配长度小于第一匹配长度，因此第二匹配对应于 `txt[i-leftMatchLen...i-1]` 内部（排除整体本身，因为其为第一匹配）的以 `txt[i-1]` 结尾的最大匹配。

注意到，根据第一匹配长度为 `leftMatchLen` 可知 `txt[i-leftMatchLen...i-1]` 与 `pat[0...leftMatchLen-1]` 是完全一致的，因此 `txt[i-1]` 处的第二匹配等价于 `pat[leftMatchLen-1]` 处在以其自身所在字符串 `pat` 为 pattern 时的第二匹配。

因此，问题转化为求解 `pat` 任一位置 `pat[i]` 相对于其自身的第二匹配长度的问题。

注意到，`pat` 在以其自身为 pattern 时的第二匹配长度与第一匹配长度的区别在于第二匹配排除了 `pat[0...i]` 这个永远正确但毫无意义的第一匹配。本文中用到的术语“第二匹配”一般被称为 **合适匹配**，或者 **Proper Longest Prefix Suffix**。

对应于算法实现而言，合适匹配（或第二匹配）与第一匹配只有一个小小的区别，即合适匹配将 `pat[0]` 处的 `leftMatchLen` 硬编码为了 0：

* 第一匹配初始化为 `leftMatchLen = 0` 和 `i = 0`，然后从 `i = 0` 开始遍历求解所有位置 `txt[i]` 对应的指标值 `leftMatchLen`。
* 合适匹配初始化 `leftMatchLen = 0` 和 `i = 1`，然后从 `i = 1` 开始遍历求解所有位置 `txt[i]` 对应的指标值 `leftMatchLen`。

# 4 算法版本一：预处理 pattern 字符串，建立 lps table
## 4.1 代码
根据上述分析，KMP 算法的一种实现方式为：

首先预处理 `pat` 字符串，并建立与其长度相等的第二匹配长度表 lps table。

然后遍历 `txt` 字符串，依次求解 `txt[i]` 处的第一匹配指标值 `leftMatchLen`。

```cpp
/* Searches for the given pattern string @pat in the given text
   string @txt using the Knuth-Morris-Pratt string matching algorithm.
   If the pattern is found, return the index of the start of the
   earliest match. Otherwise returned -1. If @pat is empty, return 0. */

int KMPSearch(const string & txt, const string & pat) {
    if (pat.empty()) {
        return 0;  // Immediate match
    }

    // Preprocess the pattern: build lps table
    vector<size_t> lps = computeLpsTable(pat);

    size_t j = 0;  // The 1st longest match length @lfetMatchLen
                   // Since it's also the index of pat, so we just use j

    // Walk through txt to calculate every leftMatchLen(or j)
    // Note, i start from 0: it's the 1st longest match length
    for (size_t i = 0; i < txt.size(); i++) {
        while (j > 0 && txt[i] != pat[j]) {
            // Update j to the next level 2nd longest match length
            j = lps[j - 1];  // Strictly decreasing
        }
        if (txt[i] == pat[j]) {
            // Next char matched, increment position
            j++;
            // Check if found
            if (j == pat.size()) {
                return i - j + 1;
            }
        }
    }

    return -1;  // Not found
}

/* Preprocess @pat and build the lps table */
vector<size_t> computeLpsTable(const string & pat) {
    vector<size_t> lps(pat.size());

    // Here is the key difference between 1st and 2nd longest match
    // Instead of starting from index 0 as in 1st longest match,
    // here we set lps[0] = 0, and start from index 1
    lps[0] = 0;
    size_t j = 0;  // The 2nd longest match length

    for (size_t i = 1; i < pat.size(); i++) {
        while (j > 0 && pat[i] != pat[j]) {
            j = lps[j - 1];
        }
        if (pat[i] == pat[j]) {
            j++;
        }
        lps[i] = j;
    }

    return lps;
}
```

## 4.2 复杂度
预处理构建 pattern 的 lps table 的复杂度为：O(m) time & O(m) space

遍历求解 `txt[i]` 处的向左最大匹配长度的复杂度为 O(n) time & O(1) space

因此，总的复杂度为：O(m + n) time & O(m) space

# 5 算法版本二：构建合成字符串 `pat$txt`
## 5.1 代码
除了上述预处理 `pat` 构建 lps table 的方式之外，KMP 算法还有另外一种实现形式，即创建一个新的字符串 `pat$txt`，其中 `$` 不能在 `pat` 或着 `txt` 中出现，然后求解这个新的字符串对其自己的第二匹配长度数组 lps 即可。

注意到，由于中间被 `$` 隔开，所以合成字符串的 `txt` 部分的第二匹配长度即为未合成时单独的 `txt` 对 `pat` 的第一匹配长度。

```cpp
/* Searches for the given pattern string @pat in the given text
   string @txt using the Knuth-Morris-Pratt string matching algorithm.
   If the pattern is found, return the index of the start of the
   earliest match. Otherwise returned -1. If @pat is empty, return 0. */

int KMPSearch(const string & txt, const string & pat) {
    if (pat.empty()) {
        return 0;  // Immediate match
    }

    // Create concatenated string "pat$txt"
    string concat = pat  + "$" + txt;

    // Create lps table
    vector<size_t> lps(concat.size());

    lps[0] = 0;
    size_t j = 0;  // The 2nd longest match length

    // Note, i start from 1: it's the 2nd longest match length
    for (size_t i = 1; i < concat.size(); i++) {
        while (j > 0 && concat[i] != concat[j]) {
            // Update j to the next level 2nd longest match length
            j = lps[j - 1];  // Strictly decreasing
        }
        if (concat[i] == concat[j]) {
            // Next char matched, increment position
            j++;
            // Check if found
            if (j == pat.size()) {
                return i - 2 * j;
            }
        }
        lps[i] = j;
    }

    return -1;  // Not found
}
```

## 5.2 复杂度
总的复杂度为：O(m + n) time & O(m + n) space

因此相对于版本一，时间复杂度一致，空间复杂度较差，但是代码更加简洁。
