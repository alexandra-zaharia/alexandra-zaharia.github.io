---
title: How to return a result from a Python thread
date: 2021-08-24 00:00:00 +0100
categories: [Python, thread, timeout]
tags: [python, threads]
---

## The problem

Suppose you have a Python thread that runs your target function.

* Simple scenario: That target function returns a result that you want to retrieve.
* A more advanced scenario: You want to retrieve the result of the target function if the thread does not time out.

There are several ways to retrieve a value from a Python thread. You can use [`concurrent.futures`][], [`multiprocessing.pool.ThreadPool`][] or just [`threading`][] with [`Queue`].

This post proposes an alternative solution that does not require any other package aside from `threading`.

## The solution

If you don't want to use anything else beside the `threading` module, the solution is simple:

1. Extend the `threading.Thread` class and add a `result` member to your new class. Make sure to take into account positional and keyword arguments in the constructor.
1. Override the base class's `run()` method: in addition to running the target function as expected (with its args and kwargs intact), it has to store the target's result in the new member `result`.
1. Override the base class's `join()` method: with args and kwargs intact, simply `join()` as in the base class but also return the result.
1. Then when you instantiate your new thread class, intercept the result returned by `join()`.

Note the stress placed upon preserving the target's positional and keyword arguments: this ensures that you can also `join()` the thread with a timeout, as you would a with a `threading.Thread` instance.

The following section illustrates these steps.

## Implementation: ReturnValueThread class

Below, the class `ReturnValueThread` extends `threading.Thread` (lines 4-19).

1. In the constructor, we declare a `result` member that will store the result returned by the target function (lines 6-8).
1. We override the `run()` method by storing the result of the target in the `result` member (lines 10-16).
1. We override the `join()` method such as to return the `result` member (lines 18-20).

```python
import threading
import sys


class ReturnValueThread(threading.Thread):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.result = None

    def run(self):
        if self._target is None:
            return  # could alternatively raise an exception, depends on the use case
        try:
            self.result = self._target(*self._args, **self._kwargs)
        except Exception as exc:
            print(f'{type(exc).__name__}: {exc}', file=sys.stderr)  # properly handle the exception

    def join(self, *args, **kwargs):
        super().join(*args, **kwargs)
        return self.result
```

## Usage example for ReturnValueThread

Here is how to use the `ReturnValueThread` class defined above. Imagine that the target functions both compute and return the square of the argument that gets passed to them:

* `square()` returns the square of its argument instantly (lines 4-5);
* `think_about_square()` returns the square of its argument after having... thought about it for a while (lines 8-10).

Why do we have two target functions in this example? Remember the scenarios mentioned at the beginning of this post:

1. A simple scenario is to simply retrieve the value returned by the target function (lines 16-19);
1. A more advanced scenario is to retrieve the value if the function finishes running before a specified timeout (lines 21-27).

```python
import time


def square(x):
    return x ** 2


def think_about_square(x):
    time.sleep(x)
    return square(x)


def main():
    value = 3

    thread1 = ReturnValueThread(target=square, args=(value,))
    thread1.start()
    result = thread1.join()
    print(f'square({value}) = {result}')

    thread2 = ReturnValueThread(target=think_about_square, args=(value,))
    thread2.start()
    result = thread2.join(timeout=1)
    if thread2.is_alive():
        print('Timeout in think_about_square')  # properly handle timeout
    else:
        print(f'think_about_square({value}) = {result}')


if __name__ == '__main__':
    main()
```

`thread1` is the thread running `square()` (instant result, retrieved as expected). `thread2`, on the other hand, runs `think_about_square()`, and it just so happens that it does not finish within the allotted time. We test whether the thread finished at line 24 via `thread2.is_alive()`.

## Caveat

The more observant types have probably noticed that although `ReturnValueThread` returns the result of the target function, our `thread2` in the above example (the thread that times out) does not exit cleanly. In fact, it runs until the `sleep()` ends. In a [previous post][pp] we have seen how to exit a Python thread cleanly. Another solution is to use a process instead of a thread, but this comes with its own set of complications. The most notable difficulty is the fact that, unlike threads, processes run in separate memory spaces.

## Further reading

* [How to exit a Python thread cleanly][pp] (using a threading event)
* [`concurrent.futures`] (Python documentation)
* [`multiprocessing.pool.ThreadPool`] (Python documentation)
* [`threading`] (Python documentation)
* [`Queue`] (Python documentation)

<!-- links -->

[pp]: {% post_url 2021-07-06-how-to-stop-a-python-thread-cleanly %}
[`concurrent.futures`]: https://docs.python.org/3/library/concurrent.futures.html
[`multiprocessing.pool.ThreadPool`]: https://docs.python.org/3/library/multiprocessing.html#module-multiprocessing.dummy
[`threading`]: https://docs.python.org/3/library/threading.html
[`Queue`]: https://docs.python.org/3/library/queue.html
