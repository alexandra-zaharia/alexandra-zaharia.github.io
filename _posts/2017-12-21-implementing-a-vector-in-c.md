---
title: Implementing a vector in C
date: 2017-12-21 00:00:00 +0100
categories: [C, data structures]
tags: [c, vector]
math: true
---

Suppose we need a generic vector data structure in C, where by *generic* we mean it can handle any type of data. A vector uses an underlying array, therefore it supports index-based access to its elements. Moreover, the underlying array is resizable, meaning that memory space is not wasted uselessly. If the vector is full, adding a new element causes the underlying array to double its size. If the vector is 75% empty, the underlying array halves its size.

## How does a vector work?

The ASCII figure below shows the example of a vector *v*, initially empty and with an initial
capacity of 4 items:

```
                          -----------------
when v is created :       |   |   |   |   |
                          -----------------

                          -----------------
append 7 to v :           | 7 |   |   |   |
                          -----------------

                          -----------------
append 1 to v :           | 7 | 1 |   |   |
                          -----------------           

                          -----------------
append 5 to v :           | 7 | 1 | 5 |   |
                          -----------------           

                          -----------------
append 9 to v :           | 7 | 1 | 5 | 9 |
                          -----------------

                            0   1   2   3   4   5   6   7
                          ---------------------------------
insert 2 at index 3 :     | 7 | 1 | 5 | 2 | 9 |   |   |   |   <-- capacity doubles   
                          ---------------------------------

                            0   1   2   3   4   5   6   7
                          ---------------------------------
insert 2 at index 3 :     | 7 | 1 | 5 | 2 | 9 |   |   |   |   
                          ---------------------------------

                            0   1   2   3   4   5   6   7
                          ---------------------------------
remove item at index 3 :  | 7 | 1 | 5 | 9 |   |   |   |   |   
                          ---------------------------------

                            0   1   2   3   4   5   6   7
                          ---------------------------------
remove item at index 1 :  | 7 | 5 | 9 |   |   |   |   |   |   
                          ---------------------------------

                            0   1   2   3  
                          -----------------
remove item at index 0 :  | 5 | 9 |   |   |   <-- capacity is reduced by half since 75% empty
                          -----------------
```

## Comparison with alternative data structures

A **fixed-size array** allows access to its elements in \\( O(1) \\) time. Adding items at the end of the array also takes \\( O(1) \\) time. However, insertion (other than at the end of the array) and deletion require \\( O(n) \\) time. As its name implies, a fixed-size array cannot change its size. 

A **vector** uses an underlying *resizing* array, meaning that its capacity is automatically adjusted to accommodate its elements.

In contrast to an array, a **linked list** allows insertion and deletion in \\( O(1) \\) time, but accessing the *k*<sup>th</sup> element requires \\( O(n) \\) time.

An array (a fixed-size array or a resizing array, i.e. a vector) should be used when indexing happens more often than insertion or deletion at arbitrary positions. A linked list is more appropriate when indexing happens rarely and when insertions and deletions are frequent.

## Resizing the vector

A vector starts out with an initial capacity, for which we can make an educated guess depending on the application. Let us suppose a good choice for an initial capacity is 4. 

When the 5<sup>th</sup> item is added to the vector, its capacity doubles, becoming 8. When the 9<sup>th</sup> item is added, the capacity doubles again, becoming 16, and so on. **Doubling the vector capacity** is thus performed only if it is absolutely necessary to do so.

**Halving the vector capacity** is more tricky. The aim is to strike a good balance between array resizing operations performed via `realloc()` and not wasting too much memory space. Suppose a vector has 9 items and its capacity is 16. If we remove one item, it would be tempting to halve the capacity on the spot. But if a 9<sup>th</sup> item needs to be added right away, we would have to double the capacity yet again. The best bet is to halve the vector capacity when it is one quarter full, because this means we have been removing items from the vector for quite some time and it is reasonable to assume that the need to double the capacity will not arise any time soon. Continuing the example, a vector would keep its capacity of 16 items as long as it has at least 5 items. When it only has 4 items, its capacity becomes 8. Consider another example of a vector with 513 elements and capacity 1024. If we start removing items, the capacity shrinks to 512 when only 256 items remain. The capacity further shrinks to 256 when only 128 items remain, and so on.

