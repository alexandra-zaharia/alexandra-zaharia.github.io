---
title: Bitwise nuggets&#58; convert an integer to a binary string
date: 2019-03-30 00:00:00 +0100
categories: [C/C++, bitwise]
tags: [c/c++, algorithms, binary]
---

When we want to display an int as a binary string, we need a utility function that builds the string representation of the integer. We do this from the least significant bit (LSB) up to the most significant bit (MSB) of the integer.

The idea, at each step, is to check whether the LSB in the number is a 0 or a 1, and then to divide the number by 2 (i.e. to right-shift it by 1 position).

Here is how we can get the binary string representation of an int in C:

```c
#include <limits.h>
#include <stdio.h>

// The binary string representation of the input `number` will be stored in
// `binary`.
void int_to_binary_string(int number, char *binary)
{
    // 1 & number gives the value of the LSB in `number`
    for (int i = CHAR_BIT * sizeof(int) - 1; i >= 0; i--, number >>= 1)
        binary[i] = (1 & number) ? '1' : '0';

    // null-terminate the string
    binary[CHAR_BIT * sizeof(int)] = '\0';
}

int main()
{
    char binary[CHAR_BIT * sizeof(int) + 1];
    int numbers[4] = {7, 2019, -1, -2};

    for (int i = 0; i < 4; i++) {
        int_to_binary_string(numbers[i], binary);
        printf("int_to_binary_string(%4d) = %s\n", numbers[i], binary);
    }
}
```

Here, the `int_to_binary_string()` function takes a `char *binary` argument, which allows us to define a char array on the stack that we can pass to the function (rather than creating a char array on the heap through dynamic memory allocation and having to remember to `free()` the array when we're done with it).

`CHAR_BIT` is a macro constant defined in `limits.h` and represents the number of bits in a char (i.e. 8).

The above program outputs:

```
int_to_binary_string(   7) = 00000000000000000000000000000111
int_to_binary_string(2019) = 00000000000000000000011111100011
int_to_binary_string(  -1) = 11111111111111111111111111111111
int_to_binary_string(  -2) = 11111111111111111111111111111110
```

Want to see more bitwise logic? There's a whole repository on my [GitHub] on bit fiddling.

<!-- links -->
[GitHub]: https://github.com/alexandra-zaharia/c-playground/tree/master/bitwise_operations