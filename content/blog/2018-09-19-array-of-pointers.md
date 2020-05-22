---
title: 指针的数组 VS 数组的指针
subtitle: 
date: '2018-09-19'
slug: array-of-pointers
categories:
  - C
  - Arrays
  - Pointers
tags:
  - arrays
  - pointers
toc: true
---

接上文 [数组和指针三大规则解析](https://kaizhang.me/note/2018/09/rules-about-arrays-and-pointers/)

在上文中关于指针的数组及数组的指针的类型名作了简单地辨析。具体如下：

指针的数据类型由指针符号 `*` 或 **带括号的指针符号 `(*)`** 加上其**指向的目标的类型**组成。

在类型名中，指向基本数据类型的指针的类型为基本数据类型加上指针符号 `*`。但是当涉及到指针的数组和数组的指针，即类型名中同时出现数组符号 `[]` 和指针符号 `*` 时，需要用带括号的指针符号 `(*)` 来表示类型名为相应的指针类型。

```C
int a1;				// type of a1: 'int'
int a2[10];			// type of a2: 'int [10]'
int * p1;			// typre of p1: 'int *'
int * p2[10];			// type of p2: 'int *[4]'
				// 即元素类型为 'int *' 的指针，长度为 4 的数组
int (* p3)[10];			// type of p3: 'int (*)[10]'
				// 即指向类型为 'int [10]' 的数组的指针
```

下面具体展开二者的辨析。

# 1 指针的数组
此处的指针只包含“指向基本数据类型的指针”，其他的情况此处不予考虑。

## 1.1 指向基本数据类型的指针
首先，指向基本数据类型的指针的类型名为相应的基本数据类型加上指针符号 `*`。如 `int * ptr` 中 `ptr` 的数据类型为 `int *`。

![Point of Basic Data Type](https://i.gyazo.com/687312ced6cd538b0b9da4415263d655.png)

## 1.2 “指向基本数据类型的指针”的数组
元素为“指向基本数据类型的指针”的数组的定义为“指针类型 + 数组名 + [size]”，如：

```C
double * arr[4];
```

其中，`arr` 的数据类型为 'double *[4]'。

![Array of Pointers](https://i.gyazo.com/6725db3c545557d743774bcd82633092.png)

有一点需要注意的是，虽然元素的类型为“指向基本数据类型的指针”，但是这个基本数据包含两种情况：

1. 基本的变量，如 `double a = 3.14`；
2. 基本数据的数组名在表达式中退化后得到的指针所指向的数组的首元素。

前者相当于实现了一个基本数据的一维数组，后者相当于实现了一个基本数据的二维数组，需要单独分析。

### 1.2.1 指针指向基本数据变量

![Array of Pointers (to basic type variable)](https://i.gyazo.com/6d535d8800ce101496f58247b4b54363.png)

### 1.2.2 指针指向基本数据的数组的首元素
首先，C 语言里边的“多维数组”实际上是“数组的数组”，即外层数组的元素为内层数组。

![Multidimensional Array](https://i.gyazo.com/1f318ddab5ffbd0b54045d0c545d5c31.png)

除了多维数组，我们可以考虑用指针的数组来实现存储多维数据的结构，即直接将基本数据的一维数组名赋值给指针的数组的元素，由于此时基本数据的数组名在表达式中退化为了指向其首元素的指针，因此指针的数组的每个元素便指向对应的一维数组的首元素。

![Array of Pointers (to 1st element of arrays of basic type)](https://i.gyazo.com/7c8dd62c848cb3e05acfae58616e88c7.png)

另外，也可以通过取多维数组的倒数第二维元素对应的一维数组，并将其赋值给指向基本数据的指针的数组。因为在表达式中，倒数第二维的元素对应的一维数组名会退化为一维数组的首元素的指针。

```C
double myMatrix[4][3];
double * arr[4] = {myMatrix[0], myMatrix[1], myMatrix[2], myMatrix[3]};
```

**注意：这两者的区别在于，指针的数组不要求指针对应的数组在内存中的连续性，即 `row0`，`row1`，`row2`，`row3`在内存中可以是不连续的，但是多维数组在内存中是连续的。**

# 2 数组的指针
数组的指针与指针的数组在形式上很容易混淆，下面给出对比。

| | 指针的数组 | 数组的指针 |
| :--: | :--: | :--: |
| 定义方式 | double * arr[4] | double (* ptr) [4] |
| 数据类型 | double *[4] | double (*)[4] |

![Pointer of array](https://i.gyazo.com/f8393254687acf4b5a5704b3b69b85af.png)

# 3 代码验证
代码如下：

```C
#include <stdio.h>
#include <stdlib.h>

int main(void) {
  // pointer of data with basic types
  int x = 5;
  int *ptr = &x;
  printf("5 should be: %d\n\n", *ptr);

  // array of pointers (to basic data variables)
  double a = 1.1;
  double b = 1.2;
  double c = 1.3;
  double * arr_pointer1[3] = {&a, &b, &c};
  printf("The number should be 1.1, 1.2, 1.3: \n");
  for (size_t i = 0; i < 3; i++) {
    printf("%lf, ", *(arr_pointer1[i]));
  }
  printf("\n\n");

  // array of pointers (to the 1st element of arrays whose elements are basic type)
  double row0[3] = {0, 1, 2};
  double row1[3] = {3, 4, 5};
  double row2[3] = {6, 7, 8};
  double row3[3] = {9, 10, 11};
  double * arr_pointer2[4] = {row0, row1, row2, row3};
  printf("The number should be 0 ~ 11: \n");
  for (size_t i = 0; i < 4; i++) {
    for (size_t j = 0; j < 3; j++) {
      printf("%lf, ", arr_pointer2[i][j]);
    }
  }
  printf("\n\n");

  // another array of pointers (to the 1st element of arrays whose elements are basic type)
  double myMatrix[4][3] = {{0, 1, 2}, {3, 4, 5}, {6, 7, 8}, {9, 10, 11}};
  printf("The number should be 0 ~ 11: \n");
  for (size_t i = 0; i < 4; i++) {
    for (size_t j = 0; j < 3; j++) {
      printf("%lf, ", myMatrix[i][j]);
    }
  }
  printf("\n\n");

  // pointer of array
  double array[4] = {1, 2, 3, 4};
  double (* ptr1) [4] = &array;
  printf("The number should be 1 ~ 4: \n");
  for (size_t i = 0; i < 4; i++) {
    printf("%lf, ", (*ptr1)[i]);
  }
  printf("\n");

  return EXIT_SUCCESS;
}

```

输出：

```bash
➜  array gcc -o array2 -std=gnu99 -pedantic -Wall -Werror array2.c
➜  array ./array2
5 should be: 5

The number should be 1.1, 1.2, 1.3:
1.100000, 1.200000, 1.300000,

The number should be 0 ~ 11:
0.000000, 1.000000, 2.000000, 3.000000, 4.000000, 5.000000, 6.000000, 7.000000, 8.000000, 9.000000, 10.000000, 11.000000,

The number should be 0 ~ 11:
0.000000, 1.000000, 2.000000, 3.000000, 4.000000, 5.000000, 6.000000, 7.000000, 8.000000, 9.000000, 10.000000, 11.000000,

The number should be 1 ~ 4:
1.000000, 2.000000, 3.000000, 4.000000,
➜  array
```


