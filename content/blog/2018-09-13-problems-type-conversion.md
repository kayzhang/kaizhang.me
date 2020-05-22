---
title: Some problems about type coversion
subtitle: arising from a program about power operation
date: '2018-09-13'
slug: problems-type-conversion
categories:
  - C
  - Type
tags:
  - type conversion
---

When I was working on testing a C program about power operation, something strange happened. The `power()` algorithm is as follows:

```C
#include <stdio.h>
#include <stdlib.h>

unsigned power(unsigned x, unsigned y) {
  //Here, we assume that 0^0 == 1.
  if (y == 0) {
    return 1;
  }

  return x * power(x, y - 1);
}

void run_check(unsigned x, unsigned y, unsigned expected_ans) {
  if (power(x, y) != expected_ans) {
    printf("power(%u, %u) != %u\n", x, y, expected_ans);
    exit(EXIT_SUCCESS);
  }
  printf("power(%u, %u) == %u\n", x, y, expected_ans);
  return;
}

int main(void) {
  run_check(-2, 3, -8);
  run_check(1.1, 2, 1.21);

  printf("All test cases approved!\n");
  return EXIT_SUCCESS;
}
```

The output is:

```
power(4294967294, 3) == 4294967288
power(1, 2) == 1
All test cases approved!
```

## First Problem: Why `run_check(-2, 3, -8)` gets approved?

The first problem that I encountered was that I thought `power(-2, 3)` won't generate `-8`. Becasue when `-2` is assigned to a unsigned variable, it will be intepreted as a very large unsigned number. And thus the result will no longer be `-8`. So what's the problem?

The answer is although it seems like that `power(-2, 3)` is calculated under `int (signed)`, it isn't true. Indeed, it's calculated under `unsigned int`, but the result is also `"-8"`, which is tricky.

When `-2` is assigned to `x`, `x` is interpreted as a large number `4294967294 == 2^32 - 2`. So `power(-2, 3)` is calculated as follows:

**First**, calculate `x^2`.

\begin{equation}\begin{split} x^2 = (2^{32} - 2) \cdot (2^{32} -2)\\\\
&= (2^{64} - 4 \cdot 2^{32}) + 4\\\\
\end{split}\end{equation}

Because unsigned int is 32-bits, so the left part `2^64 - 4 * 2^32` will be truncated due to overflow. Therefore, the result is `power(-2, 2) == 4`.

**Second**, calculate `x^3`.

\begin{equation}\begin{split} x^3 = x^2 \cdot (2^{32} - 2)\\\\
&= 4 \cdot (2^{32} - 2)\\\\
&= 4 \cdot 2^{32} - 8
\end{split}\end{equation}

So, the left part `4 * 2^32` is truncated, and the result is `power(-2, 3) == -8`.

### Verification

To verify this rule, we can see that for unsigned int number `unsigned x = -2`, if we define `unsigned y = x / 2`, then `y == (2^32 - 2) / 2 == 2147483647` instead of `y == -1` which is `2^32 - 1 == 4294967294`.

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
  unsigned x = -2;
  unsigned y = x / 2;
  unsigned z = x * x;
  unsigned p = x * x * x;
  printf("x: %d, %u\n", x, x);
  printf("y: %d, %u\n", y, y);
  printf("z: %d, %u\n", z, z);
  printf("p: %d, %u\n", p, p);
  return EXIT_SUCCESS;
}
```

The output is:

```
x: -2, 4294967294
y: 2147483647, 2147483647
z: 4, 4
p: -8, 4294967288
```

## Second Problem: Why `run_check(1.1, 2, 1.21)` gets approved?

The answer is in run_check it's not comparing `if (power(1.1, 2) == 1.21)`. Instead, it's actually comparing `if (power(1, 2) == 1)`, and the answer is obviously yes!

The reason is that when a real constant (e.g. 1.1, 2.9, ...) is assigned (more generally, converted) to integer, **the default float-to-integer conversion in C truncates the decimal part toward zero.** So `1.1 --> 1`, `2.9 --> 2`, `-1.3 --> -1`, `-2.7 --> -2`, etc.

此处具体的 float --> unsigned 的规则并非单纯的丢掉小数部分，具体的参见： [Type Conversion](https://kaizhang.me/note/2018/13/type-conversion/) 

## 验证

结合 [Type Conversion](https://kaizhang.me/note/2018/13/type-conversion/) 中的规则，可以对以下代码的输出进行验证。

```c
#include <stdio.h>
#include <stdlib.h>

unsigned power(unsigned x, unsigned y) {
  //Here, we assume that 0^0 == 1.
  if (y == 0) {
    return 1;
  }

  return x * power(x, y - 1);
}

void run_check(unsigned x, unsigned y, unsigned expected_ans) {
  if (power(x, y) != expected_ans) {
    printf("power(%u, %u) != %u\n", x, y, expected_ans);
    exit(EXIT_SUCCESS);
  }
  printf("power(%u, %u) == %u\n", x, y, expected_ans);
  return;
}

int main(void) {
  run_check(0, 0, 1);

  // test negative int --> unsigned
  run_check(-2, 3, -8);
  // run_check(2, -3, 1/8);  // segmentation fault
  // run_check(-2, -2, 1/4); // segmentation fault

  // test float --> unsigned
  run_check(1.1, 2, 1.21);
  run_check(2.5, 2, 4);
  run_check(-0.7, 2, 0);
  run_check(-1.2, 2, 1.44);

  printf("All test cases approved!\n");

  return EXIT_SUCCESS;
}
```

输出为：

```
power(0, 0) == 1
power(4294967294, 3) == 4294967288
power(1, 2) == 1
power(2, 2) == 4
power(0, 2) == 0
power(73832, 2) != 1
```
