---
title: std::map with pointers as keys
date: 2021-10-31 00:00:00 +0100
categories: [C/C++, pointers]
tags: [c/c++, pointers, stl, map]
---

When the keys in a `std::map` are pointers, `find()` will fail unless the search query is one of the actual pointers in the map. However, in most cases this might not be exactly what we want to achieve.

Consider the following example: we want to map coordinates to planets at those coordinates using `std::map`. Here are the `Coords` and `Planet` classes:

```c++
#include <iostream>
#include <map>

class Coords {
    uint16_t _x;
    uint16_t _y;
public:
    Coords(uint16_t x, uint16_t y)
        : _x(x), _y(y) { }
    friend std::ostream& operator<<(std::ostream& out, const Coords& coords)
    {
        out << "(" << coords._x << ", " << coords._y << ")";
        return out;
    }
};

class Planet {
    const Coords* _coords;
    std::string _name;
public:
    Planet(const Coords* coords, std::string name)
        : _coords(coords), _name(name) { }
    const Coords* get_coords() const { return _coords; }
    friend std::ostream& operator<<(std::ostream& out, const Planet& planet)
    {
        out << "Planet " << planet._name << " @ " << *planet._coords;
        return out;
    }
};
```

Let's create a map and add some coordinates and planets to it:

```c++
Coords c11{1, 1};
Coords c22{2, 2};
Coords c33{3, 3};
Coords c44{4, 4};
Planet proxima{&c11, "Proxima"};
Planet arrakis{&c22, "Arrakis"};
Planet remulak{&c33, "Remulak"};
Planet regulus{&c44, "Regulus"};

std::map<const Coords *, Planet *> planet_map;
planet_map.insert(std::make_pair(proxima.get_coords(), &proxima));
planet_map.insert(std::make_pair(arrakis.get_coords(), &arrakis));
planet_map.insert(std::make_pair(remulak.get_coords(), &remulak));
planet_map.insert(std::make_pair(regulus.get_coords(), &regulus));

for (auto it = planet_map.begin(); it != planet_map.end(); ++it) {
    std::cout << *it->first << ": " << *it->second << std::endl;
}
```

We obtain:

```
(1, 1): Planet Proxima @ (1, 1)
(2, 2): Planet Arrakis @ (2, 2)
(3, 3): Planet Remulak @ (3, 3)
(4, 4): Planet Regulus @ (4, 4)
```

## The problem

We can also search for (`const Coords *`, `Planet *`) pairs *if we use one of the already existing* `Coords` objects:

```c++
auto it = planet_map.find(&c33);
if (it == planet_map.end()) {
    std::cout << "Coordinates " << query << " are not present in the map" << std::endl;
} else {
    std::cout << *it->first << ": " << *it->second << std::endl;
}
```

As expected, we obtain:

```
(3, 3): Planet Remulak @ (3, 3)
```

But watch what happens if we use another object created through the default `Coords` copy constructor:

```c++
Coords query = c33;
auto it = planet_map.find(&query);
if (it == planet_map.end()) {
    std::cout << "Coordinates " << query << " are not present in the map" << std::endl;
} else {
    std::cout << *it->first << ": " << *it->second << std::endl;
}
```

This is the same code as above, except that now we declare a new `Coords` variable `query` to which we assign the `c33` variable. Internally, the assignment uses the `Coords` copy constructor `Coords(const Coords& coords)`. In this case, the query is not found:

```
Coordinates (3, 3) are not present in the map
```

This happens because, remember, the keys in the `std::map` are pointers to (`const`) `Coord` objects, and what `find()` does is try to match the pointers in the map against the pointer we pass to it (`&query` in our example).

## The solution

We need a way to tell `find()` to compare the actual coordinates, and *not* the `Coords` pointers. There are two steps involved:
1. We need to add custom logic to the coordinate comparison: in technical terms, we need to overload `operator<` for the `Coords` class;
1. We need to create the `std::map` using a custom comparator. There are actually two ways to do this, we'll get there in just a minute.

### Overload the less-than operator

Here we just need to define how two coordinates (*x<sub>1</sub>*, *y<sub>1</sub>*) and (*x<sub>2</sub>*, *y<sub>2</sub>*) are compared, returning `true` if (*x<sub>1</sub>*, *y<sub>1</sub>*) is less than (*x<sub>2</sub>*, *y<sub>2</sub>*) and `false` otherwise. In order to do this, we add the following member method to the `Coords` class:

```c++
bool operator<(const Coords& other) const
{
    if (_x == other._x) return _y < other._y;
    return _x < other._x;
}
```

### Custom comparator for the std::map

The comparator is the function object or predicate that gets to use our shiny new `operator<` overload. As hinted here and as mentioned above, there are two ways to do this:
11. We can use a function object, basically a `struct` with an overloaded `operator()`;
11. We can use a lambda expression.

#### Using a function object

We can define a `struct CmpCoordsFunctor` overloading its `operator()`.

```c++
struct CmpCoordsFunctor {
    bool operator()(const Coords *lhs, const Coords *rhs) const {
        return *lhs < *rhs;
    }
};
```

A *functor* is simply a function object i.e. a class or struct that overloads `operator()`. When creating the `std::map`, we add `CmpCoordsFunctor` as comparator as follows:

```c++
std::map<const Coords *, Planet *, CmpCoordsFunctor> planet_map;
```

#### Using a lambda expression

Instead of using a functor as we've seen above, we can also use a lambda expression to achieve the same result:

```c++
auto CmpCoordsLambda = [](const Coords *lhs, const Coords *rhs) {
    return *lhs < *rhs;
};
std::map<const Coords *, Planet *, decltype(CmpCoordsLambda)> planet_map(CmpCoordsLambda);
```

### The end result

Regardless of the method used for adding the custom comparator (function object or lambda expression), the end result is the same. We can now search whether a given set of coordinates is present in the map by only examining the *value* of the coordinates, not the pointer itself. With the same `query` variable that was used at the beginning of this post, we now obtain the correct answer:

```
(3, 3): Planet Remulak @ (3, 3)
```

That's it, happy planet mapping! :stars::telescope: