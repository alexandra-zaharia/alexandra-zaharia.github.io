---
title: Remove every occurrence of an item from a list in Python
date: 2021-09-20 00:00:00 +0100
categories: [Python, data structures]
tags: [python, occurrence]
---

A problem that I come across quite often is having to remove every occurrence of a given item from a Python list. While [built-in mutable types][list] have a `remove()` method, `my_list.remove(x)` only removes the first occurrence of item `x` from `my_list`.

Here is the pattern I use for removing every occurrence of a given item from a list:

```python
def remove_all(items, item_to_remove):
    if not isinstance(items, list):
        raise TypeError(f'invalid list type {type(items).__name__}')

    last_occurrence = False
    while not last_occurrence:
        try:
            items.remove(item_to_remove)
        except ValueError:
            last_occurrence = True
```

The documentation states that `remove()` raises a `ValueError` when the item to remove is not found in the list.

In `remove_all()`, we take advantage of this. When a `ValueError` is raised, it means the item is no longer in the list or, in other words, that we have managed to remove every occurrence of that item from the list.

Example:

```python
In [217]: items = [1, 2, 3, 1, 4, 5, 1, 6, 1, 7]

In [218]: remove_all(items, 1)

In [219]: items
Out[219]: [2, 3, 4, 5, 6, 7]
```

As can be seen above, `remove_all()` changes the input list `items`.


<!-- links -->
[list]: https://docs.python.org/3/library/stdtypes.html#mutable-sequence-types