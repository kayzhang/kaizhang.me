---
title: 指针的数组 VS 多维数组 
subtitle: 及二者作为函数参数的区别
date: '2018-09-19'
slug: arrays-of-pointers-and-multidimensional-arrays
categories:
  - C
  - Arrays
  - Pointers
tags:
  - arrays
  - pointers
  - multidimensional arrays
toc: true
---

指针的数组和多维数组（即数组的数组）在结构上有一定的相似性，但是二者又有着较大的区别。

# 1 数据类型不同导致了访问方式的不同
可以参考 [数组和指针是如何访问的](https://kaizhang.me/note/2018/09/expert-c-progarmming-c4-arrays-and-pointers/#4-3-1-数组和指针是如何访问的)

首先，给定下面的设定：

```C
int row0[3] = {0, 1, 2};
int row1[3] = {3, 4, 5};
int row2[3] = {6, 7, 8};
int row3[3] = {9, 10, 11};
int * array_of_pointers[4] = {row0, row1, row2, row3};

int multi_array[4][3] = {
{0, 1, 2},
{3, 4, 5},
{6, 7, 8},
{9, 10, 11}};
```

在 [数组和指针是如何访问的](https://kaizhang.me/note/2018/09/expert-c-progarmming-c4-arrays-and-pointers/#4-3-1-数组和指针是如何访问的) 中我们讨论了指针和数组的访问方式的不同，即对指针的下标访问要比对数组的下标访问多一次对内存的读取操作。

而指针的数组和多维数组（即数组的数组）相当于是两层结构，第一层二者均为对数组的下标访问，均为 2 个步骤。但是第二层却不一样，指针的数组的第二层属于对指针的下标访问，要进行 3 个步骤，而多维数组的第二层还是一个数组，仍然需要 2 个步骤。

因此，相较于多维数组，指针的数组的效率要低一些。

# 2 类型的不同决定了相应函数形参声明不同
当一个数组名发生向其首元素指针的退化时，其类型变为其首元素类型的指针。

如 `int a[10];` 中，当 `a` 发生退化时类型变为 `int *`。因此，可以把数组名 `a` 作为实参传给形参声明为 `int * a` 的函数。

```C
void test(int *) {};
int a[10];
test(a);
```

## 2.1 指针的数组作为形参
指针的数组发生退化则会变成“指针的指针”类型，即 `int * arr[4];` 中 `arr` 发生退化时类型变为 `int **`。因此，形参列表可以写为 `int ** arr` 或 `int * arr[]`。在函数内部，因为是指向 `int` 的指针的指针，因此第一层的步长为一个指针变量的长度（64 位系统为 8-bits），用于切换指针；第二层的步长为 `int` 变量的长度，用于切换最内层的元素。

代码：

```C
#include <stdio.h>
#include <stdlib.h>

void test(int ** arr, size_t first, size_t second) {
  printf("The number should be 1 ~ 11: ");
  for (size_t i = 0; i < first; i++) {
    for (size_t j = 0; j < second; j++) {
      printf("%d, ", arr[i][j]);
    }
  }
  printf("\n");
}

int main(void) {
  int row0[3] = {0, 1, 2};
  int row1[3] = {3, 4, 5};
  int row2[3] = {6, 7, 8};
  int row3[3] = {9, 10, 11};
  int *arr[4] = {row0, row1, row2, row3};
  test(arr, 4, 3);
  
  return EXIT_SUCCESS;
}
```

输出：

```bash
➜  array gcc -o array3 -std=gnu99 -pedantic -Wall -Werror array3.c
➜  array ./array3
The number should be 0 ~ 11:
 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,
➜  array
```

## 2.2 多维数组作为函数的形参
多维数组发生退化时会变成“数组的指针”类型，即 `int arr[4][3];` 中 `arr` 发生退化时类型变为 `int (*)[3]`。因此，形参列表可以写为数组的指针 `int (* arr)[3]` 或多维数组 `int arr[][3]`。在函数内部，因为是指向数组的指针，因此第一层的步长为内部数组的长度（即 3 个 `int` 的长度），用于切换内部数组；第二层的步长为一个 `int` 的长度，用于切换最内层元素。

代码：

```C
#include <stdio.h>
#include <stdlib.h>

void test(int (* arr)[3], size_t first, size_t second) {
  printf("The number should be 0 ~ 11:\n ");
  for (size_t i = 0; i < first; i++) {
    for (size_t j = 0; j < second; j++) {
      printf("%d, ", arr[i][j]);
    }
  }
  printf("\n");
}

int main(void) {
  int arr[4][3] = {{0, 1, 2},
		   {3, 4, 5},
		   {6, 7, 8},
		   {9, 10, 11}};
  test(arr, 4, 3);
  
  return EXIT_SUCCESS;
}
```

输出：

```bash
➜  array gcc -o array4 -std=gnu99 -pedantic -Wall -Werror array4.c
➜  array ./array4
The number should be 0 ~ 11:
 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11,
➜  array
```

# 3 访问方式的不同决定了二者的不兼容
参见 [使声明与定义相匹配](https://kaizhang.me/note/2018/09/expert-c-progarmming-c4-arrays-and-pointers//#定义为数组-但声明为指针-并以数组方式引用)

当主函数中定义为数组的数组 `int arr[4][3]`，而 `void test(int ** arr, size_t first, size_t second) {};` 声明为指针的指针（即对应的外部数据为指针的数组）时，如果调用 `test(arr, 4, 3)`，则会发生 compiler error，原因在于二者数据类型不兼容。参数的类型为 `int ** arr`，但传入的参数类型为 `int [4][3]` （实际上为退化后的 `int (*)[3]`）。

代码：

```C
#include <stdio.h>
#include <stdlib.h>

void test(int ** arr, size_t first, size_t second) {
  printf("The number should be 0 ~ 11:\n ");
  for (size_t i = 0; i < first; i++) {
    for (size_t j = 0; j < second; j++) {
      printf("%d, ", arr[i][j]);
    }
  }
  printf("\n");
}

int main(void) {
  int arr[4][3] = {{0, 1, 2},
		   {3, 4, 5},
		   {6, 7, 8},
		   {9, 10, 11}};
  test(arr, 4, 3);
  
  return EXIT_SUCCESS;
}
```

输出：

```bash
➜  array gcc -o array5 -std=gnu99 -pedantic -Wall -Werror array5.c
array5.c:19:8: error: incompatible pointer types passing 'int [4][3]' to parameter of type 'int **'
      [-Werror,-Wincompatible-pointer-types]
  test(arr, 4, 3);
       ^~~
array5.c:4:18: note: passing argument to parameter 'arr' here
void test(int ** arr, size_t first, size_t second) {
                 ^
1 error generated.
➜  array
```

此时可以通过 cast 强制将 `int [4][3]` 的类型转换成 `int **`：

```C
test((int **)arr, 4, 3);
```

程序可以编译成功，但是此时会出现和 [使声明与定义相匹配](https://kaizhang.me/note/2018/09/expert-c-progarmming-c4-arrays-and-pointers//#定义为数组-但声明为指针-并以数组方式引用) 相同的问题：

**在子函数内部将参数声明为 `int **`，那么不管实参在主函数中的定义为什么类型，都会以 `int **` 的下标访问方式来访问参数，即第一层 3 步指针访问方式，第二层也为 3 步指针访问方式。但是如果传入的实参之前的定义为多维数组，那么实际其实际传入的参数类型为 `int (*)[3]`，此时其正确的访问方式应该为第一层 3 步指针访问方式，第二层 2 步数组访问方式。但是编译器只根据形参的声明对其进行 `int **` 方式的访问，导致在第二层取到数组内元素的实际值之后，将其视为第二层的指针的内容，即指针指向元素的地址，继续去访问这个地址（地址值为数组内元素的值），便会出现不确定的行为或者 segmentation fault!**

因此，要时刻保持二者数据类型的匹配：

1. 定义为指针的数组，则声明也应该是指针的数组（更确切地说是指针的指针）；
2. 定义为多维数组，则声明也应还是多维数组（更确切地说是数组的指针）。
