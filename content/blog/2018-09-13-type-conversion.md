---
title: Type Conversion
subtitle:
date: '2018-09-13'
slug: type-conversion
categories:
  - C
  - Type
tags:
  - type conversion
---
注：本文中提到的 float 或者 floating 均是指的广义的浮点数，包括 float 或者 double 等。

# Important Resources

关于类型及类型转换的所有详细标准均可通过 C99 等相关标准进行查询，非常详实且方便。

[C0X: searching and table of contents](http://c0x.coding-guidelines.com/index.html)  
这是一个将所有要点进行整理的一个网站，并且每一个小节截取了相应的 commentary 用来详细研究。

[The New C Standard - An Economic and Cultural Commentary](http://www.coding-guidelines.com/cbook/cbook1_1.pdf)  
这是一个完整的 pdf 版本。

另外，网上还有一些其他比较好的参考材料：

[INT02-C. Understand integer conversion rules](https://wiki.sei.cmu.edu/confluence/display/c/INT02-C.+Understand+integer+conversion+rules)  
这是 CMU SEI 维护的一份在线文档。

# Type Conversion
以下内容摘自： [All of Programming](https://play.google.com/store/books/details?id=-zViCgAAQBAJ)

> The default type for integer constants (e.g. 2, or 46) is int.  
> The default type for real constants (3.14, or -8.19) is double.
>
> When the compiler inserts a type conversion, it typically must add instructions to the program which cause the processor to explicitly change the bit representation from the size and representation used by the original type to the size and representation used by the new type.
> 
> ### There are four common ways that the bit representations must be changed to convert from one type to another during a type promotion.
> 
> 1. When converting from a smaller signed integer type to a longer signed integer, the number must be sign extended—the sign bit (most significant bit) must be copied an appropriate number of times to fill in the additional bits.
> 2. When converting from a smaller unsigned integer type to a longer unsigned integer type, the number must be zero extended—the additional bits are filled in with all zeros.
> 3. The third common way that the bit representation can be changed during an automatic conversion happens when a longer integer type is converted to a shorter integer type. Here, the bit pattern is truncated to fit—the most significant bits are thrown away, and only the least significant bits are retained.
> 4. The fourth way that the bit representation may need to be changed is to fully calculate what the representation of the value is in the new type. For example, when converting from an integer type to a real type, the compiler must insert an instruction which requests that the CPU compute the floating point (binary scientific notation) representation of that integer.
> 
> ### There are other cases where a type conversion does not need to alter the bit pattern, instead just changing how it is interpreted.
> 
> For example, converting from a signed int to an unsigned int leaves the bit pattern unchanged. However, if the value was originally negative, it will now be interpreted as a large positive number.

# Integer Conversion Rules
以下内容摘自：[INT02-C. Understand integer conversion rules](https://wiki.sei.cmu.edu/confluence/display/c/INT02-C.+Understand+integer+conversion+rules)

> Conversions can occur explicitly as the result of a cast or implicitly as required by an operation.
> 
> The C integer conversion rules define how C compilers handle conversions. These rules include **integer promotions**, **integer conversion rank**, and the **usual arithmetic conversions**.

## Integer Promotions
Integer types smaller than int are promoted when an operation is performed on them. If all values of the original type can be represented as an int, the value of the smaller type is converted to an int; otherwise, it is converted to an unsigned int. Integer promotions are applied as part of the usual arithmetic conversions to certain argument expressions; operands of the unary +, -, and ~ operators; and operands of the shift operators.

## Integer Conversion Rank
Every integer type has an integer conversion rank that determines how conversions are performed. The ranking is based on the concept that each integer type contains at least as many bits as the types ranked below it. The following rules for determining integer conversion rank are defined in the C Standard, subclause [6.3.1.1](http://c0x.coding-guidelines.com/6.3.1.1.html) [ISO/IEC 9899:2011]:

* No two signed integer types shall have the same rank, even if they have the same representation.
* The rank of a signed integer type shall be greater than the rank of any signed integer type with less precision.
* The rank of long long int shall be greater than the rank of long int, which shall be greater than the rank of int, which shall be greater than the rank of short int, which shall be greater than the rank of signed char.
* **The rank of any unsigned integer type shall equal the rank of the corresponding signed integer type, if any.**
* The rank of any standard integer type shall be greater than the rank of any extended integer type with the same width.
* The rank of char shall equal the rank of signed char and unsigned char.
* The rank of _Bool shall be less than the rank of all other standard integer types.
* The rank of any enumerated type shall equal the rank of the compatible integer type.
* The rank of any extended signed integer type relative to another extended signed integer type with the same precision is implementation-defined but still subject to the other rules for determining the integer conversion rank.
* For all integer types T1, T2, and T3, if T1 has greater rank than T2 and T2 has greater rank than T3, then T1 has greater rank than T3.

The integer conversion rank is used in the usual arithmetic conversions to determine what conversions need to take place to support an operation on mixed integer types.

## Usual Arithmetic Conversions
The usual arithmetic conversions are rules that provide a mechanism to yield a common type when both operands of a binary operator are balanced to a common type or the second and third operands of the conditional operator ( ? : ) are balanced to a common type. Conversions involve two operands of different types, and one or both operands may be converted. Many operators that accept arithmetic operands perform conversions using the usual arithmetic conversions. After integer promotions are performed on both operands, the following rules are applied to the promoted operands:

1. If both operands have the same type, no further conversion is needed.
2. If both operands are of the same integer type (signed or unsigned), the operand with the type of lesser integer conversion rank is converted to the type of the operand with greater rank.
3. **If the operand that has unsigned integer type has rank greater than or equal to the rank of the type of the other operand, the operand with signed integer type is converted to the type of the operand with unsigned integer type.**
4. **If the type of the operand with signed integer type can represent all of the values of the type of the operand with unsigned integer type, the operand with unsigned integer type is converted to the type of the operand with signed integer type.**
5. **Otherwise, both operands are converted to the unsigned integer type corresponding to the type of the operand with signed integer type.**

# [6.3.1.3 Signed and unsigned integers](http://c0x.coding-guidelines.com/6.3.1.3.html)

* 682 When a value with integer type is converted to another integer type other than _Bool, if the value can be represented by the new type, it is unchanged.
* 683 Otherwise, **if the new type is unsigned, the value is converted by repeatedly adding or subtracting one more than the maximum value that can be represented in the new type until the value is in the range of the new type.**49)
* 684 Otherwise, the new type is signed and the value cannot be represented in it;
* 685 either the result is implementation-defined or an implementation-defined signal is raised.

# [6.3.1.4 Real floating and integer](http://c0x.coding-guidelines.com/6.3.1.4.html)
最 tricky 的一种类型转换是 real floating --> integer.

* 686 **When a finite value of real floating type is converted to an integer type other than _Bool, the fractional part is discarded (i.e., the value is truncated toward zero).**
* 687 **If the value of the integral part cannot be represented by the integer type, the behavior is undefined.50)**
* 688 When a value of integer type is converted to a real floating type, if the value being converted can be represented exactly in the new type, it is unchanged.
* 689 If the value being converted is in the range of values that can be represented but cannot be represented exactly, the result is either the nearest higher or nearest lower representable value, chosen in an implementation-defined manner.
* 690 48) The integer promotions are applied only: as part of the usual arithmetic conversions, to certain argument expressions, to the operands of the unary +, -, and ~ operators, and to both operands of the shift operators, as specified by their respective subclauses.
* 691 49) The rules describe arithmetic on the mathematical value, not the value of a given type of expression.
* 692 50) **The remaindering operation performed when a value of integer type is converted to unsigned type need not be performed when a value of real floating type is converted to unsigned type.**
* 693 **Thus, the range of portable real floating values is (-1, Utype_MAX+1).**
* 694 **If the value being converted is outside the range of values that can be represented, the behavior is undefined.**

下边，根据相关标准，浮点数转换成整型的规则进行研究。

> When a finite value of real floating type is converted to an integer type other than _Bool, the fractional part is discarded (i.e., the value is truncated toward zero).

首先第一步是将小数部分直接丢掉，如 `1.3--> 1`，`2.7 --> 2`，`-1.1 --> -1`，`-2.8 --> -2` 等。

> If the value of the integral part cannot be represented by the integer type, the behavior is undefined.50)
> 
> 50) The remaindering operation performed when a value of integer type is converted to unsigned type need not be performed when a value of real floating type is converted to unsigned type.  
> The conversion behavior, when the result cannot be represented in the destination type is undefined in C++ and unspecified in C.  
> Thus, the range of portable real floating values is (−1, Utype_MAX + 1).

