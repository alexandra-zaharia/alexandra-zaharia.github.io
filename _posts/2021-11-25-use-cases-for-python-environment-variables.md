---
title: Use cases for Python environment variables
date: 2021-11-25 00:00:00 +0100
categories: [Python, configuration]
tags: [python, environment, software design]
---

Today's post focuses on environment variables in Python. They are one of several possible mechanisms for setting various configuration parameters. We can:

* read environment variables (through [`os.environ`][`os`] or [`dotenv`][]) [the current post]
* have the script accept command-line arguments (use [`argparse`][])
* load configuration settings from a file, such as:
    * a JSON file (use [`json`][])
    * a YAML file (use [`pyyaml`][])
    * a XML file (use [`lxml`][], [`ElementTree`][] or [`minidom`][])
    * an INI file (use [`configparser`][]) [check out [this post][pp-data]]
    * your DIY file format (for which you will be rolling your own parser)


## What is the best solution?

The answer is... *it depends*.

There is no one-size-fits-all solution. It depends on what you're trying to achieve and on how the current software architecture looks like. If you're working on a command-line tool that must accommodate a plethora of options, chances are you'll be using `argparse`. For other types of projects (such as a server or a client), a configuration file might be more practical. Yet in other situations you may also want to consider using environment variables.

We will be looking in more detail at three such use cases in this post, where we will see how environment variables can be a good choice.

But first, let's get the next point out of the way:


## But environment variables are evil, right?

Well... *it depends*.

Indeed, using environment variables for non-sensitive information that you could just as well transmit via command-line arguments or via a configuration file is not ideal. Why? Because being *environment* variables, they actually live *outside* of the code base. Sure, you can access them based on their key (their name) and attach some meaning to them, but this is neither the most Pythonic, nor the most effective way, to do things (if this can be avoided).

Nevertheless, there are also legit cases where environment variables are preferable:
* when setting execution mode (e.g. debug or development mode vs production mode)
* when they improve security practices
* when they are the only way to get some values into a "black box" (more on that later)

Before diving into the use cases, let us first briefly see how to access environment variables in Python.


## Accessing environment variables in Python

Environment variables are read through [`os.environ`][`os`]. Although they can also be modified or cleared, such changes are only effective in the current Python session (and for subprocesses started with `os.system()`, `popen()`, `fork()` and `execv()`). In other words, if you change an environment variable in a Python script, the change will not be reflected in the environment once that script exits.

### os.environ

In the most simple form, you can export an environment variable through the shell:

```bash
export foo='bar'
```

Then you can read its value through `os.environ`:

```python
In [1]: import os

In [2]: os.environ.get('foo')
Out[2]: 'bar'
```

Note that, for non-existent keys, `os.environ.get()` returns `None`.

Also note that the values of *all* environment variables are strings. To address this, you may want to roll your own small environment parser. Mine looks like this:

```python
import os


def parse_string(value):
    if value.lower() == 'true':
        return True
    if value.lower() == 'false':
        return False

    try:
        value = int(value)
        return value
    except ValueError:
        try:
            value = float(value)
        finally:
            return value


def get_env_setting(setting):
    if setting not in os.environ:
        return None

    return parse_string(os.environ[setting])
```

I use `get_env_setting()` to retrieve a value from `os.environ` (if the key exists) and I try to convert it to different data types:
* first, as a bool (this is because if I set boolean environment variables in Python, I store their `str()` representation, meaning `'True'` for `True` and `'False'` for `False`);
* if this fails, the value is converted to an `int`;
* if this fails as well, the value is converted to a `float`:
    * if successful, `parse_string()` returns a `float`;
    * if not, it returns a `str`.

### dotenv

To set multiple environment variables, you could create a bash script and ensure you run it before starting the Python script that needs these environment variables. But there is something more effective than this: [`dotenv`][] allows you to load environment variables from a `.env` file having the following format:

```bash
# Development settings
DOMAIN=example.org
ADMIN_EMAIL=admin@${DOMAIN}
ROOT_URL=${DOMAIN}/app
```

Notice the `.env` file understands UNIX expansion (e.g. `${DOMAIN}`).

[`dotenv`] loads the environment variables from `.env` into the environment:

```python
from dotenv import load_dotenv
load_dotenv()
```

Now the environment variables `DOMAIN`, `ADMIN_EMAIL` and `ROOT_URL` are accessible to the Python script and may be retrieved via `os.environ.get()` as shown above.


## Use case: setting execution mode

Here is a classic use case for environment variables. Suppose you don't want to add an explicit `-d` / `--debug` flag for your app. Then you could just export an environment variable to do the trick:

```bash
export MY_APP_DEBUG=1
```

The app would behave differently depending on the value of `MY_APP_DEBUG`.

Taking this idea one step further, you could use an environment variable `MY_APP_MODE` to choose between `development`, `staging` and `production` modes.


## Use case: securing access tokens

Many applications require access tokens: they can be API tokens, database passwords and so on. Storing such sensitive information inside the code base is just an accident waiting to happen, no matter how *sure* you are that you're *never* going to commit that special extra line to version control.

