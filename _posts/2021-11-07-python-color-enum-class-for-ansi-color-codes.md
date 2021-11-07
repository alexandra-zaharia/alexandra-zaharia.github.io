---
title: Python color enum class for ANSI color codes
date: 2021-11-07 00:01:00 +0100
categories: [Python, color]
tags: [python, ansi codes]
---

When you want to print something with color and/or styling in a terminal, you can use one of the existing modules, such as [`colorama`][] or [`sty`][]. They are well-documented, flexible and easy to use.

## Enum class

A very good guideline in software engineering is to avoid reinventing the wheel :upside_down_face:

But let's face it, for some projects you really don't want to bring in an external dependency. If the task is simple enough, you can just roll your own implementation. In this case, you'd just need to look into [ANSI escape codes][wiki ansi] and find some [examples][tutorial].

For my very limited use cases when printing with color in a terminal, I just need to define a few foreground colors as well as the bold and the blink styles. I'm using an `Enum` class such that color names are class members, thus avoiding any possible misspelling issues:

```python
from enum import Enum, auto


class Color(Enum):
    RED = 31
    GREEN = auto()
    YELLOW = auto()
    BLUE = auto()
    MAGENTA = auto()
    CYAN = auto()
    LIGHTRED = 91
    LIGHTGREEN = auto()
    LIGHTYELLOW = auto()
    LIGHTBLUE = auto()
    LIGHTMAGENTA = auto()
    LIGHTCYAN = auto()

    _START = '\u001b['
    _BOLD = ';1'
    _BLINK = ';5'
    _END = 'm'
    _RESET = '\u001b[0m'

    @staticmethod
    def colored(color, msg, bold=False, blink=False):
        if not(isinstance(color, Color)):
            raise TypeError(f'Unknown color {color}')

        fmt_msg = Color._START.value + str(color.value)

        if bold:
            fmt_msg += Color._BOLD.value
        if blink:
            fmt_msg += Color._BLINK.value

        return fmt_msg + Color._END.value + str(msg) + Color._RESET.value
```

We can use this class as follows:

```python
from color import Color

for item in Color:
    if item.name.startswith('_'):
        continue
    print(Color.colored(item, item.name))
    if item.name.startswith('LIGHT'):
        print(Color.colored(item, '{} bold == {} bold'.format(
            item.name[5:], item.name), bold=True))
```

Here's the output:

![Colored output using the Color enum](/assets/img/posts/color_enum.png){: width="300" .normal}

## Accompanying code

The full code accompanying this post can be found on my [GitHub][] repository.


<!-- links -->
[`colorama`]: https://github.com/tartley/colorama
[`sty`]: https://github.com/feluxe/sty
[tutorial]: https://www.lihaoyi.com/post/BuildyourownCommandLinewithANSIescapecodes.html
[wiki ansi]: https://en.wikipedia.org/wiki/ANSI_escape_code#SGR_(Select_Graphic_Rendition)_parameters
[GitHub]: https://github.com/alexandra-zaharia/python-playground/tree/main/color_enum