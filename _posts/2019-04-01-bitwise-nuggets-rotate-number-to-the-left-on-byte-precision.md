---
title: Bitwise nuggets&#58; rotate a number to the left using byte precision
date: 2019-04-01 01:00:00 +0100
categories: [C/C++, bitwise]
tags: [c/c++, algorithms, binary]
---

Here we discuss how to rotate a number to the left by `k` positions, using byte precision.

What does that even mean? We want to left-shift the number by `k` bits, but the most significant `k` bits that overflow as a result of the left-shift operation must be saved and added to the `k` least significant bits of the left-shifted number (these bits are zero after the left-shift). In addition, the rotation has *byte precision*, meaning we left-shift on the number of bytes that the number requires for its representation.

Let us see an example to better visualize the rotation to the left. Suppose we want to rotate the number 100 to the left by 3 positions. As shown below, we expect to obtain 35:

![rotate_left(100, 3)](/assets/img/posts/bitwise_rotate_left_byte.png){: width="700"}

Since we're using byte precision, the actual overflow when 100 is left-shifted by 3 positions is `011`, not `11`. It is possible to rotate a number on the number of *bits* it requires for its representation, but that is a subject for another time.

Here are the steps for solving this problem:

1. determine the number of bytes that the number requires
1. compute the *overflow*, or the `k` MSB of the number
1. left-shift the number by `k` positions
1. if we go over the number of bytes used for representing the original number, zero-out the extra bits
1. add the overflow to the left-shifted number

Here is how this algorithm can be implemented in C:

```c
#include <limits.h>

// Rotate the given `number` to the left by `k` positions. Rotation takes place on
// 1 through 4 bytes, depending on how many the `number` needs for its
// representation.
unsigned int rotate_left_byte_precision(unsigned int number, unsigned int k) {
    // number of bits multiple of CHAR_BIT needed to represent `number`
    unsigned int precision = 0;

    // number of bytes needed to represent `number` (maximum: 4)
    unsigned int number_of_bytes = 1;

    while (!precision) {
        unsigned int cmp = (unsigned int) ((1 << (CHAR_BIT * number_of_bytes)) - 1);
        if (number <= cmp || ++number_of_bytes == 4)
            precision = CHAR_BIT * number_of_bytes;
    }

    // Overflowing bits are the `precision - k` most significant ones.
    unsigned int overflow = number >> (precision - k);

    // Left-shift `number` by `k` bits.
    unsigned int left_shifted = number << k;

    // Clear the unneeded bytes.
    if (precision < CHAR_BIT * sizeof(unsigned int))
        left_shifted &= (unsigned int) ((1 << precision) - 1);

    // Add the overflow.
    return left_shifted | overflow;
}
```

At lines 13-17 we determine the `precision`, i.e. the number of bits multiple of 8 (`CHAR_BIT`) required to represent the `number`. We need the `precision` in order to determine the `overflow` bits at line 20. At lines 26-27, we ensure that the rotation takes place only on the number of bytes that are required to represent the original number. Finally, at line 30 we add the `overflow` to the `left_shifted` number using the logical `OR` operator.

Here are some examples of left rotation on byte precision in action:

```
rotate_left_byte_precision(       100,  3) =         35
rotate_left_byte_precision(       244,  3) =        167
rotate_left_byte_precision(       356, 10) =      36869
rotate_left_byte_precision(      2019,  4) =      32304
rotate_left_byte_precision(    983396,  8) =      91151
rotate_left_byte_precision(3422643215,  5) = 2150400505
```

Want to see more bitwise logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations
