---
title: Bitwise nuggets&#58; clear the most significant bits up to a given position
date: 2019-03-31 02:00:00 +0100
categories: [C/C++, bitwise]
tags: [c/c++, algorithms, binary]
---

Here we discuss how to clear the most significant bits (MSB) in an integer up to a given position `pos` (including `pos`). (Check out [this post][np] for clearing the LSBs up to a given position, or [this one][np2] for clearing only the first `k` MSBs.)

What does that even mean? Well, in a number, bits are numbered starting from 0, where the bit at position 0 is the least significant bit (or LSB for short). Take the number 2019 for instance; its LSB (at position 0) is 1 and its MSB (at position 10) is also 1:

```
pos:   10        0
       v         v
2019 = 11111100011
       ^         ^
      MSB       LSB
```

Clearing the MSBs in a number up to a given position would mean zero-ing them out while leaving the LSBs untouched. For example, if we were to clear the MSBs in number 2019 up to and including position 6, we would get 35:

```
pos:       6     0
           v     v
2019 = 11111100011
           |
           v
    clear MSB up to (and including) pos 6

pos:       6     0
           v     v
  35 = 00000100011
```

The idea is to apply a mask to the integer, where the mask is all ones for the `pos` least significant bits, i.e. the bits we want to keep. We obtain the mask by left-shifting 1 by `pos` bits, then subtracting 1 (to get all ones). The mask is applied by AND-ing it with the number. It has the effect of preserving the LSBs and of clearing (zeroing) the MSBs up to and including position `pos`.

Here is how this can be implemented in C:

```c
// Clears the most significant bits in `number` up to the bit at
// position `pos` (inclusive).
int clear_most_significant_bits_up_to_pos(int number, int pos)
{
    return number & ((1 << pos) - 1);
}
```

Here is what becomes of number 2019 when we clear its MSBs up to positions 0 through 11 (recall that the MSB of 2019 is at position 10):

```
clear_most_significant_bits_up_to_pos(2019,  0) =    0 = 00000000000
clear_most_significant_bits_up_to_pos(2019,  1) =    1 = 00000000001
clear_most_significant_bits_up_to_pos(2019,  2) =    3 = 00000000011
clear_most_significant_bits_up_to_pos(2019,  3) =    3 = 00000000011
clear_most_significant_bits_up_to_pos(2019,  4) =    3 = 00000000011
clear_most_significant_bits_up_to_pos(2019,  5) =    3 = 00000000011
clear_most_significant_bits_up_to_pos(2019,  6) =   35 = 00000100011
clear_most_significant_bits_up_to_pos(2019,  7) =   99 = 00001100011
clear_most_significant_bits_up_to_pos(2019,  8) =  227 = 00011100011
clear_most_significant_bits_up_to_pos(2019,  9) =  483 = 00111100011
clear_most_significant_bits_up_to_pos(2019, 10) =  995 = 01111100011
clear_most_significant_bits_up_to_pos(2019, 11) = 2019 = 11111100011
```

Want to see more [bitwise][] logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations
[np]: {% post_url 2019-04-01-bitwise-nuggets-clear-lsb-up-to-position %}
[np2]: {% post_url 2019-04-03-bitwise-nuggets-clear-the-k-msb %}
[bitwise]: {{ site.baseurl }}{% link categories/bitwise/index.html %}