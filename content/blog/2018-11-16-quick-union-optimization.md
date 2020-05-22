---
title: Quick-Union-Optimization
subtitle: Linking-by-Size vs. Linking-by-Height vs. Linking-by-height
date: '2018-11-16'
slug: quick-union-optimization
categories:
  - Algorithms
tags:
  - graph
---

For tree-based disjoint sets and the Quick-Union Algorithm, there are two general optimizations that make the tree shorter.

One strategy happens during the union operation, and it helps the union operation to build shorter trees. There are two forms of this strategy: linking-by-size (or called union-by-size, which involves first finding the root of the two given elements and then linking them by size) and linking-by-height (or called Union-by-Height).

The second strategy, called path compression, gives the find operation the power to shorten trees.

# Linking-by-Size vs. Linking-by-Height
These two strategy try to shorten the tree by determining which node sets its link field to the other.

1. Linking-by-Size: Make the node whose set size is smaller point to the node whose set size is bigger. Break ties arbitrarily.
2. Linking-by-Height: The height of a set is the longest number of links that it takes to get from one of its items to the root. With union by height, you make the node whose height is smaller point to the node whose height is bigger. Break ties arbitrarily. If tie, increase rank of new root by 1.

# Path Compression
Path compression flattens the structure of the tree by making every node point to the root whenever find operation is used on it. This is valid, since each element visited on the way to a root is part of the same set. The resulting flatter tree speeds up future operations not only on these elements, but also on those referencing them.

Tarjan and Van Leeuwen also developed one-pass find algorithms that are more efficient in practice while retaining the same worst-case complexity: path splitting and path halving.

* Path halving: Path halving makes every other node on the path point to its grandparent.
* Path splitting: Path splitting makes every node on the path point to its grandparent.

# Union-by-Rank with Path Compression
Union-by-size or union-by-height both can be combined with path compression. However, if union-by-height is used, it is not clear how to recomputed heights efficiently. Instead, use estimated heights, an upper bound on the height of the node, called ranks. If tie, increase rank of new root by 1.

Rank needs not be very accurate as long as it always gives an upper bounds of heights is enough. When Union is performed, only the rank
of the roots may change :

* If both roots have same rank: rank of new root increases by 1.
* Else, no change.

Union-by-rank works exactly like union by height, except you don't really know the height of the set. Instead, you use the set's rank, which is what its height would be, were you using union by height. The reason that you don't know the height is that when you call find(i) on an element, you perform "path compression." What this does is set the link of all non-root nodes from the path from i to the root, to the root.

**Property: For link-by-rank, the tree roots, node ranks, and elements within a tree are the same with or without path compression.**

# Union-by-Size with Path Compression vs. Union-by-Rank with Path Compression
Linking by rank seems preferable to linking by size since it requires less storage: it needs only log log n bits per node (to store a rank in the range [0, log n], actually lower bound of log n) instead of log n bits per node (to store a size in the range [1, n]). Linking by rank also tens to require less updating than linking by size. (Because we update the size of the new root for every linking operation, but we only update the rank of the new root by 1 when there is a tie.)

**But in terms of time complexity, all the bounds hold equally for linking by rank and linking by size (both combined with path compression or not).**

References:

1. [Worst-Case Analysis of Set Union Algorithms](http://www.csd.uwo.ca/~eschost/Teaching/07-08/CS445a/p245-tarjan.pdf)
2. [CS302 Lecture Notes - Disjoint Sets](http://web.eecs.utk.edu/~plank/plank/classes/cs302/Notes/Disjoint/)
3. [https://www.cs.princeton.edu/courses/archive/spring13/cos423/lectures/UnionFind.pdf](https://www.cs.princeton.edu/courses/archive/spring13/cos423/lectures/UnionFind.pdf)
4. [CS4311 Design and Analysis of Algorithms, Lecture 21: Data Structures for Disjoint Sets II](http://www.cs.nthu.edu.tw/~wkhon/algo08-lectures/lecture21.pdf)
5. [Lecture 15 Graph Algorithms III: Union-Find](https://www.cs.cmu.edu/~avrim/451f11/lectures/lect1020.pdf)
6. [Union-find algorithms](https://cs.gmu.edu/~rcarver/cs310/UnionFind.pdf)
