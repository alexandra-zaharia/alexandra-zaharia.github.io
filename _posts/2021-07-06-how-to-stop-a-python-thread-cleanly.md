---
title: How to stop a Python thread cleanly
date: 2021-07-06 00:00:00 +0100
categories: [Python, parallelism, thread]
tags: [python, threads]
---

Suppose a Python thread needs to be stopped cleanly (it might need to perform cleanup). 

For illustration, we will take a very simple program in with a single "worker" thread that displays a message when it is done. The message is a placeholder for real cleanup, and the thread itself sleeps for a given number of iterations (as a placeholder for significant work). In our example, we want to stop the thread through a keyboard interrupt (Ctrl + C).


## By default, the thread is not stopped cleanly

In this first version, the program can be stopped by hitting Ctrl + C, but the thread keeps running. Here is the program:

```python
import threading
import time


def do_some_work(n_iter):
    for i in range(n_iter):
        print(f'iteration {i + 1}/{n_iter}')
        time.sleep(0.5)
    print('Thread done')


if __name__ == '__main__':
    n_iter = 10
    thread = threading.Thread(target=do_some_work, args=(n_iter,))
    thread.start()
    thread.join()
    print('Program done')
```

Here is what happens when stopping it via a keyboard interrupt (`...` denotes output that was snipped for readability purposes):

```
iteration 1/10
iteration 2/10
^CTraceback (most recent call last):
  ...
KeyboardInterrupt
iteration 3/10
iteration 4/10
^CException ignored in: <module 'threading' from '/home/alex/miniconda3/lib/python3.7/threading.py'>
  ...
KeyboardInterrupt
```

The first Ctrl + C stops the main program, but not the thread. The second time, the thread is stopped as well.

## Using a daemon thread is not a good idea

The Python [`threading`][] documentation explains that a thread may be started as a daemon, meaning that "the entire Python program exits when only daemon threads are left". The main program itself is not a daemon thread. 

While this approach has the merit of effectively stopping the thread, it does not allow to exit it cleanly. From the Python documentation:

> **Note:** Daemon threads are abruptly stopped at shutdown. Their resources (such as open files, database transactions, etc.) may not be released properly. If you want your threads to stop gracefully, make them non-daemonic and use a suitable signalling mechanism such as an `Event`.

## A clean thread exit using events and signals

Following on the previous note, a threading [`Event`][] is a simple object that can be set or cleared. It can be used to signal to the thread that it needs perform its cleanup and then stop.

The idea is to use such an event here (let us call it a _stop event_). Initially not set, the stop event becomes set when a keyboard interrupt is received. The worker thread then breaks out from the loop if the stop event is set and performs its cleanup.

Creating the stop event is straightforward (it can take any name):

```python
stop_event = threading.Event()
```

The worker thread checks whether the stop event is set:

```python
if stop_event.is_set():
    break
```

The stop event needs to be set when a keyboard interrupt is intercepted. This is done by registering the SIGINT [signal][Signals] with a handler function. The registration is done in the main program:

```python
signal.signal(signal.SIGINT, handle_kb_interrupt)    
```

The handler function `handle_kb_interrupt` must have two arguments, the signal and the frame, even though the second argument is not used:

```python
def handle_kb_interrupt(sig, frame):
    stop_event.set()
```

Here is the full program:

```python
import signal
import threading
import time


def do_some_work(n_iter):
    for i in range(n_iter):
        if stop_event.is_set():
            break
        print(f'iteration {i + 1}/{n_iter}')
        time.sleep(0.5)
    print('Thread done')


def handle_kb_interrupt(sig, frame):
    stop_event.set()


if __name__ == '__main__':
    stop_event = threading.Event()
    signal.signal(signal.SIGINT, handle_kb_interrupt)    
    n_iter = 10
    thread = threading.Thread(target=do_some_work, args=(n_iter,))
    thread.start()
    thread.join()
    print('Program done')

```

Here is the output when hitting Ctrl + C in the new version of the program:

```
iteration 1/10
iteration 2/10
iteration 3/10
^CThread done
Program done
```

Notice that when the thread is stopped it now finally gets to the `print('Thread done')` line (a placeholder for an actual cleanup task). Moreover, the main program also gets to the `print('Program done')` line. Clean exit from a thread can therefore be achieved using a threading event and a signal handler.

## Further reading

* [`threading`][] (Python documentation)
* [`signal`][] (Python documentation)
* [Signals][] (Wikipedia)

<!-- links -->

[`threading`]: https://docs.python.org/3/library/threading.html
[`Event`]: https://docs.python.org/3/library/threading.html#threading.Event
[`signal`]: https://docs.python.org/3/library/signal.html
[Signals]: https://en.wikipedia.org/wiki/Signal_(IPC)
