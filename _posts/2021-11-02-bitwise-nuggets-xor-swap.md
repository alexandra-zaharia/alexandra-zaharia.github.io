---
title: Bitwise nuggets&#58; XOR swap
date: 2021-11-02 00:00:00 +0100
categories: [algorithms, bitwise]
tags: [algorithms, swap, xor]
---

XOR swap is a kind of in-place swap that only uses XOR operations. No temporary variable nor addition/subtraction operation is needed. However, it only works for integers.

**Reminder:** XOR (short for _exclusive or_) is a logical operation that yields true only if its arguments differ (one argument is true, the other one is false). Substitute `0` for `false` and `1` for `true` in the following:

* `0 XOR 0 = 0`
* `0 XOR 1 = 1`
* `1 XOR 0 = 1`
* `1 XOR 1 = 0`

The following C++ program illustrates the use of the XOR swap to exchange two integers (passed as pointers to the function):

```c++
#include <iostream>

void xor_swap(int *a, int *b)
{
    if (*a == *b) return;
    *a ^= *b;
    *b ^= *a;
    *a ^= *b;
}

int main()
{
    int x = 5, y = 7;
    std::cout << "x = " << x << ", y = " << y << std::endl;
    xor_swap(&x, &y);
    std::cout << "x = " << x << ", y = " << y << std::endl;
    return 0;
}
```

Not that it's very useful in practice since modern compilers optimize swapping when using a temporary variable (and there's also the `XCHG` assembler instruction), but it's fun to know about it :upside_down_face: