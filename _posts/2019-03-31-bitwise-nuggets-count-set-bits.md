---
title: Bitwise nuggets&#58; count set bits
date: 2019-03-31 00:00:00 +0100
categories: [C/C++, bitwise]
tags: [c/c++, algorithms, binary]
---

If we want to count the number of set bits (i.e. bits that are `1`) in an int, we can check whether its least significant bit (LSB) is 1, count it as set if applicable, then right-shift the int by one position until we've seen the number of bits in an int.

Here is how this can be implemented in C:

```c
#include <stdio.h>
#include <limits.h>

// Returns the number of "on" bits (bits set to 1) in the input `number`.
unsigned int count_set_bits(int number)
{
    unsigned int n_bits_on = 0;

    int value = number;

    for (size_t i = 0; i < CHAR_BIT * sizeof(int); i++) {
        if ((1 & value) == 1)
            ++n_bits_on;
        value >>= 1;
    }

    return n_bits_on;
}

int main()
{
    int numbers[] = {7, 2019, -1, -2};
    for (int i = 0; i < 4; i++)
        printf("count_set_bits(%4d) = %2d\n", numbers[i], count_set_bits(numbers[i]));
}
```

At line 11, we iterate for the number of bits in an int, which can also be written as `CHAR_BIT` (macro constant defined in `limits.h` that represents the number of bits in a char, i.e. 8) multiplied by the number of bytes in an int (which is `sizeof(int)`).

Here is the output of the above program:

```
count_set_bits(   7) =  3
count_set_bits(2019) =  8
count_set_bits(  -1) = 32
count_set_bits(  -2) = 31
```

Want to see more [bitwise][] logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations
[bitwise]: {{ site.baseurl }}{% link categories/bitwise/index.html %}