---
title: How Many Positions on a Rubik’s Cube?
subtitle:
date: '2018-09-03'
slug: rubiks-cube
categories:
  - Group
tags:
  - group
  - BFS
---

在 MIT 算法课中，讲 BFS 时提到可以用 BFS 来解决 2x2 的魔方还原问题，具体课程为：[Lecture 13: Breadth-First Search (BFS)](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/lecture-videos/lecture-13-breadth-first-search-bfs/)

课程中提到，2 阶魔方所有可能的状态个数为：

`$$\frac{\left ( 8! \cdot 3^8 \right )}{\left ( 24 \cdot 3 \right )}$$`

注意，二阶魔方尚且可以用 BFS 来实现，三阶及以上 BFS 则不再适合，运行时间是天文数字，而这就涉及一个全新的领域了，即 **Group**。本文暂不研究 group，留待后续进行相关研究。此处仅解决一个问题：How many positions on a Rubik's Cube?

# General Rules
在考虑所有可能的排列 （permutations） 及色相 （orientations） 的情况下，要注意以下两点：

1. 一些状态可能存在重复计算；
2. 一些状态可能永远无法实现。

# 2x2 Rubik's Cube
对于 2x2 的魔方而言，其共有 8 个角，每个角包含 3 种颜色，存在 3 种不同的色相，可以通过以下 2 种方式来思考：

### 方式一：先计算最大排列及色相分布，再消除重复及错误状态
1) 考虑最大可能的排列 `$8!$` 及色相分布 `$3^8$`，则状态个数为

`$$\left ( 8! \cdot 3^8 \right )$$`

2) 重复状态：由于 2x2 魔方没有中心轴，因此每个状态重复的次数为排列重复的次数 (8) 与色相重复的次数 (3) 之积，即

`$$8 \cdot 3 = 24$$`

3) 错误状态：由于无法在保持其他小方块状态不变的情况下旋转某个特定的小方块，即只有 7 个小方块的色相是相互独立的，因此只有 `$\frac{1}{3}$` 的状态是可以实现的。

综合以上的考虑，2x2 魔方所有可能的状态总数为

`$$\frac{\left ( 8! \cdot 3^8 \right )}{\left ( 24 \cdot 3 \right )}$$`

### 方式二：避免重复和错误状态直接计算最终结果
首先，因为 2x2 魔方没有中心轴，因此不失一般性，先任意选取一个角将其固定（位置、色相均固定），则剩下的 7 个角的排列为 `$7!$`，由于其中只有 6 个的色相是独立的，因此色相分布的种数为 `$3^6$`，因此所有可能的状态总数为

`$$7! \cdot 3^6$$`

* **2x2 魔方的一个思维误区：之前由于 3x3 魔方无法在保持其他小方块状态不变的情况下交换任意两个角或者任意两条边，便想当然地认为 2x2 的魔方也无法只交换两个角，因此一直无法理解上述计算公式。实际上 2x2 的魔方是不受这个条件限制的，因为 2x2 魔方每次移动相当于进行奇数次交换，而 3x3 魔方每次移动相当于进行偶数次交换，因此 3x3 魔方的所有角的排列及所有边的排列综合起来看其总的排列的 parity 应该呈现偶数性，而 2x2 魔方的排列呈奇数性或偶数性均可。事实上，2x2 魔方其中 6 个位置是正确的，只有另外相邻的 2 个位置发生了交换的情况是很常见的，这种状态称为 PLL，只是这两个小方块的色相并不是独立的。**

