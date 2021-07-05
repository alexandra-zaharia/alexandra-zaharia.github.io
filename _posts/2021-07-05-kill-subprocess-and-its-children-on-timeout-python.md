---
title: Kill a Python subprocess and its children when a timeout is reached
date: 2021-07-05 00:00:00 +0100
categories: [Python, subprocess, timeout]
tags: [python, process, subprocess, signal]
---

Suppose a Python script needs to launch an external command. This can be done using the [`subprocess`][] module in one of two ways:

* either use the "convenience" function `subprocess.run()`
* or use the more flexible `Popen` interface

## Stopping a subprocess on timeout

The "convenience" function `subprocess.run()` allows to do quite a number of useful things, such as capturing output, checking the external command's return code or setting a timeout, among others. 

If we are simply interested in stopping the execution of the external command after a given timeout has been reached, it is sufficient to `subprocess.run()` the command and catch the `TimeoutExpired` exception if it is raised:

```python
import subprocess

cmd = ['/path/to/cmd', 'arg1', 'arg2']  # the external command to run
timeout_s = 10  # how many seconds to wait 

try:
    p = subprocess.run(cmd, timeout=timeout_s)
except subprocess.TimeoutExpired:
    print(f'Timeout for {cmd} ({timeout_s}s) expired')
```

## Stopping a subprocess and its children on timeout

The situation gets more complicated when the external command may launch one or several child processes. In order to be able to stop the child processes as well as the parent, it is necessary to use the `Popen` constructor. 

> **Note:** The following only applies to UNIX-like operating systems. (Read: it won't work on Windows.)

The reason for using the `Popen` constructor for this scenario is that it can be instructed to launch a new *session* for the external command. Then, the whole *process group* belonging to the external command can be terminated on timeout. A **process group** is simply a group of processes that can be controlled at once (via [signals][Signals]), while a **session** is a collection of process groups. Here are the official [definitions][posix defs],  taken from the [POSIX.1-2008 standard][posix]:

> **3.296 Process Group** - A collection of processes that permits the signaling of related processes. Each process in the system is a member of a process group that is identified by a process group ID. A newly created process joins the process group of its creator.

> **3.343 Session** - A collection of process groups established for job control purposes. Each process group is a member of a session. A process is considered to be a member of the session of which its process group is a member. A newly created process joins the session of its creator. A process can alter its session membership; see `setsid()`. There can be multiple process groups in the same session.

### The reason for using a session instead of a process group

Reading the above definitions, one may wonder why should we bother with creating a new session instead of simply using a new process group for the external command. That's an excellent question! It is technically possible, but not advisable. In order to create a process group, we'd need to call `os.setpgrp()` (which uses the [`setpgrp()`][] system call). However, there are two problems with this approach:

* `setpgrp()` is marked as obsolete and may be removed in future versions (check the `man` page);
* the only way to call `os.setpgrp()` from within the `Popen` constructor is to pass it to the `preexec_fn` parameter, which is *not* thread-safe. 
    
The Python documentation for [`Popen()`][popen] states the following:

> **Warning:** The `preexec_fn` parameter is not safe to use in the presence of threads in your application. The child process could deadlock before `exec` is called. If you must use it, keep it trivial! Minimize the number of libraries you call into.

In the note following the warning, it is mentioned that:

> The `start_new_session` parameter can take the place of a previously common use of `preexec_fn` to call `os.setsid()` in the child.

The workaround, therefore, is to simply create a new session by setting the `start_new_session` argument of the `Popen` constructor to `True`. According to the Python documentation, it is the equivalent of using `preexec_fn=os.setsid` (based on the [`setsid()`][] system call), but without the un-thread-safe warning.

### Implementation

With all the above explanations, the implementation is straight-forward:

```python
import os
import signal
import subprocess
import sys

cmd = ['/path/to/cmd', 'arg1', 'arg2']  # the external command to run
timeout_s = 10  # how many seconds to wait 

try:
    p = subprocess.Popen(cmd, start_new_session=True)
    p.wait(timeout=timeout_s)
except subprocess.TimeoutExpired:
    print(f'Timeout for {cmd} ({timeout_s}s) expired', file=sys.stderr)
    print('Terminating the whole process group...', file=sys.stderr)
    os.killpg(os.getpgid(p.pid), signal.SIGTERM)
```

The `Popen` interface is different than that of the convenience `subprocess.run()` function. The timeout needs to be specified in `Popen.wait()`. If you want to capture `stdout` and `stderr`, you need to pass them to the `Popen` constructor as `subprocess.PIPE` and then use `Popen.communicate()`. Regardless of the differences, whatever can be done with `subprocess.run()` can also be achieved with the `Popen` constructor.

When the timeout set in `Popen.wait()` has elapsed, a `TimeoutExpired` exception is raised. Then, in line 15, we send a SIGTERM to the whole process group (`os.killpg()`) of the external command (`os.getpgid(p.pid)`).

That's it. Happy infanticide! (Err... I was referring to child processes :grin:)

## Further reading

* [`subprocess`][] (Python documentation)
* [`signal`][] (Python documentation)
* [Signals][] (Wikipedia)
* [POSIX.1-2008 standard][posix]
* [POSIX.1-2008 definitions][posix defs]

<!-- links -->

[`subprocess`]: https://docs.python.org/3/library/subprocess.html
[`signal`]: https://docs.python.org/3/library/signal.html
[Signals]: https://en.wikipedia.org/wiki/Signal_(IPC)
[posix]: https://pubs.opengroup.org/onlinepubs/9699919799/
[posix defs]: https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html
[`setpgrp()`]: https://pubs.opengroup.org/onlinepubs/9699919799/functions/setpgrp.html
[popen]: https://docs.python.org/3/library/subprocess.html#subprocess.Popen
[`setsid()`]: https://pubs.opengroup.org/onlinepubs/009604599/functions/setsid.html