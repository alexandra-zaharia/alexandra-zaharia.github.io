---
title: Bitwise nuggets&#58; clear the least significant bits up to a given position
date: 2019-04-01 00:00:00 +0100
categories: [C/C++, bitwise]
tags: [c/c++, algorithms, binary]
---

Here we discuss how to clear the least significant bits (LSB) in an integer up to a given position `pos` (including `pos`). (Check out [this post][pp] for clearing the MSBs up to a given position.)

What does that even mean? Well, in a number, bits are numbered starting from 0, where the bit at position 0 is the least significant bit (or LSB for short). Take the number 2019 for instance; its LSB (at position 0) is 1 and its MSB (at position 10) is also 1:

```
pos:   10        0
       v         v
2019 = 11111100011
       ^         ^
      MSB       LSB
```

Clearing the LSBs in a number up to a given position would mean zero-ing them out while leaving the MSBs untouched. For example, if we were to clear the LSBs in number 2019 up to and including position 6, we would get 1920:

```
pos:       6     0
           v     v
2019 = 11111100011
           |
           v
    clear LSB up to (and including) pos 6

pos:       6     0
           v     v
1920 = 11110000000
```

The idea is to apply a mask to the integer, where the mask is all zeros for the `pos + 1` least significant bits, i.e. the bits we want to clear. We obtain the mask by left-shifting 1 by `pos + 1` bits, then subtracting 1 (to get all ones), and finally inverting (logical `NOT`) the whole mask. The mask is applied by AND-ing it with the number. It has the effect of preserving the MSBs starting at position `pos + 1` and of clearing (zeroing) LSBs up to and including position `pos`.

Here is how this can be implemented in C:

```c
// Clears the least significant bits in `number` up to the bit at
// position `pos` (inclusive).
int clear_least_significant_bits_up_to_pos(int number, int pos)
{
    return number & ~((1 << (pos + 1)) - 1);
}
```

Here is what becomes of number 2019 when we clear its LSBs up to positions 0 through 11 (recall that the MSB of 2019 is at position 10):

```
clear_least_significant_bits_up_to_pos(2019,  0) = 2018 = 11111100010
clear_least_significant_bits_up_to_pos(2019,  1) = 2016 = 11111100000
clear_least_significant_bits_up_to_pos(2019,  2) = 2016 = 11111100000
clear_least_significant_bits_up_to_pos(2019,  3) = 2016 = 11111100000
clear_least_significant_bits_up_to_pos(2019,  4) = 2016 = 11111100000
clear_least_significant_bits_up_to_pos(2019,  5) = 1984 = 11111000000
clear_least_significant_bits_up_to_pos(2019,  6) = 1920 = 11110000000
clear_least_significant_bits_up_to_pos(2019,  7) = 1792 = 11100000000
clear_least_significant_bits_up_to_pos(2019,  8) = 1536 = 11000000000
clear_least_significant_bits_up_to_pos(2019,  9) = 1024 = 10000000000
clear_least_significant_bits_up_to_pos(2019, 10) =    0 = 00000000000
```

Want to see more bitwise logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations
[pp]: {% post_url 2019-03-31-bitwise-nuggets-clear-msb-up-to-position %}