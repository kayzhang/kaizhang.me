---
title: Graph Traversals
subtitle: DFS & BFS
date: '2018-09-02'
slug: graph-traversals
categories:
  - Algorithms
  - Graph
tags:
  - traversal
---

Reference:  
[GRAPHS](https://people.eecs.berkeley.edu/~jrs/61b/lec/28)  
[GRAPHS (continued)](https://people.eecs.berkeley.edu/~jrs/61b/lec/29)

# Graph Presentation
对于一个 graph: G = (V, E) 而言，有两种常用的实现方式。

1. _adjacency_ _matrix_: 一个 |V|-by-|V| 的 boolean 数组，其中第 i 行第 j 列的元素表示 vertex (i, j) 是否存在。对于无向图而言，该矩阵关于主对角线对称。另外需要指出的一点是，有向图最多有 |V|^2 条边，无向图最多有 (|V|^2 + V) / 2) 条边。但在实际应用中，边数要比 Theta(|V|^2) 小得多，这种图称为稀疏图 （_sparse_）。对于稀疏图而言，adjacency matrix 显然是非常浪费的，因此引入 graph 的另一种实现方式。
2. _adjacency_ _list_: 此方式适用于稀疏图，其并非只用一个 list，而是用一系列的 lists 来表示。每个 vertex v 对应于一个 list 来存储所有从 v 出发经一条边能够到达的其他 vertices。注意此处的 list 指的是广义的 list，数组、链表、search tree 都是可行的，比如：当节点名为数字时，可以用数组的数组来实现，即用一个长度为 |V| 的数组代表所有的节点，然后该数组的每个元素中放置一个存储该节点的所有邻节点的数组；当节点名为字符串时，可以用 hash 表来实现，即 hash 表的 key 为所有的节点名，value 为相应节点对应的邻节点组成的链表。adjacency list 消耗的总内存为 Theta(|V| + |E|)。

An adjacency list is more space- and usually time-efficient than an adjacency matrix for a sparse graph, but less efficient for a *complete graph*.  A complete graph is a graph having every possible edge; that is, for every vertex u and every vertex v, (u, v) is an edge of the graph.
# Graph Traversals
graph 的遍历有两种常用的方式：DFS (Depth-first search) 和 BFS (Breadth-first search)。对于 undirected tree 而言，preorder traversal 和 postorder traversal 均为 DFS，而 level-order traversal 属于 BFS。与 tree 不同的是， graph 的任意两个节点之间的 path 可以是不唯一的，即每个节点均可以通过多种路径被访问，因此需要给每个节点添加一个标志位，即一个称为 "visited” 的 boolean field，来防止重复访问同一个节点。

## Depth-first Search

```java
public class Graph {
    // Before calling dfs(), set every "visited" flag to false; takes O(|V|) time
    public void dfs(Vertex u) {
        u.visit();                  // Do some unspecified thing to u
        u.visited = true;           // Mark the vertex u visited
        for (each vertex v such that (u, v) is an edge in E) {
            if (!v.visited) {
                dfs(v);
            }
        }
    }
}
```

## Running Time of DFS
DFS runs in O(|V| + |E|) time.

More specificly, O(|V| + |E|) time on adjacency lists, and O(|V|^2) time on adjacency matrices.   
(Because for adjacency matrices, O(|V| + |E|) = O(|V| + |V|^2) = O(|V|^2).)

**Why does DFS on an adjacency list run in O(|V| + |E|) time?**

The O(|V|) component comes up solely because we have to initialize all the
"visited" flags to false (or at least construct an array of flags) before we start.

The O(|E|) component is trickier.  Take a look at the "for" loop of the dfs() pseudocode above.  How many times does it iterate?  If the vertex u has outdegree d(u), then the loop iterates d(u) times.  Different vertices have different degrees.  Let the total degree D be the sum of the outdegrees of all the vertices in V.

    D = sum d(v).
        v in V

A call to dfs(u) takes O(d(u)) time, NOT counting the time for the recursive calls it makes to dfs().  A depth-first search never calls dfs() more than once on the same vertex, so the total running time of all the calls to dfs() is in O(D).  How large is D?

- If G is a directed graph, then D = |E|, the number of edges.
- If G is an undirected graph with no self-edges, then D = 2|E|, because each edge offers a path out of two vertices.
- If G is an undirected graph with one or more self-edges, then D < 2|E|.

In all three cases, the running time of depth-first search is in O(|E|), NOT
counting the time required to initialize the "visited" flags.

## Breadth-first Search
BFS 比 DFS 要复杂一些，需要引入一个 queue 来实现。

```java
public void bfs(Vertex u) {
    for (each vertex v in V) {          // O(|V|) time
        v.visited = false;
    }
    u.visit(null);                      // Do some unspecified thing to u
    u.visited = true;                   // Mark the vertex u visited
    q = new Queue();                    // New queue...
    q.enqueue(u);                       // ...initially containing u
    while (q is not empty) {            // With adjacency list, O(|E|) time
        v = q.dequeue();
        for (each vertex w such that (v, w) is an edge in E) {
            if (!w.visited) {
                w.visit(v);             // Do some unspecified thing to w
                w.visited = true;       // Mark the vertex w visited
                q.enqueue(w);
            }
        }
    }
}
```
BFS 的思路是先访问 depth 为 1 的节点，并依次压入队列，然后依次弹出队列，并访问 depth 为 2 的节点，并压入队列，以此循环下去。

注意，每次 visit 一个节点时，将边的 origin 作为参数传给了函数 visit()，这样我们可以在 visit() 函数内实现以下两个功能：

1. Find the distance of the vertex from the starting vertex.
2. Find the shortest path between the vertex with the starting vertex.

```java
public class Vertex {
    protected Vertex parent;
    protected int depth;
    protected boolean visited;
    
    public void visit (Vertex origin) {
        this.parent = origin;
        if (origin == null) {
            this.depth = 0;
        } else {
            this.depth = origin.depth + 1;
        }
    }
}
```

## Running Time of BFS
与 DFS 一样，BFS 的运行时间为：

O(|V| + |E|) time on adjacency lists；  
O(|V|^2) time on adjacency matrices.

**注意：对于 unweighted graph 的 *shortest path problem* 可以用 BFS 来实现，但是对于 weighted graph，则需要用 Dynamic Programming 或者其他更高阶的算法来实现。**