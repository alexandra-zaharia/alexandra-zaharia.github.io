---
title: Function timeout in Python using the signal module
date: 2016-02-05 00:00:00 +0100
categories: [Python, timeout]
tags: [python, signal]
---

## The problem

Sometimes you may want to impose a timeout on a Python function. Why would you want to do such a thing? Let's say you're computing something but you know there are some hopeless scenarios where the computation just takes too long, and you'd be OK to just skip them and go on with the rest of the workflow. 

For an illustration, the figure below shows several tasks. Those that take longer than the specified timeout should be aborted (orange) and the remaining tasks that take a reasonable amount of time should be executed normally (green).

![A timeout is a cutoff for the duration of a task](/assets/img/posts/function_timeout.png){: width="600"}

There are several ways in which setting a timeout on a function may be achieved such that the execution continues past the timed-out method. We will be examining two solutions here:
* in this post we will be using [`signal`] module;
* in the [next post][np] we will be using the [`multiprocessing`] module.

## Solution using the signal module

### What are signals?

[Signals][signal] are a form of [inter-process communication][ipc] that only applies to [POSIX][]-compliant operating systems. Note that Microsoft Windows is _not_ POSIX-compliant, so this solution cannot be used when running Python on Windows. 

Signals can be regarded as software interrupts sent from the kernel to a process in order to inform it that a special event took place. The process receiving the signal can choose to handle it in a specific way (if the program was written with this intention, that is). Otherwise, signals are handled in a default manner specified by the default signal handlers. For example, when you press Ctrl + C in your Linux terminal to stop a running program, you are in fact sending it the SIGINT signal. The default handler for SIGINT is to stop process execution.

Check out this [article][stackabuse] for more information on handling UNIX signals in Python.

### Using signals to set a timeout 

Suppose we have a method `do_stuff()` that can sometimes be very time-consuming. We'll be keeping this very simple:

```python
import time

def do_stuff(n):
    time.sleep(n)
```

Let's say we only want to run `do_stuff()` to completion if it finishes in less than 6 seconds. With the `signal` module, this can be achieved if we set a timer (an "alarm") for 6 seconds just before calling `do_stuff()`. If the timer runs out before the function completes, SIGALRM is sent to the process. We will therefore be using `signal.alarm(6)` to set a timer for 6 seconds before calling `do_stuff()`. Note that the argument to `signal.alarm()` must be an integer. Let's check what happens:

```python
import signal
import time


def do_stuff(n):
    time.sleep(n)
    print('slept for {}s'.format(n))


def main():
    signal.alarm(6)
    do_stuff(2)
    do_stuff(5)
    do_stuff(6)


if __name__ == '__main__':
    main()
```

Here is the output:

```
$ python timeout_signal.py 
slept for 2s
Alarm clock
$ echo $?
142
```

What happened? Well, the timer was set to 6 seconds and it finally ran out, one second before the second call to `do_stuff()` would have normally finished. The process exits with code 142 (SIGALRM). Let us change the `main()` function to reset the alarm after each call to `do_stuff()`:

```python
def main():
    signal.alarm(6)
    do_stuff(2)
    signal.alarm(0)

    signal.alarm(6)
    do_stuff(5)
    signal.alarm(0)

    signal.alarm(6)
    do_stuff(6)
    signal.alarm(0)
```

Note that `signal.alarm(delay)` arms a timer for `delay` seconds. This means that if `do_stuff()` takes exactly `delay` seconds to complete, SIGALRM gets transmitted nonetheless.

We now obtain:

```
slept for 2s
slept for 5s
Alarm clock
```

Next, we will define a handler for SIGALRM. A **handler** is a function that "handles a signal" in the specific way we instruct it to behave. User-defined handlers are used to override the default signal handlers. For example, suppose you want your program to ask the user to confirm her desire to quit the program when she presses Ctrl + C in the terminal. In this case you'd need a SIGINT handler that only exits upon confirmation. Note that signal handlers must respect a fixed prototype. To quote from the [Python documentation][`signal`]:

> The handler is called with two arguments: the signal number and the current stack frame (...).

Even if a a signal handler does not use these two arguments, they must be present in the handler's prototype (and no other arguments may be passed). Here is our simple handler; it just throws a `TimeoutError`:

```python
def handle_timeout(sig, frame):
    raise TimeoutError('took too long')
```

This handler only makes sense if it is registered for SIGALRM. Registering `handle_timeout()` for SIGALRM should be added to the `main()` function of the script above. Here is how to do it:

```python
signal.signal(signal.SIGALRM, handle_timeout)
```

By re-running our script, we can see that it now stops with a `TimeoutError`:

```
slept for 2s
slept for 5s
Traceback (most recent call last):
  File "timeout_signal.py", line 31, in <module>
    main()
  File "timeout_signal.py", line 26, in main
    do_stuff(7)
  File "timeout_signal.py", line 10, in do_stuff
    time.sleep(n)
  File "timeout_signal.py", line 6, in handle_timeout
    raise TimeoutError('took too long')
TimeoutError: took too long
```

This is great, we have an exception now -- and exceptions are something we can deal with in Python.

### Making execution continue past the timeout

Next, let's handle the `TimeoutError`. We will change our script such that it loops indefinitely and at each iteration through the loop it attempts to `do_stuff()` for a random number of seconds between 1 and 10. If `do_stuff()` is called with 6 seconds or more, then SIGALRM is sent and handled by raising a `TimeoutError`. We catch that `TimeoutError` and continue execution until hitting Ctrl + C. As an added bonus, we also include a handler for SIGINT (Ctrl + C).

```python
import sys
import random
import signal
import time


def handle_sigint(sig, frame):
    print('SIGINT received, terminating.')
    sys.exit()


def handle_timeout(sig, frame):
    raise TimeoutError('took too long')


def do_stuff(n):
    time.sleep(n)


def main():
    signal.signal(signal.SIGINT, handle_sigint)
    signal.signal(signal.SIGALRM, handle_timeout)

    max_duration = 5
    
    while True:
        try:
            duration = random.choice([x for x in range(1, 11)])
            print('duration = {}: '.format(duration), end='', flush=True)
            signal.alarm(max_duration + 1)
            do_stuff(duration)
            signal.alarm(0)
        except TimeoutError as exc:
            print('{}: {}'.format(exc.__class__.__name__, exc))
        else:
            print('slept for {}s'.format(duration))


if __name__ == '__main__':
    main()
```

So how does the execution continue past the first timeout? As we've seen above, we installed a handler for SIGALRM (lines 12-13 and 22) that raises a `TimeoutError`. Exception handling is performed in the `main()` function inside an infinite loop (lines 26-36). If `do_stuff()` succeeds, the script displays a message informing the user for how long the function ran (lines 35-36). If the `TimeoutError` is caught, it is simply displayed and the script continues.

Here is how the output might look like:

```
duration = 3: slept for 3s
duration = 1: slept for 1s
duration = 10: TimeoutError: took too long
duration = 7: TimeoutError: took too long
duration = 5: slept for 5s
duration = 9: TimeoutError: took too long
duration = 2: slept for 2s
duration = 5: slept for 5s
duration = 1: slept for 1s
duration = 6: TimeoutError: took too long
duration = 2: slept for 2s
duration = 10: ^CSIGINT received, terminating.
```

### Drawbacks

Well, _it works_ but there are two problems with this solution:

1. As mentioned in the introduction to signals, this mechanism is only present on UNIX-like systems. If the script needs to run in a classic Windows environment, the `signal` module is not suitable.
2. A SIGALRM can arrive at any time; however, its handler may only be ran between **atomic** instructions. By definition, atomic instructions cannot be interrupted. So if the timer runs out during such an operation, even though SIGALRM is sent, it won't be handled until that long computation you've been trying to abort finally completes. Typically, when using external libraries implemented in pure C for performing long computations, the handling of SIGALRM may be delayed.

## Conclusion

In this post we've seen a simple solution involving UNIX signals that may be used in some situations to set a timeout on a Python function. However, this solution is less than ideal for two reasons: the operating system must be POSIX-compliant and it can only work between atomic operations. In the [next post][np] we will examine a better solution using the `multiprocessing` module.

## Further reading

* [Signals][signal] (Wikipedia)
* [`signal`][] (Python documentation)
* [Handling UNIX signals in Python][stackabuse] (Frank Hofmann on stackabuse)

<!-- links -->

[`signal`]: https://docs.python.org/3/library/signal.html
[`multiprocessing`]: https://docs.python.org/3/library/multiprocessing.html
[np]: {% post_url 2016-02-08-function-timeout-in-python-multiprocessing %}
[signal]: https://en.wikipedia.org/wiki/Signal_(IPC)
[ipc]: https://en.wikipedia.org/wiki/Inter-process_communication
[POSIX]: https://en.wikipedia.org/wiki/POSIX
[stackabuse]: https://stackabuse.com/handling-unix-signals-in-python/
