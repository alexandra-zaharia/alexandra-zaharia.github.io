---
title: The overloaded insertion operator in C++ exceptions
date: 2021-11-14 00:00:00 +0100
categories: [C++, operators]
tags: [c++, exception, stringstream]
---

The `<<` operator, when applied to streams, is called the **insertion** operator. (Likewise, the extraction operator is `>>`). If we overload it for a class (as non-member), we can output instances of that class. That's very handy.

Note that in order for the overloaded `operator<<()` method to have access to private members of the class, we need to declare it as `friend`.

Take the very basic example of 2D coordinates:

```c++
#include <iostream>

class Coords {
    uint16_t _x;
    uint16_t _y;
public:
    Coords(uint16_t x, uint16_t y): _x(x), _y(y)
    {

    }
    friend std::ostream& operator<<(std::ostream& out, const Coords& coords)
    {
        out << "(" << coords._x << ", " << coords._y << ")";
        return out;
    }
};

int main()
{
    Coords c(0, 1);
    std::cout << c << std::endl;
}
```

This produces, as expected:

```
(0, 1)
```

The problem with `operator<<()` (through my Python-tinted glasses, of course) is that it cannot unfortunately be used directly to create a custom exception description :confused:

We must use a `stringstream` instead:

```c++
#include <exception>
#include <sstream>

class Coords {
    // ...
    void move_left()
    {
        if (_x == 0) {
            std::stringstream ss;
            ss << "Coords: cannot move left from " << *this;
            throw std::invalid_argument(ss.str());
        }
    }
};
```

This produces:

```
Coords: cannot move left from (0, 1)
```