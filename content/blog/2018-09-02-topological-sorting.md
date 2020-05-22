---
title: Topological Sorting
subtitle: 
date: '2018-09-02'
slug: topological-sorting
categories:
  - Algorithms
  - Graph
  - Dynamic Programming
tags:
  - topological sorting
---

Topological Sorting 是针对有向无环图 (DAG, Directed Acyclic Graph 而言的)。每个 DAG 都存在 topological sorting，但是这种 sorting 并不一定唯一，可能存在多个不同的 topological sorting。

Topological Sorting 算法与 [Depth-first Search
](https://kaizhang.me/note/2018/09/graph-traversals/) 算法非常类似。DFS 算法是从一个起点出发，然后递归地去访问其 adjacent vertices，每当第一次访问一个节点时，对其进行相关操作，最终达到对所有节点的遍历。而 topological sorting 与 DFS 不同，其不是在第一次访问一个节点时对其进行操作，而是在每次访问该节点后，在递归退出时将该节点压入栈，当遍历所有节点之后，栈中元素依次出栈的顺序就是 topological sorting。其原理在于：每次压入栈的元素，其所有 adjacent vertices (and their adjacent vertices and so on) 都已经在栈中了，因此栈外的元素均在其 topological 的前边位置，因此可以放心地把该元素压入栈。

下边给出这两个算法进行对比。

Topological Sorting:

```java
public class Graph {
    // Before calling topologicalSort(), set every "visited" flag to false; takes O(|V|) time
    public void topologicalSort() {
        Stack stack = new Stack();
        for (each vertex v in V) {
            if (!v.visited) {
                topologicalSort(v, stack);
            }
        }
        
        POP ALL ELEMENT FROM THE STACK WILL GET TOPOLOGICAL SORTING
    }
    
    public void topologicalSort(Vertex u, Stack stack) {
        u.visited = true;           	// Mark the vertex u visited
        for (each vertex v such that (u, v) is an edge in E) {
            if (!v.visited) {
                topologicalSort(v, stack);
            }
        }
        u.visit();                   // Do some unspecified thing to u AFTER RECURSIVE CALLS
        stack.push(v);
    }
}
```

DFS:

```java
public class Graph {
    // Before calling dfs(), set every "visited" flag to false; takes O(|V|) time
    public void dfs(Vertex u) {
        u.visit();                   // Do some unspecified thing to u BEFORE RECURSIVE CALLS
        u.visited = true;           	// Mark the vertex u visited
        for (each vertex v such that (u, v) is an edge in E) {
            if (!v.visited) {
                dfs(v);
            }
        }
    }
}
```

### Running time

时间复杂度与 DFS 相同：

O(|V| + |E|)

More specificly, O(|V| + |E|) time on adjacency lists, and O(|V|^2) time on adjacency matrices.  
(Because for adjacency matrices, O(|V| + |E|) = O(|V| + |V|^2) = O(|V|^2).)