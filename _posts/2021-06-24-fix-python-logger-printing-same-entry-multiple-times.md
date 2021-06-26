---
title: How to fix Python logger printing the same entry multiple times
date: 2021-06-23 23:00:00 +0100
categories: [Python, logging]
tags: [python, logging]
---

The [previous post][pp] explained how to get a simple colored formatter for your custom logger using the Python [`logging`] module. This one explains why a logger may print the same record multiple times, and how to fix this.

## The scenario

Let us suppose you already have the logger from the [previous post][pp] up and running. Now, the idea is to share it across several modules and/or classes. 

Suppose your logger is defined in `logger.py` as follows:

```python
import logging
import datetime


def get_logger():
    # Create custom logger logging all five levels 
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)

    # Define format for logs
    fmt = '%(asctime)s | %(levelname)8s | %(filename)s:%(lineno)2d | %(message)s'

    # Create stdout handler for logging to the console (logs all five levels)
    stdout_handler = logging.StreamHandler()
    stdout_handler.setLevel(logging.DEBUG)
    stdout_handler.setFormatter(CustomFormatter(fmt))

    # Create file handler for logging to a file (logs all five levels)
    today = datetime.date.today()
    file_handler = logging.FileHandler('my_app_{}.log'.format(today.strftime('%Y_%m_%d')))
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(logging.Formatter(fmt))

    # Add both handlers to the logger
    logger.addHandler(stdout_handler)
    logger.addHandler(file_handler)

    return logger
```

Suppose you also have a file called `module.py` where you define three classes, `A`, `B`, and `C`, each one using the same logger:

```python
from logger import get_logger


class A:
    def __init__(self):
        self.logger = get_logger()
        self.logger.warning('This is from class A')


class B:
    def __init__(self):
        self.logger = get_logger()
        self.logger.warning('This is from class B')


class C:
    def __init__(self):
        self.logger = get_logger()
        self.logger.warning('This is from class C')


def main():
    a = A()
    b = B()
    c = C()
    

if __name__ == '__main__':
    main()
```

## The problem

The problem is immediately apparent when running `module.py`:

```
2021-06-24 00:38:02,055 |  WARNING | module.py: 7 | This is from class A
2021-06-24 00:38:02,055 |  WARNING | module.py:13 | This is from class B
2021-06-24 00:38:02,055 |  WARNING | module.py:13 | This is from class B
2021-06-24 00:38:02,055 |  WARNING | module.py:19 | This is from class C
2021-06-24 00:38:02,055 |  WARNING | module.py:19 | This is from class C
2021-06-24 00:38:02,055 |  WARNING | module.py:19 | This is from class C
```

Instead of one log record for each of the three classes, we get one for class `A` as expected, but two for class `B` and three for class `C`. :confounded:

## Why does this happen?

The [`logging`] documentation ensures us that `logging.getLogger()` returns the same logger instance each time this function is called:

> All calls to this function with a given name return the same logger instance. This means that logger instances never need to be passed between different parts of an application.

So why does this happen? The answer is that although we get the same *logger*, each time we call our `get_logger()` function from `logger.py` we are actually attaching distinct _handlers_ to it. 

## Two solutions

Don't worry, there are two fixes for this problem. In both cases we get only one printed line per record, as expected:

```
2021-06-24 00:51:40,057 |  WARNING | module.py: 7 | This is from class A
2021-06-24 00:51:40,057 |  WARNING | module.py:13 | This is from class B
2021-06-24 00:51:40,057 |  WARNING | module.py:19 | This is from class C
```

### Import the same logger every time

We can simply create the logger in `logger.py` and import it directly in our module, without ever having to call `get_logger()`. Here is how `logger.py` changes:

```python
import logging
import datetime


def get_logger():
    ...
    return logger


logger = get_logger()
```

Everything is just as before, except that we create the logger in `logger.py` (it's a global variable, yes, I know).

And here is how `module.py` changes:

```python
from logger import logger


class A:
    def __init__(self):
        self.logger = logger
        self.logger.warning('This is from class A')


class B:
    def __init__(self):
        self.logger = logger
        self.logger.warning('This is from class B')


class C:
    def __init__(self):
        self.logger = logger
        self.logger.warning('This is from class C')


def main():
    a = A()
    b = B()
    c = C()


if __name__ == '__main__':
    main()
```
    
We just use the same variable in each of the three classes and the problem goes away, since no unnecessary handlers are created. However, this solution is not ideal since it involves (potentially many) modifications, and furthermore we can't be sure that in two months from now we'll still remember that we were not supposed to call `get_logger()` directly.


### Check if handlers are present
The best solution is to check whether any handlers are already attached before adding them to the logger. This fix only involves changing `logging.py`:

```python
import logging
import datetime


def get_logger():
    ...

     if not logger.hasHandlers():
        logger.addHandler(stdout_handler)
        logger.addHandler(file_handler)

    return logger

```

Problem fixed. Happy logging!


## Further reading

* [`logging`][] (Python documentation)
* [Python `logging` tutorial at Real Python][`log_tut`]

<!-- links -->

[`logging`]: https://docs.python.org/3/library/logging.html
[`log_tut`]: https://realpython.com/python-logging/
[pp]: {% post_url 2021-06-23-make-your-own-custom-color-formatter-with-python-logging %}