## Vector definition

Here is a bare-bones definition of a vector in C:

```c
typedef struct {
    void **data;     /* information stored in the vector */
    size_t count;    /* number of elements currently stored in the vector */
    size_t capacity; /* maximum capacity of the vector */
} Vector;
```

We want this data structure to be generic, meaning it must be able to handle any type of item: integers, doubles, strings, as well as user-defined data structures. This is why items in a `Vector` are of type `void *`. As vector items are organized in an array, what we need for the vector `data` is a pointer to pointer to void (`void **`).

We should also define the initial capacity of the vector. Note however that this choice is highly dependent on the application. Here, we will use an initial capacity of 4:

```c
const size_t VECTOR_INIT_CAPACITY = 4;
```

The `Vector` implementation in [libgcds][] (**Lib**rary for **G**eneric **C D**ata **S**tructures) defines the data structure as above, with the addition of several function pointers (`struct`s with function pointers are the [ancestors of classes][cpp]; for the C ecosystem, it will just have to do). In this post the aim is to keep things simple and easy to understand, not to build a library, so we will just be defining stand-alone ("normal") functions for the `Vector` API.

[libgcds]: https://github.com/alexandra-zaharia/libgcds
[cpp]: https://www.amazon.com/Design-Evolution-C-Bjarne-Stroustrup/dp/0201543303

## A basic vector API

Any basic `Vector` API should have the following methods:

* A method to create a vector: `Vector *vector_create()` creates a `Vector` and returns a pointer to it, or the `NULL` pointer in case of failure.
* A method to free a vector: `void vector_free(Vector *vector)` frees the specified `vector`.
* A method to add an item at the end of a vector: `int vector_add(Vector *vector, void *item)` attempts to add the given `item` at the end of the `vector`, doubling the size of the underlying array if necessary. Returns 0 for success and -1 for failure. 
* A method to insert an item at an arbitrary position: `int vector_insert(Vector *vector, void *item, int index)` attempts to insert the given `item` at a specified `index` in the `vector`, doubling the size of the underlying array if necessary. Returns 0 for success and -1 for failure. 
* A method to delete an item at an arbitrary position: `int vector_delete(Vector *vector, int index)` attempts to delete the `item` at the specified `index` in the `vector`, halving the size of the underlying array if necessary. Returns 0 for success and -1 for failure. 

In the remainder of this section we will implement each of these methods in turn.

### vector_create()

We start by allocating memory for a vector and return `NULL` if the allocation fails. Then we initialize the number of elements to 0 and the capacity to the initial capacity. We must also allocate memory for the underlying array `vector->data`. If this is not possible, we free the vector pointer and return `NULL`. If everything went fine, the function returns a pointer to the brand new vector.

```c
Vector *vector_create()
{
    Vector *vector = (Vector *)malloc(sizeof(Vector));
    if (!vector) return NULL;

    vector->count = 0;
    vector->capacity = VECTOR_INIT_CAPACITY;

    vector->data = (void *)malloc(vector->capacity * sizeof(void *));
    if (!vector->data) {
        vector_free(vector);
        return NULL;
    }

    return vector;
}
```

### vector_free()

If the pointer to `Vector` is not `NULL`, we attempt to deallocate its data, then the vector itself.

```c
void vector_free(Vector *vector)
{
    if (vector) {
        if (vector->data)
            free(vector->data);
        free(vector);
    }
}
```

### _vector_resize()

Yes, `_vector_resize()` is not listed above. The reason for this is that this method is not part of the public API, but it is required for methods that may need to resize the underlying array: `vector_add()`, `vector_insert()` and `vector_delete()`. The client of the `Vector` API does not even need to know that this function exists. In order to keep it *private* to the implementation file of the vector (the `.c` file), we will be declaring it `static`.

The function starts out by attempting to reallocate the vector's underlying array `data` with the new `capacity`. 

> **Note**: We use a new `void **` pointer for this reallocation. This is important, because `realloc()` is not guaranteed to return a pointer to the memory location occupied by the array to be resized. 

