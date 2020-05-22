---
title: Dynamic Programming Concepts
subtitle: 动态规划中的相关概念
date: '2018-11-12'
slug: dynamic-programming-concepts
categories:
  - Algorithms
tags:
  - dynamic programming
---

注：本文摘自维基百科

# 1. [Dynamic Programming](https://en.wikipedia.org/wiki/Dynamic_programming)

Dynamic programming is both a [mathematical optimization](https://en.wikipedia.org/wiki/Mathematical_optimization) method and a computer programming method. **In both contexts it refers to simplifying a complicated problem by breaking it down into simpler sub-problems in a recursive manner.** *While some decision problems cannot be taken apart this way, decisions that span several points in time do often break apart recursively.* Likewise, in computer science, if a problem can be solved optimally by breaking it into sub-problems and then recursively finding the optimal solutions to the sub-problems, then it is said to have [optimal substructure](https://en.wikipedia.org/wiki/Optimal_substructure).

If sub-problems can be nested recursively inside larger problems, so that dynamic programming methods are applicable, then there is a relation between the value of the larger problem and the values of the sub-problems. In the optimization literature this relationship is called the [Bellman equation](https://en.wikipedia.org/wiki/Bellman_equation).

# 2. [Optimal substructure](https://en.wikipedia.org/wiki/Optimal_substructure)

In computer science, a problem is said to have optimal substructure if an optimal solution can be constructed from optimal solutions of its subproblems. **This property is used to determine the usefulness of dynamic programming and greedy algorithms for a problem.**

Typically, a [greedy algorithm](https://en.wikipedia.org/wiki/Greedy_algorithm) is used to solve a problem with optimal substructure *if it can be proven by induction that this is optimal at each step*. Otherwise, provided the problem exhibits [overlapping subproblems](https://en.wikipedia.org/wiki/Overlapping_subproblems) as well, dynamic programming is used. If there are no appropriate greedy algorithms and the problem fails to exhibit overlapping subproblems, often a lengthy but straightforward search of the solution space is the best alternative.

In the application of [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming) to [mathematical optimization](https://en.wikipedia.org/wiki/Mathematical_optimization), Richard Bellman's [Principle of Optimality](https://en.wikipedia.org/wiki/Bellman_equation) is based on the idea that in order to solve a dynamic optimization problem from some starting period *t* to some ending period *T*, one implicitly has to solve subproblems starting from later dates s, where *t<s<T*. This is an example of optimal substructure. The Principle of Optimality is used to derive the [Bellman equation](https://en.wikipedia.org/wiki/Bellman_equation), which shows how the value of the problem starting from *t* is related to the value of the problem starting from *s*.

注：Optimal substructure 是用来判断动态规划和贪婪算法有效性的一个指标。

贪婪算法有效性的条件为：

1. Optimal substructure
2. 每一步均为最优解

动态规划有效性的条件为：

1. Optimal substructure
2. Overlapping subproblems

# 3. [Greedy algorithm](https://en.wikipedia.org/wiki/Greedy_algorithm)

A greedy algorithm is an [algorithmic paradigm](https://en.wikipedia.org/wiki/Algorithmic_paradigm) that follows the [problem solving](https://en.wikipedia.org/wiki/Problem_solving) [heuristic](https://en.wikipedia.org/wiki/Heuristic_(computer_science)) of making the locally optimal choice at each stage with the intent of finding a global optimum. In many problems, a greedy strategy does not usually produce an optimal solution, but nonetheless a greedy heuristic may yield locally optimal solutions that approximate a globally optimal solution in a reasonable amount of time.

For example, a greedy strategy for the [traveling salesman problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem) (which is of a high [computational complexity](https://en.wikipedia.org/wiki/Computational_complexity)) is the following heuristic: "At each step of the journey, visit the nearest unvisited city." This heuristic does not intend to find a best solution, but it terminates in a reasonable number of steps; finding an optimal solution to such a complex problem typically requires unreasonably many steps. In [mathematical optimization](https://en.wikipedia.org/wiki/Mathematical_optimization), greedy algorithms optimally solve [combinatorial problems](https://en.wikipedia.org/wiki/Combinatorial_optimization) having the properties of [matroids](https://en.wikipedia.org/wiki/Matroid), and give constant-factor approximations to optimization problems with [submodular structure](https://en.wikipedia.org/wiki/Submodular_set_function).

# 4. [Overlapping subproblems](https://en.wikipedia.org/wiki/Overlapping_subproblems)

In computer science, a problem is said to have overlapping subproblems if the problem can be broken down into subproblems which are reused several times or a recursive algorithm for the problem solves the same subproblem over and over rather than always generating new subproblems.

For example, the problem of computing the [Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number) exhibits overlapping subproblems. The problem of computing the *n* th Fibonacci number *F* (*n*), can be broken down into the subproblems of computing *F* (*n* − 1) and *F* (*n* − 2), and then adding the two. The subproblem of computing *F* (*n* − 1) can itself be broken down into a subproblem that involves computing *F* (*n* − 2). Therefore, the computation of *F* (*n* − 2) is reused, and the Fibonacci sequence thus exhibits overlapping subproblems.

A naive recursive approach to such a problem generally fails due to an [exponential complexity](https://en.wikipedia.org/wiki/Time_complexity#Exponential_time). If the problem also shares an [optimal substructure](https://en.wikipedia.org/wiki/Optimal_substructure) property, [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming) is a good way to work it out.

# 5. [Bellman's Principle of Optimality](https://en.wikipedia.org/wiki/Bellman_equation)

The dynamic programming method breaks this decision problem into smaller subproblems. Richard Bellman's principle of optimality describes how to do this:

Principle of Optimality: An optimal policy has the property that whatever the initial state and initial decision are, the remaining decisions must constitute an optimal policy with regard to the state resulting from the first decision.

# 6. [The Bellman equation](https://en.wikipedia.org/wiki/Bellman_equation)
