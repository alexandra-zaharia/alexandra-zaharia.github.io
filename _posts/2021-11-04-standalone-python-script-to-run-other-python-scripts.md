---
title: Standalone Python script to run other Python scripts
date: 2021-11-04 00:00:00 +0100
categories: [Python, packaging]
tags: [python, exec, runpy]
---

In the [previous post][pp] we've seen how to use [`setuptools`][] to package a Python project along with a standalone executable that can be invoked on the command-line, system-wide (or rather _environment_-wide).

Now, what if you need that standalone script to be a **runner**, sort of like a master script running other Python scripts that import modules from the newly installed package?

Although it seems straightforward, there might be some issues with getting the Python scripts to actually run, as my recent [Stack Overflow][so] experience has shown.

## What we want to obtain

Here is how a script using your package might look like:

```python
# my_script.py
from mypackage import capitalize

print(f'Running {__file__}')
print(capitalize('my text'))
print(f'Done running {__file__}')
```

You would invoke your standalone script, let's call it `runner`, as follows:

```
runner my_script.py
```

And you'd obtain:

```
runner v1.0.0 started on 2021-11-04 00:11:26
Running my_script.py
MY TEXT
Done running my_script.py
runner v1.0.0 finished on 2021-11-04 00:11:26
```

## Making the necessary adjustments

We start off by editing `setup.py` to which we add a new console script entry point. There is no need to have the function pointed at by the entry point to reside in a different file than the `__main__.py` that we already have:

```python
from setuptools import setup, find_packages

setup(
    name="mypackage",
    # [snip]
    entry_points = {'console_scripts': [
        'capitalize = mypackage.__main__:main',
        'runner = mypackage.__main__:runner',  # this gets added
    ]},
)
```

The `__main__.py` file gets a new `runner()` function:

```python
import argparse
from datetime import datetime
import sys


def now():
    return datetime.now().strftime('%Y-%m-%d %H:%m:%S')


def runner():
    version = '1.0.0'
    parser = argparse.ArgumentParser(prog='runner')
    parser.add_argument('script', help='Python script to run')
    parser.add_argument('-v', '--version', help='display version', action='version',
                        version=f'%(prog)s {version}')
    args = parser.parse_args()

    if args.script:
        print(f'{parser.prog} v{version} started on {now()}')
        exec(open(args.script).read())
        print(f'{parser.prog} v{version} finished on {now()}')
    else:
        parser.print_usage()
        sys.exit(1)
```

## But does it work?

However, in the cases I've tested, this will not work. The `exec()` call does not seem to have any effect. One way to deal with this is to compile the script specified via command-line and execute the resulting code:

```python
import os
...
def runner():
    ...
    # exec(open(args.script).read())
    exec(compile(open(args.script).read(), os.path.basename(args.script), 'exec'))
```

While this _does_ work, it has the disadvantage that the `__file__` builtin of the executed script gets overwritten. You might get something like this:

```
runner v1.0.0 started on 2021-11-04 00:11:26
Running /full/path/to/mypackage/mypackage/__main__.py
MY TEXT
Done running /full/path/to/mypackage/mypackage/__main__.py
runner v1.0.0 finished on 2021-11-04 00:11:26
```

## The solution

There is still hope, thanks to the [`runpy`][] module in the standard library:

```python
...
def runner():
    ...
    # exec(open(args.script).read())
    # exec(compile(open(args.script).read(), os.path.basename(args.script), 'exec'))
    argparse.Namespace(**runpy.run_path(args.script))
```

Now we finally get the expected output, with `__file__` in `my_script.py` not being overwritten since `runpy` takes care to set it along with several other special global variables before `exec()`ing the code.

## Accompanying code

The full code accompanying this post can be found on my [GitHub][] repository.

<!-- links -->
[pp]: {% post_url 2021-11-03-distribute-a-python-package-with-a-standalone-script-using-setuptools %}
[`setuptools`]: https://setuptools.pypa.io/
[so]: https://stackoverflow.com/questions/69830933
[`runpy`]: https://docs.python.org/3/library/runpy.html
[GitHub]: https://github.com/alexandra-zaharia/python-playground/tree/main/packaging_a_standalone_runner