```c
static int _vector_resize(Vector *vector, size_t capacity)
{
    void **data = realloc(vector->data, capacity * sizeof(void *));
    vector->capacity = capacity;

    if (!data) return -1;
    if (data != vector->data)
        vector->data = data;
    data = NULL;

    return 0;
}
```

### vector_add()
Adding an item at the end of a vector can fail if the vector or its data is `NULL`, or if the resizing is unsuccessful. Resizing the underlying array is performed if there is no free slot in the vector to add the new item.

```c
int vector_add(Vector *vector, void *item)
{
    if (!vector || !vector->data) return -1;

    if (vector->count == vector->capacity) {
        if (_vector_resize(vector, 2 * vector->capacity) == -1)
            return -1;
    }

    vector->data[vector->count++] = item;

    return 0;
}
```

### vector_insert()

Inserting an item at an arbitrary position in a vector can fail if the vector or its data is `NULL`, if the index is incorrect, or if the resizing is unsuccessful. As for `vector_add()`, resizing is performed if there is no free slot for the new item. In addition, every item in the vector after the position designated by `index` must be shifted by one position to the right. A special case is identified where insertion takes place at the end of the vector, in which case `vector_add()` is used directly. As we've seen above, `vector_add()` may also fail, so a check for its return code is equally performed.

```c
int vector_insert(Vector *vector, void *item, int index)
{
    if (!vector || !vector->data) return -1;
    if (index < 0 || index > vector->count) return -1;

    if (index == vector->count) {
        if (vector_add(vector, item) == -1) return -1;
    } else {
        if (vector->count == vector->capacity) {
            if (_vector_resize(vector, 2 * vector->capacity) == -1)
                return -1;
        }

        for (int i = vector->count; i > index; i--)
            vector->data[i] = vector->data[i-1];

        vector->data[index] = item;
        vector->count++;
    }

    return 0;
}
```

### vector_delete()

Deleting an item at an arbitrary position in a vector can fail if the vector or its data is `NULL`, if the index is incorrect, or if the resizing is unsuccessful. Resizing the underlying array to half its capacity is performed if the vector is one quarter full after deletion. Every item in the vector after the position designated by `index` must be shifted by one position to the left.

```c
int vector_delete(Vector *vector, int index)
{
    if (!vector || !vector->data) return -1;
    if (index < 0 || index >= vector->count) return -1;

    vector->count--;

    for (int i = index; i < vector->count; i++)
        vector->data[i] = vector->data[i+1];

    if (vector->count == vector->capacity / 4
            && vector->capacity > VECTOR_INIT_CAPACITY) {
        if (_vector_resize(vector, vector->capacity / 2) == -1)
            return -1;
    }

    return 0;
}
```

## An improved Vector API

For all practical purposes, there is a high chance that the basic API we have just seen above is not sufficient. As the need arises, we may add several useful functions to our `Vector` API, such as:

* A method to check whether the vector contains a given item: `bool vector_contains(Vector* vector, void* item)` determines whether the `vector` contains the specified `item`. Returns `true` if the item exists, `false` otherwise.
* A method to determine the index of a given item in the vector: `int vector_index(Vector* vector, void* item)` determines the index of the specified `item` in the `vector`, or -1 if no such item exists.

> **Note**: In both cases that we do not actually examine the _value_ of the item, since at this point we cannot even know what kind of items we are dealing with. We simply compare void pointers, which enables us to determine whether the given item exists in the vector's `data` array.

### vector_contains()

If the vector is not `NULL`, we iterate its `data` array and compare its every item against the specified one.

```c
bool vector_contains(Vector* vector, void* item)
{
    if (!vector) return false;
    for (unsigned int i = 0; i < vector->size; i++)
        if (vector->data[i] == item)
            return true;
    return false;
}
```

### vector_index()

If the vector is not `NULL`, we iterate its `data` array until we find the specified item or until we hit the end of the array. 
```c
int vector_index(Vector* vector, void* item)
{
    if (!vector) return -1;

    for (unsigned int i = 0; i < vector->size; i++) {
        if (vector->data[i] == item)
            return (int) i;
    }

    return -1;
}
```

## Examples of vectors

Here we will see how two clients may use the `Vector` API that we have just examined: one client creates a vector of integers, and the other one creates a vector of user-defined data structures.