# 3x3 Rubik's Cube
以下内容多处摘自 [Ryan Heise](https://www.ryanheise.com/)

因为 3x3 魔方有一个中心轴，即 6 个面的中心点的相对位置是固定的，因此不失一般性，可以将中心轴固定于一个特定的状态开始考虑，由此出发所有可能的状态均不会出现重复。

注：此处采用“先计算最大排列及色相分布，再消除重复及错误状态”的方式进行计算，另一种方式与之类似，此处不再推导。

首先，对于 8 个角而言，所有可能状态的总数为

`$$\left ( 8! \cdot 3^8 \right )$$`

其次，对于 12 个边而言，所有可能的状态总数为

`$$\left ( 12! \cdot 2^{12} \right )$$`

因此，不考虑错误（不存在重复，不用考虑）的情况下，所有可能的状态总数为

`$$\left ( 8! \cdot 3^8 \cdot 12! \cdot 2^{12} \right )$$`

下边分析所有的错误状态：

## Rubik's Cube theory - 摘自 [Laws of the cube](https://www.ryanheise.com/cube/cube_laws.html)

> An interesting fact about Rubik's Cube is that if you take it apart and put it together randomly, there will be only a **1 in 12** chance of it actually being solvable by legal moves (that is, without taking the cube apart again).
> 
> ## 1. Only half of the permutations are reachable
> 
> <img src="https://i.gyazo.com/937eef69ee4b883450ff6732cd948334.png" height="100%" width="100"> 
> 
> <center>An impossible state</center>
> 
> As it turns out, every cube state reachable by legal moves can always be represented by an even number of swaps, and at the same time cannot be represented by an odd number of swaps (the two are mutually exclusive). Since the above cube has an odd number of swaps ("one" swap), this state cannot be reached.
> 
> To understand why this is so, we need to realise that each legal move always performs the equivalent of an even number of swaps. No matter how many moves you perform, the number of accumulated swaps will therefore always remain even.
> 
> For example, consider the turning of one face by 90 degrees:
> 
    C1  E1  C2       C4  E4  C1
    E4      E2  -->  E3      E1
    C4  E3  C3       C3  E2  C2
> 
> The new corner state can be obtained via 3 swaps (swap C1/C4, swap C1/C3, swap C1/C2). Similarly, the new edge state can be obtained via 3 swaps. All together, this is 6 swaps which is even. Therefore, no matter how many moves you perform, always an even number of swaps will have been performed.
> 
> Since exactly half of the conceivable permutations are even and the other half are odd, only half of the cube's permutations (ignoring orientation) are reachable by legal moves.
> 
> ## 2. Only half of the edge orientations are reachable
> 
> <img src="https://i.gyazo.com/c653790d609f7371454d384f40bd4312.png" height="100%" width="100"> 
> 
> <center>An impossible state</center>
> 
> It also turns out that each legal move on Rubik's Cube always flips an even number of edges, and so the above state would be impossible to reach via legal moves. To establish this, it is necessary to decide on a frame of reference for correct edge orientation, regardless of where an edge is positioned on the cube. The most common frame of reference is to say that an edge in the wrong position has correct orientation if, when it is moved to its correct position using only the left, right, top and bottom faces, it would have correct orientation. Using this frame of reference, it is easy to see that any move on the left, right, top and bottom faces will always flip zero edges, which is an even number. The only remaining faces are the front and back faces. In both of these cases, a 90 degree move will flip all 4 edges, which is again an even number of flips. Therefore, it is never possible, using only legal moves, to flip an odd number of edges.
> 
> ## 3. Only one third of the corner orientations are reachable
> 
> <img src="https://i.gyazo.com/8592d6cc50587f7774efc392d0d746e1.png" height="100%" width="100"> 
> 
> <center>An impossible state</center>
> With corner orientations, things are slightly more difficult to account for, since each corner piece has three possible orientations, not two. Let's say that a corner has orientation '0' if it is twisted the correct way, it has an orientation of '1' if it is twisted clockwise, and it has an orientation of '2' if it is twisted even one step more clockwise (this is the same as just twisting anti-clockwise). It turns out that each legal move on Rubik's Cube always twists the corners in such a way that the sum of all of their orientations is exactly divisible by 3, and so the above state would by impossible to reach since its total corner orientation is 1 (which is not divisible by 3).
> Once again, to establish this, it is necessary to decide on a frame of reference for correct corner orientation. Notice that every corner either belongs to the top or bottom and therefore each corner has one of its stickers with either the colour of the top face or the colour of the bottom face. The most common frame of reference for correct corner orientation is to say that a corner has correct orientation if its top/bottom sticker is facing up or down. If it is facing in any other direction, then this corner is not correctly oriented. Using this frame of reference, it is easy to see that any twist of the top and bottom faces will not change the orientation of the corners, and therefore the total orientation will remain exactly divisible by 3. For any of the 4 sides, a 90 degree turn will twist two corners by orientation 1 and the other two corners by orientation 2. The total change in orientation is 1+1+2+2 = 6, which is divisible by 3. Therefore, no matter how many legal moves you make in a row, the corner orientation will always remain divisible by 3.
> 
> ## Overall
> 
> Combining these laws, only **1/12 (1/2 * 1/2 * 1/3**) of the conceivable cube states are reachable by legal moves.

关于原则 1，可以用 [Parity of a permutation](https://en.wikipedia.org/wiki/Parity_of_a_permutation) 来分析：

## Rubik's Cube theory - 摘自 [Parity](https://www.ryanheise.com/cube/parity.html)

> The parity of a permutation refers to whether that [permutation](https://www.ryanheise.com/cube/basic_definitions.html) is even or odd. An even permutation is one that can be represented by an even number of swaps while an odd permutation is one that can be represented by an odd number of swaps.
> 
> When considering the permutation of all edges and corners together, the overall parity must be even, as dictated by [laws of the cube](https://www.ryanheise.com/cube/cube_laws.html). However, when considering only edges or corners alone, it is possible for their parity to be either even or odd. To obey the laws of the cube, if the edge parity is even then the corner parity must also be even, and if the edge parity is odd then the corner parity must also be odd.

综上：3x3 魔方所有可能的状态总数为

`$$\frac{\left ( 8! \cdot 3^8 \cdot 12! \cdot 2^12\right )}{\left ( 2 \cdot 2 \cdot 3 \right )}$$`

# Reference
[Ryan Heise: Laws of the cube](https://www.ryanheise.com/cube/cube_laws.html)  
[Ryan Heise: Parity](https://www.ryanheise.com/cube/parity.html)  
[Wikipedia: Pocket Cube - Permutations](https://en.wikipedia.org/wiki/Pocket_Cube#Permutations)  
[Wikipedia: Page semi-protected
Rubik's Cube - Permutations](https://en.wikipedia.org/wiki/Rubik%27s_Cube#Permutations)  
[Wikipedia: Parity of a permutation](https://en.wikipedia.org/wiki/Parity_of_a_permutation)  
[MIT: The Mathematics of the Rubik’s Cube](http://web.mit.edu/sp.268/www/rubik.pdf)  