Here's where environment variables come in handy. You could add your secret tokens to the `.env` file and load it with `dotenv` as we've seen above. Of course, you'd need to make sure that your `.gitignore` or `.hgignore` contains the `.env` file.

In short, instead of:

```python
# NOTE TO SELF: DO ***NOT*** COMMIT THIS TO VERSION CONTROL!
SECRET_TOKEN = '56a682c4d000c676f543124b332a2921'
# ...
do_stuff_with(SECRET_TOKEN)
```

prefer adding your `SECRET_TOKEN` to `.env`, adding `.env` to your version control's ignore file, and finally:

```python
dotenv.load_dotenv()
# ...
SECRET_TOKEN = os.environ.get('SECRET_TOKEN')
do_stuff_with(SECRET_TOKEN)
```

## Use case: injecting configuration into a black box

This final use case is something you're not going to come across very often in internet discussions. It's something I call a "black box", meaning code that you have no control over: you didn't write it, you cannot change it but you have to run it. Along these lines, remember how I wrote in a [previous post][pp] about creating a Python script that runs user code from other Python scripts. That's the kind of use case I am referring to.

OK, you may ask, *but why???* Why would you want to run code that you have no control over? Well, suppose you're writing a testing framework that other people may use to write tests for... well, testing stuff. The tests are not relevant, only the part about _having to run them_ is. There are two aspects at play here:
* the framework is a **library** that users import from in order to write their tests;
* the framework is also a **framework**, meaning a master runner script that runs the user scripts.

For the users' sake, their only task should be to read and understand the framework's well-documented API. They should _not_ have to fiddle around with passing configuration options into their code. The configuration options for running their scripts through the framework may be sent through the command line and/or through configuration files.

What a user script typically does is to import abstractions from the framework and to use them for creating and executing tests. For example:

```python
# user_script.py
from fancy_framework import Test, PhaseResult

def a_test_phase(api):
    do_stuff()  # assume this exists

def another_test_phase(api):
    if not check_stuff():  # assume this exists
        return PhaseResult.FAIL

my_test = Test(a_test_phase, another_test_phase, name='My Test')
my_test.execute()
```

Let us suppose that if the user script is ran with verbosity off (default), it only shows the test result:

```
$ fancy_framework user_script.py

====================== Running test My Test (attempt #1) =======================

Finished running test My Test ......................................... [ FAIL ]
________________________________________________________________________________

$
```

When the script is ran with the `--verbose` flag, it displays phase results as well:

```
$ fancy_framework --verbose user_script.py

====================== Running test My Test (attempt #1) =======================

Phase a_test_phase .................................................... [ PASS ]
Phase another_test_phase .............................................. [ FAIL ]

Finished running test My Test ......................................... [ FAIL ]
________________________________________________________________________________

$
```

Now remember that what the `fancy_framework` does among other things is to simply run the provided `user_script.py`. How should a `fancy_framework.Test` object know whether verbosity is on when its `execute()` method is called? Here is where environment variables step in to save the day:
* The `fancy_framework` exports an environment variable `FANCY_FRAMEWORK_VERBOSITY` according to the user's choice (whether the `--verbose` flag was used).
* When a `Test` object is initialized, it reads the value of `FANCY_FRAMEWORK_VERBOSITY` from `os.environ` and stores it in an instance variable `self._verbose`.
* When the `execute()` method of the `Test` instance is called, details are printed to stdout only if `self._verbose` is true.

Here is a simplified version (using the `get_env_setting()` helper we've seen above):

```python
class Test:
    def __init__(self, *phases, name=None):
        self._phases = create_phases(phases)  # assume this exists
        self._name = name
        self._verbose = get_env_settings('FANCY_FRAMEWORK_VERBOSITY')

    def execute(self):
        print_running_test(self)  # assume this exists

        for phase in self._phases:
            phase.run()
            if self._verbose:
                print_phase_outcome(phase)  # assume this exists

        print_test_outcome(self)  # assume this exists
```

Isn't that neat? In this use case we've seen how environment variables can be used to inject configuration into a black-box system.

Check out [this article][pp-data] for a discussion of passing configuration options in Python in such a way as to only use *identifiers* instead of strings for the configuration keys.


<!-- links -->
[pp]: {% post_url 2021-11-04-standalone-python-script-to-run-other-python-scripts %}
[pp-data]: {% post_url 2021-11-05-python-configuration-and-dataclasses %}
[`argparse`]: https://docs.python.org/3/library/argparse.html
[`json`]: https://docs.python.org/3/library/json.html
[`pyyaml`]: https://wiki.python.org/moin/YAML
[`lxml`]: https://lxml.de/
[`minidom`]: https://docs.python.org/3/library/xml.dom.minidom.html
[`ElementTree`]: https://docs.python.org/3/library/xml.etree.elementtree.html
[`configparser`]: https://docs.python.org/3/library/configparser.html
[`os`]: https://docs.python.org/3/library/os.html
[`dotenv`]: https://pypi.org/project/python-dotenv/