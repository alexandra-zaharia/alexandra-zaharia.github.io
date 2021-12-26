---
title: Bitwise nuggets&#58; count total bits
date: 2019-03-30 02:00:00 +0100
categories: [C/C++, bitwise]
tags: [c/c++, algorithms, binary]
---

If we want to count the total number of bits in an integer, we can divide that number by two until it becomes zero, counting a bit at each step.

Here is how this can be implemented in C:

```c
#include <stdio.h>

// Determine the number of bits that the given `number` requires.
unsigned int count_total_bits(unsigned int number)
{
    unsigned int bits = 0;
    while (number) {
        number >>= 1;
        ++bits;
    }
    return bits;
}

int main()
{
    int numbers[] = {7, 2019, -1, -2};
    for (int i = 0; i < 4; i++)
        printf("count_total_bits(%4d) = %2d\n", numbers[i], count_total_bits(numbers[i]));
}
```

At line 8, instead of dividing `number` by two, we right-shift it by 1 position (since the logical shift operation is not expensive).

Here is the output of the above program:

```
count_total_bits(   7) =  3
count_total_bits(2019) = 11
count_total_bits(  -1) = 32
count_total_bits(  -2) = 32
```

Want to see more [bitwise][] logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations
[bitwise]: {{ site.baseurl }}{% link categories/bitwise/index.html %}