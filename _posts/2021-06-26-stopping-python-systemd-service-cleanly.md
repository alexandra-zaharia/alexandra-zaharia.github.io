---
title: Stopping a Python systemd service cleanly
date: 2021-06-26 00:00:00 +0100
categories: [Python, systemd]
tags: [python, systemd, service]
---

Suppose you are running a Python systemd service that opens some file descriptors (these can be regular files, named pipes, sockets, and so on). When the service is stopped, it needs to finish cleanly by closing and/or removing the associated resources. This post presents two manners in which the  service may be stopped gracefully.

First, we will look at how a Python script `my_service.py` may be ran as a systemd service. Next, we will discuss the general form of that Python script. Finally, we will see how to stop the service gracefully.

## Running a Python script as a systemd service 

An executable can be made into a systemd service by creating a _unit (configuration) file_, also known as a `.service` file (see the [systemd.service][] man page). The `.service` files are stored in a specific location and have a certain format.

### Unit file location

There are actually three places where [unit files][systemd.unit] may be stored:

* `/etc/systemd/system/`: system-specific; take precedence over run-time unit files
* `/run/systemd/system/`: run-time; take precedence over default unit files
* `/usr/lib/systemd/system/`: default; may be overwritten when the system updates

If you create your own systemd service, place the `.service` unit file in `/etc/systemd/system/`.

### Unit file format

A unit file (for a service) has three sections: `[Unit]`, `[Service]` and `[Install]`. You can read more on [unit file anatomy][unit_anatomy]. For this example, let us assume a very simple `my-service.service` file:

```
[Unit]
Description=My test service
After=network.target

[Service]
User=alex
Type=simple
ExecStart=/usr/bin/python /home/alex/services/my_service.py

[Install]
WantedBy=multi-user.target
```

### Using the service 

First, we need to add the new service and to reload systemd:

```bash
sudo ln -sf ~/services/my-service.service /etc/systemd/system/my-service.service
sudo systemctl daemon-reload
```

Then we can enable the service (this way it will be ran each time the system boots) and start it:

```bash
sudo systemctl enable my-service.service
sudo systemctl start my-service.service
```

If `my_service.py` is configured to log information (and it should!), we can check the system logs with `journalctl`:

```bash
journalctl -u my-service.service
```

Add an `-f` to the previous command if you want to follow the journal as it updates, just like with `tail -f`.

### The danger zone

Recall the original assumption: `my_service.py` opens file descriptors and uses the underlying resources. Whenever the service stops running (through `sudo systemctl stop my-service.service`, for example), it must do so gracefully by closing the open resources first.

Before diving into the possible solutions, let us first take a look at the general structure of the `my_service.py` script.

## The structure of the Python service

The Python service (`my_service.py`) may be a TCP server, client, or any other process that needs to run as a daemon. For the purpose of this example, let us suppose that the service uses a standard event loop and opens a named pipe in `/tmp` to read from it. Instead of doing something useful, our example service will just sleep and then log something when it wakes up. The general structure is as follows (possible exceptions from the `os` module are not caught for brevity purposes):

```python
import logging
import os
import sys
import time


class MyService:
    FIFO = '/tmp/myservice_pipe'

    def __init__(self, delay=5):
        self.logger = self._init_logger()
        self.delay = delay
        if not os.path.exists(MyService.FIFO):
            os.mkfifo(MyService.FIFO)
        self.fifo = os.open(MyService.FIFO, os.O_RDWR | os.O_NONBLOCK)
        self.logger.info('MyService instance created')
        
    def _init_logger(self):
        logger = logging.getLogger(__name__)
        logger.setLevel(logging.DEBUG)
        stdout_handler = logging.StreamHandler()
        stdout_handler.setLevel(logging.DEBUG)
        stdout_handler.setFormatter(logging.Formatter('%(levelname)8s | %(message)s'))
        logger.addHandler(stdout_handler)
        return logger

    def start(self):
        try:
            while True:
                time.sleep(self.delay)
                self.logger.info('Tick')
        except KeyboardInterrupt:
            self.logger.warning('Keyboard interrupt (SIGINT) received...')
            self.stop()

    def stop(self):
        self.logger.info('Cleaning up...')
        if os.path.exists(MyService.FIFO):
            os.close(self.fifo)
            os.remove(MyService.FIFO)
            self.logger.info('Named pipe removed')
        else:
            self.logger.error('Named pipe not found, nothing to clean up')
        sys.exit(0)


if __name__ == '__main__':
    service = MyService()
    service.start()
```

