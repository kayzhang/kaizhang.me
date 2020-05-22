---
title: 使用 fgetc() 函数的常见错误
subtitle: 
date: '2018-09-24'
slug: a-subtle-bug-of-fgetc
categories:
  - C
  - IO
tags:
  - IO
---

在使用 fgetc() 函数时容易出现以下错误，导致程序存在比较 tricky 和 subtle 的 bug。

# fgetc() 函数简介

通过 `man fgetc` 可以得到以下信息：

```C
SYNOPSIS

#include <stdio.h>
int fgetc(FILE *stream);

DESCRIPTION

fgetc() reads the next character from stream and returns it as an unsigned char cast to an int, or EOF on end of file or error.
```

这里边有两点需要注意：

1. Reading the character advances the current position in the stream.
2. fgetc() returns an **int** so that it can return all possible **char**s, plus a distinct value to indicate that there are no more characters available in the stream: that the end of file (*EOF*) has been reached.

# Bug 1. Forget the position of the stream is advanced automatically.

下面的 broken 代码尝试逐个打印输入文件中的每个字符。

```C
FILE * f = fopen(inputFileName, "r");
if(f == NULL) {*/ error handling code omitted */}
while (fgetc(f) != EOF) {
  char c = fgetc(f);
  printf("%c", c);
}
// ...other code...
```

这段代码的运行过程为：读取一个字符，检查是否到达文尾，然后读取**另外一个不同的字符**，并将其打印输出。其实际功能为：当字符个数为偶数（从 1 开始编号）时，打印所有偶数位的字符；当字符个数为奇数时，打印所有偶数位的字符，并且在最后加上 `ÿ`，即 ASCII 中 255（或者 0xFF） 对应的字符，因为此时的 EOF（0xFFFFFFFF）被 `char c = fgetc(f)` 语句 cast 为了 char（0xFF）。

正确的代码如下：

```C
FILE * f = fopen(inputFileName, "r");
if(f == NULL) {*/ error handling code omitted */}
int c;
while ((c = fgetc(f)) != EOF) {
  printf("%c", c);
}
// ...other code...
```

# Bug 2. Assigning the return value of fgetc() to a char.
注意，从 fgetc() 的函数描述中可以看到，其返回值分为以下几种情况：

1. 如果读取到的是 ASCII 字符（8-bits），则首先将其视为 unsigned char，然后将其 cast 为 int 并返回。注意此处因为将其视为 unsigned char，因此为补零扩展，范围为 0x00000000 ~ 0x000000FF，即 0 ~ 255.
2. 如果到达了文件结尾，则返回 EOF。其中，EOF 是在 stdio.h 中通过 macro 定义的 int 常量，值为 -1，即 0xFFFFFFFF。
3. 当遇到读取错误时，返回 error 信息。

因此，此处 fgetc() 为了能够返回所有的 characters（共 N = 256 个）加上一个额外的 distinct value for EOF，即总共 (N + 1 = 257) 个值，其返回值为 int 而非 char。

如果我们将其返回值赋值给 char 类型变量时，会出现什么呢？

首先要搞清楚 char， unsigned char，signed char 之间的关系。

1. char 或 (plain) char 的具体实现在 C 标准中并未定义，应视具体的平台而定。对于特定的机器，可以通过打印 limits.h 中的常量 `CHAR_MIN` 和 `CHAR_MAX` 来确定。若为 unsigned char 则，`CHAR_MIN = 0`，`CHAR_MAX = 255`；若为 signed char 则，`CHAR_MIN = -128`，`CHAR_MAX = 127`。
2. 若 char 为 unsigned char，则其转换为 int 的规则与 unsigned char 保持一致，即补零扩展。
3. 若 char 为 signed char，则其转换为 int 的规则与 signed char 保持一致，即补符号位扩展。

此处以学校服务器为例进行说明，在该服务器上 char 为 signed char。

下边回到之前的问题，当 fgetc() 返回值赋值给 char 类型变量时，相当于发生了信息丢失，即把取值范围包含 (N + 1) 个数据的变量赋值给了取值范围包含 N 个数据的变量。

此时，0x00000000 ~ 0x000000FE 的数据仍对应于 char 中的唯一字符，但是 0x000000FF（即 ASCII 255 字符 `ÿ`）与 0xFFFFFFFF （即 EOF）会被 cast 为同一个 char，即 0xFF（对应于 `ÿ`）。

此时会发生两种情况。

第一，当直接将 c 作为 char 使用时，`ÿ` 和 EOF 都会被认为是 `ÿ`。如 Bug 1 中字符个数为奇数的情况。如：

```C
FILE * f = fopen(inputFileName, "r");
char c = fgetc(f);
printf("%c", c);
```

上述代码中，无论 fgetc(f) 读到的是 `ÿ` 还是 EOF，`printf("%c", c);` 的结果都为 `ÿ`，因为 %c 将 c 转换为了 char，即 0xFF，其对应 `ÿ`。

第二，当将 c 再次转换为 int 时，由于其为 signed char，会发生补符号位扩展，因此 `ÿ` 和 EOF 都会被认为是 EOF。如:

```C
FILE * f = fopen(inputFileName, "r");
char c;
while ((c = fgetc(f)) != EOF) {
  //code
}
```

上述代码中，无论 fgetc(f) 读到的是 `ÿ` 还是 EOF，`(c = fgetc(f)) != EOF` 的结果都为 false（在比较的时候 c 发生了向 int 的 promotion），while 循环都会结束，相当于遇到了文件结束符。
