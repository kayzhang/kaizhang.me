---
title: 数组和指针三大规则解析
subtitle: 
date: '2018-09-16'
slug: rules-about-arrays-and-pointers
categories:
  - C
  - Expert C Programming
  - Arrays
  - Pointers
tags:
  - arrays
  - pointers
toc: true
---

本文继续上文 [《Expert C Programming》 第 9 章：再论数组](https://kaizhang.me/note/2018/09/expert-c-progarmming-c4-more-about-arrays/) 的讨论，通过一些具体的实例来进一步阐述《Expert C Programming》第 9 章中提到的关于数组和指针的三个规则。

# 1 数组和指针三大规则
这三条规则用于辨析数组和指针，阐释了二者在哪些情况下相同，哪些情况下不同。

* 规则 1. **表达式**中的数组名（用于区别声明中的数组名）被编译器当作一个指向该数组第一个元素的指针（具体释义见 ANSI C 标准第 6.2.2.1 节）。（例外情况在下面讨论）

* 规则 2. 数组的下标引用总是与指针的偏移量引用相同（具体释义见 ANSI C 标准第 6.3.2.1 节）。

* 规则 3. 在函数参数的声明中，数组名被编译器当作指向该数组第一个元素的指针（具体释义见 ANSI C 标准第 6.7.1 节）。

注：规则 1 存在几个极为少见的例外，即需要将数组作为一个整体来考虑的情况。在下列情况下，对数组的引用不能用指向该数组第一个元素的指针来代替：

> * 数组作为 `sizeof` 的操作数；  
* 使用 `&` 操作符取数组的地址（得到的是指向整个数组的指针）；
* 数组是一个另一个数组的字符串（或宽字符串）常量（string literal）初始值。  

具体的 C 参考文档中是这么描述规则 1 的：

> Array to pointer conversion
> 
> Any lvalue expression of array type, when used in any context other than
> 
> * as the operand of the address-of operator
* as the operand of sizeof
* as the string literal used for array initialization
* as the operand of _Alignof
(since C11)
> 
> undergoes an implicit conversion to the pointer to its first element. The result is not an lvalue.

# 2 关于数组和指针操作的一些总的原则
下面指出与数组和指针的操作相关的一些原则，这些原则其实是跟 C 语言其他数据类型的操作原则一致的，只是在处理数组和指针的时候这些概念比较容易混淆。

首先，所有的数据结构都有其对应的类型，编译器根据数据的类型来采取相应的处理方式，这是一个非常关键的概念。（注：此处关于类型的表述可能不是很准确，有待后续订正。）

## 2.1 数组和指针的数据类型
### 2.1.1 数组的数据类型
数组的数据类型由两部分组成，即：

1. 数组最内层元素的类型；
2. 数组的 size 信息。

比如：

```C
int arr1[9];			// the type of arr1 is 'int [9]'
int arr2[10];			// the type of arr2 is 'int [10]'
float flt1[9];			// the type of flt1 is 'float [9]'
int arr3[10][20];		// the type of arr3 is 'int [10][20]'
arr3[2]				// the type of arr3[2] is 'int [20]`
```

注意到，数组的数据类型包含最内层元素类型及数组大小这 2 个信息，只有当这两个信息均相同时，其数据类型才相同。数组的操作要严格按照其数据类型进行。

### 2.1.2 指针的数据类型
指针的数据类型由指针符号 `*` 或 **带括号的指针符号 `(*)`** 加上其**指向的目标的类型**组成。

注：在类型名中，指向基本数据类型的指针的类型为基本数据类型加上指针符号 `*`。但是当涉及到数组的指针和指针的数组，即类型名中同时出现数组符号 `[]` 和指针符号 `*` 时，需要用带括号的指针符号 `(*)` 来表示类型名为相应的指针类型。

```C
int a1;				// type of a1: 'int'
int a2[10];			// type of a2: 'int [10]'
int * p1;			// typre of p1: 'int *'
int * p2[10];			// type of p2: 'int *[4]'
				// 即元素类型为 'int *' 的指针，长度为 4 的数组
int (* p3)[10];			// type of p3: 'int (*)[10]'
				// 即指向类型为 'int [10]' 的数组的指针
```

此处仅简单举例，具体分析，参见 [指针的数组 VS 数组的指针](https://kaizhang.me/note/2018/09/array-of-pointers/)

更多关于指针数据类型的例子如下：

```C
int * ptr1;			// the type of ptr1 is 'int *'
float * flt1; 			// the type of flt1 is 'float *'

int arr1[9];
int arr2[10];
float flt1[9];
int arr3[10][20];

&arr1				// the type of &arr1 is 'int (*)[9]'
&arr2				// the type of &arr2 is 'int (*)[10]'
&flt1				// the type of &flt1 is 'float (*)[9]'
&arr3				// the type of &arr3 is 'int (*)[10][20]'
&arr3[1]			// the type of &arr3[1] (which is &(arr3[1])) is 'int (*)[20]'

// 下面的式子的 type 为相应的数组
(&arr1)[1]			// the type of (&arr1)[1] is 'int [9]'
				// 即指针 (&arr1 + 1) 所指向的目标的类型，等同于 *(&arr1 + 1)
(&arr3)[1]			// the type of (&arr3)[1] is 'int [10][20]'
				// 即指针 (&arr3 + 1) 所指向的目标的类型，等同于 *(&arr3 + 1)
(&arr3)[1][1]			// the type of (&arr3)[1][1] is 'int [20]' 
				// 此处用到了数组操作，此处 (&arr3)[1] 发生了向其首元素
				// （类型为 'int [20]'）的指针的退化，等同于 *((arr3)[1] + 1
```

## 2.2 数组和指针的相关操作
在搞清楚数组和指针的数据类型之后，就可以更加清晰地认识其相关操作的规则。

### 2.2.1 数组操作的相关规则
#### 2.2.1.1 作为取地址操作符 `&` 的操作数
当数组名作为 `&` 的操作数（如 `&arr`）时，数组本身的类型保持不变，并不会发生向其首元素指针的退化，表达式 `&arr` 得到的结果是指向数组 `arr` 的指针，其数据类型即为 `(*)` 加上数组 `a` 的类型。

```C
int arr1[9];
int arr2[10];
float flt1[9];
int arr3[10][20];

&arr1				// the type of &arr1 is 'int (*)[9]'
&arr2				// the type of &arr2 is 'int (*)[10]'
&flt1				// the type of &flt1 is 'float (*)[9]'
&arr3				// the type of &arr3 is 'int (*)[10][20]'
&arr3[1]			// the type of &arr3[1] (which is &(arr3[1])) is 'int (*)[20]'
```

由于 `&arr` 得到的是一个指针，因此对其的操作遵循指针操作的规则。

#### 2.2.1.2 作为 `sizeof` 操作符的操作数
当数组名作为 `sizeof` 的操作数（如 `sizeof arr`）时，数组本身的类型保持不变，并不会发生向其首元素指针的退化，表达式 `sizeof arr` 得到的结果为该数组的类型所对应的字节数，即整个数组在内存中占据的空间，亦即所有最内层元素的字节数之和。

```C
int arr1[9];
int arr2[10];
float flt1[9];
int arr3[10][20];

sizeof arr1			// 36
sizeof arr2			// 40
sizeof flt1			// 36
sizeof arr3			// 800
size of arr3[5]			// 80
sizeof (&arr1)[1]		// 36
sizeof (&arr3)[1]		// 800
sizeof (&arr3)[1][1]		// 80
```

由此，可以通过以下方式获得数组的长度：

```C
// 获得数组长度的方法一
int arr[10];
size_t size1 = (sizeof arr) / (sizeof arr[0]);
```

#### 2.2.1.3 出现在表达式或函数的形参列表中
此处“表达式”指的是规则 1 中的除了特殊情况之外的情况，此时，数组名退化为指向该数组的首元素的指针，即此时指针的类型为 `*` 或 `(*)`加上数组首元素的类型。此时相当于将数组解绑了，转换为指针之前数组是一个整体，而转换为指针之后，指针指向的是数组的下级元素，不再是一个整体了。

由于此时数组退化为指针，所以对其的所有操作均要按照指针的操作规则进行。值得注意的是，此时的指针(即退化后的 `arr`)的类型为指向数组下级元素的指针，而不是指向整个数组的指针（即 `&arr`），这两个指针的类型是不同的，因此相关操作的结果也不同。

### 2.2.2 指针操作的相关规则
#### 2.2.2.1 步长由其所指向元素的类型决定
在对指针进行操作时，最关键的就是指针移动的步长是由“指针的类型”严格决定的！上边已经讨论过，指针的类型由 `*` 或 `(*)`加上其所指向的目标的类型决定，因此指针的移动的步长即为其所指向的目标的类型所占的空间。

```C
int * ptr1;			// the type of ptr1 is 'int *' --> 步长为 4-bits
float * flt1; 			// the type of flt1 is 'float *' --> 步长为 4-bits 

int arr1[9];
int arr2[10];
float flt1[9];
int arr3[10][20];

&arr1				// the type of &arr1 is 'int (*)[9]' --> 步长为 9 * 4 bits
&arr2				// the type of &arr2 is 'int (*)[10]' --> 步长为 10 * 4 bits
&flt1				// the type of &flt1 is 'float (*)[9]' --> 步长为 9 * 4 bits
&arr3				// the type of &arr3 is 'int (*)[10][20]' --> 步长为 10 * 20 * 4 = 800 bits
&arr3[1]			// the type of &arr3[1] (which is &(arr3[1])) is 'int (*)[20]' --> 步长为 20 * 4 = 80 bits
```

#### 2.2.2.2 下标操作符会被编译器自动转化为指针偏移量
不管是数组名还是指针，在对其进行下标操作时，编译器均先将其转化为指针加偏移量的形式，在对其进行操作。如 `arr[i] --> *(arr + i)` 及 `ptr[i] --> *(ptr + i)`

#### 2.2.2.3 指针相减得到的步长的个数
由此，可以通过以下方式获得数组的长度：

```C
// 获得数组长度的方法二
int arr[10];
size_t size2 = (&arr)[1] - arr;
```

具体的计算过程为，首先 `&arr` 为指向整个数组的指针，`(&arr)[1]` 即 `*(&arr + 1)` 为指针 `&arr + 1` 指向的目标，亦即数组所在内存之后的类型为 `int[10]` 的数组，由于在表达式中药发生退化，所以此处 `(&arr)[1]` 为指向该数组第一个元素（即数组 `arr` 后边第一个越界的元素）的指针（类型为 `int *`）。然后，数组名 `arr` 在此处退化成指向其第一个元素的指针（类型为 `int *`）。因此，二者相减，得到的是类型为 `int *` 从 `arr` 的第一个元素到对应的第一个越界的元素需要经过的步长，即数组的长度。

# 代码验证
代码：

```C
#include <stdio.h>
#include <stdlib.h>

int main(void) {
  // test sizeof operator on arrays
  int arr1[9];
  int arr2[10];
  float flt1[9];
  int arr3[10][20];

  printf("sizeof arr1 is: %lu\n", sizeof arr1);
  printf("sizeof arr2 is: %lu\n", sizeof arr2);
  printf("sizeof flt1 is: %lu\n", sizeof flt1);
  printf("sizeof arr3 is: %lu\n", sizeof arr3);
  printf("sizeof arr3[5] is: %lu\n", sizeof arr3[5]);
  printf("sizeof (&arr1)[1] is: %lu\n", sizeof (&arr1)[1]);
  printf("sizeof (&arr3)[1] is: %lu\n", sizeof (&arr3)[1]);
  printf("sizeof (&arr3)[1][1] is: %lu\n", sizeof (&arr3)[1][1]);

  // tset the methods for obtaining the length of arrays
  int arr[10];
  // method 1
  size_t size1 = (sizeof arr) / (sizeof arr[0]);
  printf("The length of arr[10] is: %lu\n", size1);
  // method 2
  size_t size2 = (&arr)[1] - arr;
  printf("The length of arr[10] is: %lu\n", size2);

  // test pointer operations
  // The key point is printf() is a function, so the parameter is always pointer.
  int array[5] = {1};
  printf("The address of array is %p\n", array);
  printf("The address of &array is %p\n", &array);
  printf("The address of (array + 1) is %p\n", array + 1);
  printf("The address of (&array + 1) is %p\n", &array + 1);

  return EXIT_SUCCESS;
}
```

输出：

```bash
➜  array gcc -o array1 array1.c
➜  array ./array1
sizeof arr1 is: 36
sizeof arr2 is: 40
sizeof flt1 is: 36
sizeof arr3 is: 800
sizeof arr3[5] is: 80
sizeof (&arr1)[1] is: 36
sizeof (&arr3)[1] is: 800
sizeof (&arr3)[1][1] is: 80
The length of arr[10] is: 10
The length of arr[10] is: 10
The address of array is 0x7ffeea0d55d0
The address of &array is 0x7ffeea0d55d0
The address of (array + 1) is 0x7ffeea0d55d4
The address of (&array + 1) is 0x7ffeea0d55e4
➜  array
```