If `my_service.py` is ran in a terminal, we can stop it by hitting Ctrl+C (registered as SIGINT):

```
python test_service.py 
    INFO | MyService instance created
    INFO | Tick
    INFO | Tick
^C WARNING | Keyboard interrupt (SIGINT) received...
    INFO | Cleaning up...
    INFO | Named pipe removed
```

## How to stop the Python service gracefully

In the previous section we've seen that the named pipe is removed as expected if `my_service.py` is executed in a console. However, if we run it as a systemd service, stopping it via `systemctl` will result in an improper shutdown of our service: the named pipe is not removed :scream:

This happens because, by default, the kill signal sent by systemd when using `systemctl stop` is SIGTERM, not SIGINT, therefore we cannot catch it as a `KeyboardInterrupt`.

Here are two solutions for this problem.

### Use SIGINT instead of SIGTERM

The most immediate solution is to instruct systemd to send a SIGINT (registered as a keyboard interrupt) instead of the default SIGTERM. This is done by adding the following line to the `[Service]` section of the `my-service.service` unit file:

```
KillSignal=SIGINT
```

This way, whenever we stop or restart the service with `systemctl`, the signal sent to kill `my_service.py` is SIGINT and it is caught by the `except` block in lines 32-34.

### Use a SIGTERM handler

The other solution is to register a signal handler for SIGTERM. For this, we will need to `import signal` and instruct Python to do something useful when receiving a SIGTERM. We add the following method to the `MyService` class:

```python
    def _handle_sigterm(self, sig, frame):
        self.logger.warning('SIGTERM received...')
        self.stop()
```

Note the two arguments `sig` and `frame`: although not used explicitly, they must be present because this is the method that we will register for handling SIGTERM. If they are absent we get a `TypeError` when a SIGTERM is received. We add the following line to the `__init__()` method:

```python
        signal.signal(signal.SIGTERM, self._handle_sigterm)
```

If we check the system log with `journalctl -u my-service.service` we see the following:

```
juin 26 18:35:17 phantom systemd[1]: Started My test service.
juin 26 18:35:17 phantom python[558725]:     INFO | MyService instance created
juin 26 18:35:22 phantom python[558725]:     INFO | Tick
juin 26 18:35:27 phantom python[558725]:     INFO | Tick
juin 26 18:35:32 phantom python[558725]:  WARNING | SIGTERM received...
juin 26 18:35:32 phantom python[558725]:     INFO | Cleaning up...
juin 26 18:35:32 phantom python[558725]:     INFO | Named pipe removed
juin 26 18:35:32 phantom systemd[1]: Stopping My test service...
juin 26 18:35:32 phantom systemd[1]: test-service.service: Succeeded.
juin 26 18:35:32 phantom systemd[1]: Stopped My test service.
```

## Conclusion

That's it! You can now use either of the two methods to stop your Python systemd service gracefully: either tell systemd to kill it with a SIGINT or, better yet, install a custom SIGTERM handler.

There's a caveat with both approaches if the service launches more than one process. The [systemd.kill] man page explains that by default (when `KillMode=control-group`) all the processes launched by the unit file will receive SIGTERM (or SIGINT if we use `KillSignal=SIGINT` as explained here). However, if they fail to stop, they will be knocked-out with SIGKILL. In such situations, one should be extra-careful with proper shutdown and cleanup.


## Further reading

* [systemd.service man page][systemd.service]
* [systemd.unit man page][systemd.unit]
* [Anatomy of a systemd unit file][unit_anatomy]
* [systemd.kill man page][systemd.kill]

<!-- links -->

[systemd.service]: https://www.freedesktop.org/software/systemd/man/systemd.service.html
[systemd.unit]: https://www.freedesktop.org/software/systemd/man/systemd.unit.html
[unit_anatomy]: https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files
[systemd.kill]: https://www.freedesktop.org/software/systemd/man/systemd.kill.html