---
title: Distribute a Python package with a standalone script using setuptools
date: 2021-11-03 00:00:00 +0100
categories: [Python, packaging]
tags: [python, setuptools]
---

Suppose you've written a Python package that you want to be able to `pip install` locally. Additionally, you also want to be able to run one of the scripts in the package via its name, without the `.py` extension and without explicitly using the Python interpreter to launch it. In other words, you want to be able to run

```
standalone_script
```

instead of

```
python /path/to/my/package/standalone_script.py
```

This post explains how to achieve this using [`setuptools`][].

## A world without setuptools

Suppose the package is a simple one, containing only a readme file and a single module `core.py`. There is also going to be a `__init__.py` file detailing the imports from `core.py` that you would want to have available when you `import mypackage`. The starting project structure may look something like this:

```
mypackage/
    README.md
    __init__.py
    core.py
```

For the purpose of illustration, let us suppose that `core.py` contains a function `capitalize()` that takes a string and returns it in all caps (uppercase):

```python
# core.py

def capitalize(text):
    if not isinstance(text, str):
        raise ValueError('Need string to capitalize')
    return text.upper()
```

The `__init__.py` file imports the only thing it can, i.e. the `capitalize()` function:

```python
# __init__.py

from mypackage.core import capitalize
```

This way, each time you want to use the `mypackage` module, `capitalize()` becomes available. But you would need to go through the very tedious and error-prone approach of manually adding the path to `mypackage` every time you want to use it:

```python
# some_script.py

import sys
sys.path.append('/path/to/mypackage')
from mypackage import capitalize

print(capitalize('this'))
```

That's pretty lame, but fortunately we can do better.

## Enter setuptools

You can package the project and then install it via `pip` locally. Then any script that needs the newly installed package can simply `import` it:

```python
# some_script.py

from mypackage import capitalize

print(capitalize('this'))
```

In order to achieve this, only two steps are involved:
1. reorganize the project structure
1. create a `setup.py` file

For the project structure, simply create a subdirectory with the same name as the package name and move modules i.e. `.py` files inside it:

```
mypackage/
    README.md
    setup.py
    mypackage/
        __init__.py
        core.py
```

The `setup.py` file has the following contents:

```python
from setuptools import setup, find_packages

setup(
    name='mypackage',
    version='1.0.0',
    description='Capitalize strings',
    author='John Doe',
    author_email='doe@example.com',
    packages=find_packages(),
)
```

We can then install the package locally using `pip`:

```
pip install -e /path/to/mypackage
```

The `/path/to/mypackage` above refers to the *top-level* `mypackage/` directory.

The package may be uninstalled with:

```
pip uninstall mypackage
```

## Standalone script

What if we wanted a standalone script to be installed along with `mypackage` that would run the `capitalize()` function on any string that we pass through the command line? Here is how the script would be used:

```
$ capitalize
usage: capitalize [-h] [-v] [string [string ...]]
$ capitalize my text
MY TEXT
```

In order to achieve this, we need to:
1. create the script that runs the `capitalize()` function on the string that gets passed to it via the command line
1. edit `setup.py` to instruct it how to "install" the script (i.e. how to make it accessible system-wide)

We will be creating the standalone script in a file called `__main__.py` that we place in the subdirectory containing the other Python modules:

```
mypackage/
    README.md
    setup.py
    mypackage/
        __init__.py
        __main__.py
        core.py
```

Then we write the script in the `main()` method of `__main__.py`:

```python
import argparse
import sys
from mypackage import capitalize


def main():
    parser = argparse.ArgumentParser(prog='capitalize')
    parser.add_argument('string', nargs='*', help='string to capitalize')
    parser.add_argument('-v', '--version', help='display version', action='version',
                        version=f'%(prog)s 1.0.0')
    args = parser.parse_args()

    if args.string:
        text = ' '.join(word for word in args.string)
        print(capitalize(text))
    else:
        parser.print_usage()
        sys.exit(1)


if __name__ == '__main__':
    sys.exit(main())
```

At lines 7-11, we add an [argument parser][argparse]. It can either display the program version (through `-v` or `--version`) or consume all the command line arguments (`nargs='*'`) in order to pass them to the `capitalize()` function (lines 13-15).

The only thing left to do now is to point `setup.py` to the `main()` function of the `__main__.py` module and to ask it to add it as a console script "entry point" called `capitalize`:

```python
from setuptools import setup, find_packages

setup(
    name="mypackage",
    # [snip]
    entry_points = {'console_scripts': ['capitalize = mypackage.__main__:main']},
)
```

That's it! Now the package may be installed with `pip` as shown above and the `capitalize` script becomes available system-wide in the current Python environment. You might want to read the [next post][np] for a special tricky situation involving the use of the standalone script as a runner.

<!-- links -->

[`setuptools`]: https://setuptools.pypa.io/
[argparse]: https://docs.python.org/3/library/argparse.html
[np]: {% post_url 2021-11-04-standalone-python-script-to-run-other-python-scripts %}