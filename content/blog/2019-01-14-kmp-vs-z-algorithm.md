---
title: KMP 算法与 Z 算法的对比
subtitle: KMP Algorithm VS Z Algorithm
date: '2019-01-14'
slug: kmp-vs-z-algorithm
categories:
  - Algorithms
tags:
  - pattern searching
  - pattern matching
---

通过对线性字符串匹配算法 [KMP 算法](https://kaizhang.me/note/2019/01/kmp-algorithm/) 和 [Z 算法](https://kaizhang.me/note/2019/01/z-algorithm/) 的详细分析，可以看出二者存在一定的共通的地方。

为了实现线性操作，二者均在寻求一种能够在遍历的过程中重复利用之前操作结果的一种方式，KMP 算法引入了一个向前最大匹配长度的衡量指标，而 Z 算法引入了一个向后最大匹配长度的衡量指标。

二者均有两种常见的实现形式，一种是对 pattern 进行预处理，获得其 lps table 或者 Z table，然后遍历 text 求解每一位置的向左或向右最大匹配长度；另一种是构造一个合成字符串 `pat$txt`，然后求解合成字符串的 lps table 或者 Z table。

二者有一个非常重要的区别，即 KMP 是一个 [Streaming Algorithm](https://en.wikipedia.org/wiki/Streaming_algorithm)，而 Z 算法不是，这一点是 KMP 的一个优势。

以下引自 [Knuth-Morris-Pratt string matching](https://www.nayuki.io/page/knuth-morris-pratt-string-matching)

> The text string can be streamed in because the KMP algorithm does not backtrack in the text. (This is another improvement over the naive algorithm, which doesn’t naturally support streaming.) If streaming, the amortized time to process an incoming character is O(1) but the worst-case time is O(min(m, n')), where n' is the number of text characters seen so far.