---
title: Make your own custom color formatter with Python logging
date: 2021-06-23 00:00:00 +0100
categories: [Python, logging, color]
tags: [python, logging, formatter, ansi codes]
---

## Logging

There comes a time in the life of a Python package when proper logs beat `print()`ing to standard output :smiley: The standard Python library offers the versatile [`logging`][] module, and if that does not fit your needs there's this elegant package called [`loguru`][]. In this article I will only be addressing the standard library `logging` module.

This [tutorial][`log_tut`] explains how to get up and running with `logging`. You basically have the choice between the basic logging configuration and creating your own custom logger, more easily adapted to meet your specific requirements.

## The scenario

Let us suppose you need to log both to standard output and to a file. Furthermore, you want your console output to be colored. If your intention is to **avoid bringing new dependencies to your project** (otherwise you'd use `loguru`, or at the very least `colorama`), you can do this with a bare-bones custom [`logging.Formatter`] class and with [ANSI escape codes][`color_codes`]. Sounds fun? Let's dive right in.

### The logger

Here is what your logger might look like:

```python
import logging
import datetime

# Create custom logger logging all five levels
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Define format for logs
fmt = '%(asctime)s | %(levelname)8s | %(message)s'

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
```

First, we create a custom logger (line 5) and set it to log everything (line 6, from debug through critical messages).
Every log record should display the time and date, the message level and the message (line 9). Notice that we align the message level on 8 characters (this way the five possible strings for `levelname`, namely DEBUG, INFO, WARNING, ERROR and CRITICAL are all right-aligned).

Next, we set up two handlers, one for console output (lines 12-14) and the other one for logging to a file (lines 17-20). Both handlers log all five levels of messages and both use the log format we've just discussed. The file handler creates a new log file every day and names it appropriately (lines 17-18). However, the console handler should have the ability to print every level in a distinct color, which is why we will be using the `CustomFormatter` class presented below.

Finally, in lines 23-24, we add the two handlers to our logger and we're ready to go.

### A custom color formatter

For building our own custom formatter, we will extend the `logging.Formatter` class, give it the log format we want, and instruct it to print out each message level in a distinct color. ANSI escape codes for 8-color, 16-color and 256-color terminals may be found [here][`color_codes`].

Here is how my custom color formatter looks like:

```python
class CustomFormatter(logging.Formatter):
    """Logging colored formatter, adapted from https://stackoverflow.com/a/56944256/3638629"""

    grey = '\x1b[38;21m'
    blue = '\x1b[38;5;39m'
    yellow = '\x1b[38;5;226m'
    red = '\x1b[38;5;196m'
    bold_red = '\x1b[31;1m'
    reset = '\x1b[0m'

    def __init__(self, fmt):
        super().__init__()
        self.fmt = fmt
        self.FORMATS = {
            logging.DEBUG: self.grey + self.fmt + self.reset,
            logging.INFO: self.blue + self.fmt + self.reset,
            logging.WARNING: self.yellow + self.fmt + self.reset,
            logging.ERROR: self.red + self.fmt + self.reset,
            logging.CRITICAL: self.bold_red + self.fmt + self.reset
        }

    def format(self, record):
        log_fmt = self.FORMATS.get(record.levelno)
        formatter = logging.Formatter(log_fmt)
        return formatter.format(record)
```

And here it is in practice:

```python
logger.debug('This is a debug-level message')
logger.info('This is an info-level message')
logger.warning('This is a warning-level message')
logger.error('This is an error-level message')
logger.critical('This is a critical-level message')
```

![Colored logging output](/assets/img/posts/colored_logging.png){: width="600"}

The ANSI escape codes can obviously be substituted with [`colorama`][], if extra dependencies are not an issue.

## Further reading

* [`logging`][] (Python documentation)
* [Python `logging` tutorial at Real Python][`log_tut`]
* [ANSI escape codes for colored terminal output][`color_codes`]


<!-- links -->

[`logging`]: https://docs.python.org/3/library/logging.html
[`loguru`]: https://github.com/Delgan/loguru
[`log_tut`]: https://realpython.com/python-logging/
[`logging.Formatter`]: https://docs.python.org/3/library/logging.html#logging.Formatter
[`color_codes`]: https://www.lihaoyi.com/post/BuildyourownCommandLinewithANSIescapecodes.html
[`colorama`]: https://pypi.org/project/colorama/
