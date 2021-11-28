---
title: Bitwise nuggets&#58; get bit at position
date: 2019-03-31 01:00:00 +0100
categories: [C/C++, bitwise]
tags: [c/c++, algorithms, binary]
---

Here is how to obtain the bit at a given position `pos` in an integer. Bit positions start at 0, and the bit at position 0 is the least significant bit (LSB).

The idea is to apply a mask to the integer, where the mask is all zeros except for a bit set at 1 at position `pos`. We obtain the mask by left-shifting 1 by `pos` bits. The mask is applied by AND-ing it with the number. If the result is 0, it means the bit at `pos` is also 0, otherwise its value is 1.

Here is the process described above, step-by-step, for getting the bit at position 6 in the number 2019:

```
mask:                  1
mask << 6:       1000000
2019:        11111100011
2019 & mask: 00001000000
```

We see that `2019 & mask` is different than 0, so the bit at position 6 in 2019 is 1.

Let's now get the bit at position 3 in 2019:

```
mask:                  1
mask << 3:          1000
2019:        11111100011
2019 & mask: 00000000000
```

Now the result is 0, so the bit at position 3 in 2019 is also 0.

Here is how this can be implemented in C:

```c
// Returns the bit at position `pos` in the input `number`.
// The LSB is at position 0.
unsigned int get_bit_at_position(int number, int pos)
{
    return (number & (1 << pos)) != 0;
}
```

Want to see more bitwise logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations