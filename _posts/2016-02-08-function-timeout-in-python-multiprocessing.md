---
title: Function timeout in Python using the multiprocessing module
date: 2016-02-08 00:00:00 +0100
categories: [Python, timeout]
tags: [python, multiprocessing]
---

## The problem

Sometimes you may want to impose a timeout on a Python function. Why would you want to do such a thing? Let's say you're computing something but you know there are some hopeless scenarios where the computation just takes too long, and you'd be OK to just skip them and go on with the rest of the workflow.

For an illustration, the figure below shows several tasks. Those that take longer than the specified timeout should be aborted (orange) and the remaining tasks that take a reasonable amount of time should be executed normally (green).

![A timeout is a cutoff for the duration of a task](/assets/img/posts/function_timeout.png){: width="600"}

There are several ways in which setting a timeout on a function may be achieved such that the execution continues past the timed-out method. We will be examining two solutions here:
* in the [previous post][pp] we have used the [`signal`] module;
* in this post we will be using the [`multiprocessing`] module.

## Solution using the multiprocessing module

Just like in the [previous post][pp], suppose we have a method that can be very time-consuming:

```python
import time

def do_stuff(n):
    time.sleep(n)
    print('slept for {}s'.format(n))
```

For the purpose of this example, we want to let this function `do_stuff()` run until it either completes or hits the 5-second mark, whichever event comes first. Actually, in the [previous post][pp], we let it run just below 6 seconds, because the argument to `signal.alarm()` is necessarily an integer. If that argument was 5, `do_stuff()` would not have been allowed to run for 5 seconds. Apart from the shortcomings of the `signal`-based solution, the `multiprocessing` module also solves this nagging issue; we can now use a non-integer timeout, for example 5.01 seconds.

Although `multiprocessing` is the package that comes to mind when attempting to parallelize processes, its basic role is to simply spawn processes, as its name implies. (Processes spawned with `multiprocessing` *may*, but do not *have to*, be parallel.) We can set a timeout on the processes that are spawned, which is exactly what we are looking for here.

The script below runs indefinitely. At each passage through the infinite loop, it randomly selects a duration between 1 and 10 seconds. It then spawns a new `multiprocessing.Process` that executes the time-consuming `do_stuff()` function for the random duration. If `do_stuff()` doesn't finish in 5 seconds (actually, 5.01 seconds), the process terminates:

```python
import multiprocessing as mp
import random
import time


def do_stuff(n):
    time.sleep(n)
    print('slept for {}s'.format(n))


def main():
    max_duration = 5

    while True:
        duration = random.choice([x for x in range(1, 11)])
        print('duration = {}: '.format(duration), end='', flush=True)

        process = mp.Process(target=do_stuff, args=(duration,))
        process.start()
        process.join(timeout=max_duration + 0.01)

        if process.is_alive():
            process.terminate()
            process.join()
            print('took too long')


if __name__ == '__main__':
    main()
```

A `multiprocessing.Process` is spawned at line 18 with `do_stuff()` as its target and the random `duration` as argument for `do_stuff()`. Next, we start the process at line 19, and then we "join" it (meaning we wait for it to finish) at line 20, but with a twist: it is here that we actually specify the timeout. In other words, we wait for it to finish for the specified timeout. In lines 22-25 we check whether the process actually finished, in which case `is_alive()` returns false. If it is still running, we terminate the process and display a message on STDOUT.

Here is the output of this script:

```
duration = 7: took too long
duration = 9: took too long
duration = 2: slept for 2s
duration = 5: slept for 5s
duration = 2: slept for 2s
duration = 6: took too long
duration = 5: slept for 5s
duration = 3: slept for 3s
duration = 7: ^C
```

## Notes

1. Simply spawning processes with the `multiprocessing` module does not mean we have parallelism. In order to do this we'd need to add tasks to a `multiprocessing.Pool`. [This][mp1] article or [this][mp2] one show examples of pools.
2. Care must be taken when using `terminate()` to stop a process. Here is what the Python documentation has to say about it:
  > **Warning:** If this method is used when the associated process is using a pipe or queue then the pipe or queue is liable to become corrupted and may become unusable by other process [_sic_]. Similarly, if the process has acquired a lock or semaphore etc. then terminating it is liable to cause other processes to deadlock.

## Conclusion

In this post we've seen another solution for setting a timeout on a function in Python, this time using the `multiprocessing` module. It is easy to implement and does not suffer from any of the drawbacks of the `signal`-based solution described in the [previous post][pp].

## Further reading

* [`multiprocessing`] (Python documentation)
* [Parallel processing in Python][stackabuse] (Frank Hofmann on stackabuse)
* [`multiprocessing` -- Manage processes like threads][pymotw] (Doug Hellmann on Python Module of the Week)

<!-- links -->

[pp]: {% post_url 2016-02-05-function-timeout-in-python-signal %}
[mp1]: {% post_url 2019-04-17-run-python-script-as-subprocess-with-multiprocessing %}
[mp2]: {% post_url 2019-04-17-multiprocessing-in-python-with-shared-resources %}
[`signal`]: https://docs.python.org/3/library/signal.html
[`multiprocessing`]: https://docs.python.org/3/library/multiprocessing.html
[stackabuse]: https://stackabuse.com/parallel-processing-in-python/
[pymotw]: https://pymotw.com/3/multiprocessing/index.html
