---
title: How to read safely from stdin in C
date: 2018-12-11 00:00:00 +0100
categories: [C/C++, string]
tags: [c/c++, stdin]
---

Ah, C and strings. :confounded:

When reading from stdin, we can use [`fgets(str, n, stdin)`][`fgets()`] to read at most `n - 1` characters into a char array pointed to by `str`. We can also create a utility function to replace the newline character `\n` with the null terminator `\0`.

The following is inspired from [C Primer Plus][primer] by Stephen Prata, sixth edition, listing 11.10:

```c
#include <stdio.h>

// Read at most `n` characters (newline included) into `str`.
// If present, the newline is removed (replaced by the null terminator).
void s_gets(char* str, int n)
{
    char* str_read = fgets(str, n, stdin);
    if (!str_read)
        return;

    int i = 0;
    while (str[i] != '\n' && str[i] != '\0')
        i++;

    if (str[i] == '\n')
        str[i] = '\0';
}

int main()
{
    char my_string[10];
    s_gets(my_string, 10);
    printf("my_string = %s\n", my_string);
}
```

If we run the above program with the input `test 12345`, we obtain a char array of 9 characters followed by the null terminator `\0`:

```
my_string = test 1234
```

<!-- links -->
[`fgets()`]: https://en.cppreference.com/w/c/io/fgets
[primer]: https://www.amazon.com/dp/0321928423