### Example #1: A vector of integers

Suppose we need a vector to handle integers. First, values 2, 4 and 6 are added at the end of the vector using `vector_add()`. The vector is `[ 2 4 6 ]`. Second, the values 1, 3 and 5 are inserted in between using `vector_insert()`, such that the vector becomes `[ 1 2 3 4 5 6 ]`. Finally, the last three values are deleted from the vector using `vector_delete()`. The vector is now `[ 1 2 3 ]`. The following program details these steps:

```c
#include "vector.h"
#include <stdio.h>
#include <stdlib.h>

void vector_int_print(Vector *vector);

int main(void)
{
    int i;
    Vector *vector = vector_create();
    if (!vector) {
        fprintf(stderr, "Cannot allocate vector.\n");
        exit(EXIT_FAILURE);
    }

    int even[3] = {2, 4, 6};
    for (i = 0; i < 3; i++)
        vector_add(vector, &even[i]);
    vector_int_print(vector); /* [ 2 4 6 ] */

    int odd[3] = {1, 3, 5};
    vector_insert(vector, &odd[0], 0);
    vector_insert(vector, &odd[1], 2);
    vector_insert(vector, &odd[2], 4);
    vector_int_print(vector); /* [ 1 2 3 4 5 6 ] */

    for (i = 5; i > 2; i--)
        vector_delete(vector, i);
    vector_int_print(vector); /* [ 1 2 3 ] */

    vector_free(vector);

    return EXIT_SUCCESS;
}

void vector_int_print(Vector *vector)
{
    printf("[ ");
    for (size_t i = 0; i < vector->count; i++)
        printf("%d ", *((int *) vector->data[i]));
    printf("]\n");
}
```

> **Note**: Checks for return codes from `vector_add()`, `vector_insert()` and `vector_delete()` should be performed, but have been omitted here for the sake of brevity.

### Example #2: A vector of user-defined data structures

Suppose we now need a vector to handle a user-defined data structure representing 2D points. We first add the points with coordinates (1, 10) and (3, 30) to the vector using `vector_add()`. We then insert the point (2, 20) in between using `vector_insert()`. The following program details these steps:

```c
#include "vector.h"
#include <stdio.h>
#include <stdlib.h>

void vector_point_print(Vector *vector);

typedef struct {
    int x;
    int y;
} Point;

int main(void)
{
    Vector *vector = vector_create();
    if (!vector) {
        fprintf(stderr, "Cannot allocate vector.\n");
        exit(EXIT_FAILURE);
    }

    Point p1 = {.x = 1, .y = 10};
    Point p2 = {.x = 2, .y = 20};
    Point p3 = {.x = 3, .y = 30};
       
    vector_add(vector, &p1);
    vector_add(vector, &p3);
    vector_insert(vector, &p2, 1);

    vector_point_print(vector); /* [ (1, 10) (2, 20) (3, 30) ] */
    vector_free(vector);

    return EXIT_SUCCESS;
}

void vector_point_print(Vector *vector)
{
    printf("[ ");
    for (size_t i = 0; i < vector->count; i++) {
        Point *point = (Point *) vector->data[i];
        printf("(%d, %d) ", point->x, point->y);
    }
    printf("]\n");
}
```

> **Note**: Checks for return codes from `vector_add()` and `vector_insert()` should be performed, but have been omitted here for the sake of brevity.

## Testing the implementation
This `Vector` implementation has been extensively tested using the [cmocka](https://cmocka.org/) testing framework. The tests are provided in the file [vector-test.c]. 

[vector-test.c]: https://github.com/alexandra-zaharia/libgcds/blob/master/tests/vector-test.c

Install cmocka first. Then, to build the tests, run:

```bash
gcc -Wall vector-test.c vector.c -g -o vector-test -lcmocka
```

To execute the tests, run:

```bash
./test-vector
```

To execute the tests using [valgrind](https://www.valgrind.org/) in order to detect memory leaks, run:

```bash
valgrind --leak-check=full ./test-vector
```

There should be no memory leaks :-)

## Availability

The full `Vector` implementation along with the tests is available on [GitHub][libgcds].