然后我们看剩余的整数部分，如果这个不能被目标整型所表示（注意：int --> unsigned 时采用的 remindering operation，如 -2 --> 2^32 - 2，在进行 float --> unsigned 时不再采用，即认为 -2 不能被 unsigned int 所表示），则这种转换行为在 C 中是不确定的，在 C++ 中是没有被定义的。

因此，float --> unsigned 的有效的范围是开区间 (-1, Utype_MAX + 1)。

> When a floating-point value in the range (-1.0, -0.0) is converted to an integer type, the result is required to be a positive zero.

**验证**

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
  unsigned  x1 = 1.1;
  unsigned  x2 = 2.9;
  unsigned  x3 = 0.0;
  unsigned  x4 = -0.0;
  unsigned  x5 = -1.3;
  unsigned  x6 = -2.7;
  unsigned  x7 = -3.3;
  unsigned  x8 = -4.7;
  unsigned  x9 = -1.0;
  unsigned  x10 = -0.7;
  unsigned  x11 = -0.3;

  printf("%u, %u, %u, %u, %u, %u, %u, %u, %u, %u, %u\n", x1, x2, x3, x4, x5, x6, x7, x8, x9, x10, x11);
  return EXIT_SUCCESS;
}
```

输出为：

```
1, 2, 0, 0, 771, 771, 771, 771, 771, 0, 0
```

注：出现 711 是因为，C 语言对这些不能被 unsigned 表示的负实数采取了 unspecified 行为，因此这些值对应的 unsigned 值是不确定的。
