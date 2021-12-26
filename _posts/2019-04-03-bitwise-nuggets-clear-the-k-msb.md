---
title: Bitwise nuggets&#58; clear the first k most significant bits
date: 2019-04-03 00:00:00 +0100
categories: [C/C++, bitwise]
tags: [c/c++, algorithms, binary]
---

Here we discuss how to clear the first `k` most significant bits (MSB) in an integer. This is a more intuitive variation of the [clear MSB up to a given position][pp] problem that we have seen earlier.

What does that even mean? Well, in a number, bits are numbered starting from 0, where the bit at position 0 is the least significant bit (or LSB for short). Take the number 2019 for instance; its LSB (at position 0) is 1 and its MSB (at position 10) is also 1:

```
pos:   10        0
       v         v
2019 = 11111100011
       ^         ^
      MSB       LSB
```

Clearing the `k` MSBs in a number would mean zero-ing them out while leaving the LSBs untouched. For example, if we were to clear the first 3 MSBs in number 2019 up, we would get 227:

```
k:       3
         v
2019 = 11111100011
           |
           v
    clear the 3 MSBs

k:       3
         v
 227 = 00011100011
```

The idea is to apply a mask to the integer, where the mask is all zeroes for the `k` most significant bits, i.e. the bits we want to clear. The remaining mask is all ones. We obtain the mask by left-shifting 1 by the difference between the total number of bits and `k`, then subtracting 1 (to get all ones). The mask is applied by AND-ing it with the number. It has the effect of preserving the LSBs and of clearing (zeroing) the first `k` MSBs.

We first need `count_total_bits()`, a helper function to count the [total number of bits][total-bits] in a number. Using this function, we can now clear the `k` most significant bits as follows:

```c
// Clears the first `k` most significant bits in `number`.
int clear_most_significant_bits(int number, unsigned int k)
{
    unsigned int n_bits = count_total_bits(number);
    return number & ((1 << (n_bits - k)) - 1);
}
```

Here is what becomes of number 2019 when we clear its MSBs up to positions 0 through 11 (recall that the MSB of 2019 is at position 10):

```
clear_msb(2019,  0) = 2019 = 11111100011
clear_msb(2019,  1) =  995 = 01111100011
clear_msb(2019,  2) =  483 = 00111100011
clear_msb(2019,  3) =  227 = 00011100011
clear_msb(2019,  4) =   99 = 00001100011
clear_msb(2019,  5) =   35 = 00000100011
clear_msb(2019,  6) =    3 = 00000000011
clear_msb(2019,  7) =    3 = 00000000011
clear_msb(2019,  8) =    3 = 00000000011
clear_msb(2019,  9) =    3 = 00000000011
clear_msb(2019, 10) =    1 = 00000000001
clear_msb(2019, 11) =    0 = 00000000000
```

Want to see more [bitwise][] logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[pp]: {% post_url 2019-03-31-bitwise-nuggets-clear-msb-up-to-position %}
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations
[bitwise]: {{ site.baseurl }}{% link categories/bitwise/index.html %}
[total-bits]: {% post_url 2019-03-30-bitwise-nuggets-count-total-bits %}