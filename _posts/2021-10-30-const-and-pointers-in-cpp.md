---
title: Const and pointers in C++
date: 2021-10-30 00:00:00 +0100
categories: [C++, pointers]
tags: [c++, pointers, const]
---

The possible uses of the `const` qualifier with C++ pointers:
* pointer to const variable
* const pointer to variable
* const pointer to const variable

All examples below will be using the same `struct MyType` defined below:

```c++
struct MyType {
    int _x;
    MyType(int x): _x(x) {}
    friend std::ostream& operator<<(std::ostream& out, const MyType &my_type)
    {
        out << my_type._x;
        return out;
    }
};
```

We declare two variables of type `MyType` as follows:

```c++
MyType a{1};
MyType b{2};
```


## Pointer to const variable

When using a pointer to a `const` variable, we cannot change the value of the variable pointed to by the pointer. However, we can reassign the pointer itself:

```c++
// pointer to const variable
const MyType *ptr1 = &a;
std::cout << "ptr1: " << *ptr1 << std::endl;
// ptr1->_x = 33; // error
ptr1 = &b;
std::cout << "ptr1: " << *ptr1 << std::endl;
```

Output:

```
ptr1: 1
ptr1: 2
```


## Const pointer to variable

When using a `const` pointer to a variable, we can change the value of the variable pointed to by the pointer. However, we cannot reassign the pointer. It is the opposite of the situation above:

```c++
// const pointer to variable
MyType* const ptr2 = &a;
std::cout << "ptr2: " << *ptr2 << std::endl;
ptr2->_x = 10;
std::cout << "ptr2: " << *ptr2 << std::endl;
// ptr2 = &b; // error
```

Output:

```
ptr2: 1
ptr2: 10
```


## Const pointer to const variable

When using a `const` pointer to a `const` variable, we can neither change the value of the variable pointed by the pointer, nor reassign the pointer:

```c++
// const pointer to const variable
const MyType* const ptr3 = &b;
std::cout << "ptr3: " << *ptr3 << std::endl;
// ptr3->_x = 33; // error
// ptr3 = &a; // error
```

Output:

```
ptr3: 2
```


## Conclusion

Here is the recap in table form:

|                 | variable                                        | `const` variable                                  |
| pointer         | can change value;<br />can reassign pointer     | cannot change value;<br />can reassign pointer    |
| `const` pointer | can change value;<br /> cannot reassign pointer | cannot change value;<br />cannot reassign pointer |