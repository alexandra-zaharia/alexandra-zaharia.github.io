---
title: Bitwise nuggets&#58; invert the n least significant bits
date: 2019-03-30 00:00:00 +0100
categories: [C, bitwise]
tags: [algorithms, binary]
---

Suppose we want to invert the `n` least significant bits in a number. For example:

```
2019 = 11111100011
              ||||
2028 = 11111101100
```

If we invert the least significant 4 bits in the number 2019, we obtain 2028.

When we hear `invert`, we automatically think of XOR:

* `0 XOR 0 = 0`
* `0 XOR 1 = 1`
* `1 XOR 0 = 1`
* `1 XOR 1 = 0`

We therefore need to XOR the input number with a mask of `n` bits that are all 1...1. To obtain this mask, we can left-shift 1 for `n` positions, then we subtract 1. Following our previous example:

```
2019          : 11111100011
mask = 1 << 4 :       10000
mask -= 1     :       01111
2019 ^ mask   : 11111101100 = 2028
```

Here is how this operation may be implemented in C:

```c
// Inverts the `n_bits` least significant bits in `number`
// and returns the resulting number.
int invert_n_lsb(int number, int n_bits)
{
    return number ^ ((1 << n_bits) - 1);
}
```

Want to see more bitwise logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations