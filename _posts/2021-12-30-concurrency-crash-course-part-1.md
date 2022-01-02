---
title: Concurrency crash course (Part 1)
date: 2021-12-30 00:00:00 +0100
categories: [Computer science, concurrency]
tags: [thread, process]
---

This is a multi-part post:
* Part 1 (this article) establishes terminology (tasks, threads and processes and how they relate to concurrency and parallelism) and gives an overview of challenges faced in concurrent programming.
* Part 2 (TODO) explains some common thread synchronization primitives, accompanied by Python examples.
* Part 3 (TODO) explains some common process synchronization primitives (inter-process communication mechanisms), accompanied by Python examples.
* Part 4 (TODO) tackles parallel algorithm design and performance evaluation.

I'm writing this for my frustrated past self, who couldn't wrap her head around these concepts. Moreover, my future self will likely benefit as well (I'm inferring this by extrapolating my current self's goldfish-grade memory). And last but not least, I'm also writing this post for anybody out there still struggling. :hugs:

## Tasks, processes and threads

First things first: let's establish some terminology.

Not everybody agrees on the definition of a **task**, but this term is so ubiquitously used that it is worth mentioning. A task refers to a set of instructions that are executed. For the purpose of this post, tasks can be seen as roughly equivalent to functions of a computer program.

A **thread** is the smallest set of instructions that can be managed by a scheduler. At the operating system (OS) level, a scheduler assigns resources (e.g. CPUs) to perform tasks.

A **process** is an instance of a running program. A process has at least one thread. However, programs can spawn multiple processes (e.g. a webserver may have a master process and several worker processes). To complicate things further, it is also possible to launch multiple instances of the same program (e.g. your favorite text editor).

I will attempt to give a more intuitive understanding of these terms in the following figure:

![Processes, threads and tasks](/assets/img/posts/process_thread_task.png){: width="700"}

In this example, we have a program that launches three processes. Processes 1, 2 and 3 have 2, 1, and 3 threads, respectively. Each thread runs a given task A through F.

There is an important distinction to consider: the threads of a given process all share that process's address space, but come with their own stacks and registers. In other words:
* A thread has its own stack and registers
* A thread has access to the code, data and resources of its owner process

Therefore, in terms of resource sharing:
* Threads (of a given process) share the same resources. Care must be taken as to how threads access those resources. Thread synchronization primitives such as condition variables, semaphores, mutexes or barriers allow to control the way in which threads access shared resources and will be discussed in a future post.
* Processes do not share the same resources, by default. Process synchronization, also known as [Inter-Process Communication (IPC)][ipc], must be used if resource sharing is necessary. Some of the most common mechanisms for achieving IPC are signals, sockets, message queues, pipes and shared memory. IPC will be discussed in a future post.

How are tasks executed with respect to one another? How are processes executed with respect to the CPUs? Here's where the next part comes in, where we discuss concurrency vs parallelism.

## Concurrency and parallelism

Here's what Rob Pike (best known as being one of the three creators of the Go programming language) has to say ([slides][pike-slides], [talk][pike-talk]):

>**Concurrency** is about *dealing* with lots of things at once. **Parallelism** is about *doing* lots of things at once. Not the same, but related.

He also goes on to say:

>**Concurrency** is about *structure*, **parallelism** is about *execution*. Concurrency provides a way to structure a solution to solve a problem that *may* (but not necessarily) be parallelizable.

(emphasis mine)

In operating system terms, the "things" Rob Pike refers to are tasks. Concurrency simply means that the operating system will schedule the tasks to run in an interleaved fashion, thus creating the illusion of them being executed at the same time:

![Concurrency](/assets/img/posts/concurrency.png){: width="700"}

Whereas parallelism means that tasks are run on actually distinct CPUs:

![Parallelism](/assets/img/posts/parallelism.png){: width="700"}

Make sure to also check out Jakob Jenkov's [post][jenkov] on concurrency vs parallelism to get a broader picture (and to see how he defines parallelism in a stricter sense than what I have conveyed here).

## What is concurrency used for?

Concurrency is useful for two types of problems: I/O-bound and CPU-bound.

![I/O- vs CPU-bound problems](/assets/img/posts/io_cpu_bound.png){: width="700"}

**I/O-bound problems** are affected by long input/output wait times. The resources involved may be files on a hard drive, peripheral devices, network requests, you name it. In the above diagram, red blocks show how much time is spent for I/O operations. When downloading files from the internet, for instance, an important speedup can be attained if we download concurrently instead of sequentially. The speedup comes from overlapping the I/O-bound wait times (the red blocks in the diagram). Therefore, **concurrency** (launching more **threads**) can improve **I/O-bound problems**.

For **CPU-bound problems**, on the other hand, the limiting factor is the CPU speed. These are generally computational problems. If such programs can be decomposed into independent tasks (with the typical example being matrix multiplication), then an important speedup can be attained if we throw more CPUs at the problem. Therefore, **parallelism** (launching more **processes**) can improve **CPU-bound problems**.

## Challenges in concurrent programming

Writing a concurrent program is more difficult than writing its sequential version. There are many things to consider and account for. Often times, isolation testing is a nightmare. Here we will discuss some of the most common challenges.

### Race condition

A **race condition** leads to inconsistent results that stem from the order in which threads or processes act on some shared state.

<a href='https://imgflip.com/i/5h1twc'><img src='https://i.imgflip.com/5h1twc.jpg'/></a><a href='https://imgflip.com/gif-maker'> via Imgflip GIF Maker</a>

For example, suppose the shared state is the string `"wolf"`. We have two threads, each prefixing the shared state with a different word: thread A prefixes the shared string with `"bad"` and thread B prefixes it with `"big"`.
* If A runs before B, the shared state becomes `wolf => bad wolf => big bad wolf`
* If B runs before A, the shared state becomes `wolf => big wolf => bad big wolf`

We can try to isolate race conditions using `sleep()` statements that will hopefully modify timing and execution order.

Race conditions occur because access to the shared state happens outside of synchronization mechanisms. A possible mitigation strategy is to use barriers (see the next post in the series on *Thread synchronization primitives*).

### Deadlock

A **deadlock** occurs when several tasks are blocked indefinitely while holding a shared resource and while waiting for another one.

![Deadlock](/assets/img/posts/deadlock.png){: width="700"}

Deadlocks occur when the *Coffman conditions* below are satisfied simultaneously:
1. *Mutual exclusion*: at least two shared resources are held without sharing them with other tasks.
2. *Hold and wait*: a task that holds a resource is requesting another resource which is held by another task.
3. *No preempt*: the task is responsible to release the resource voluntarily.
4. *Circular wait*: each task is waiting for a resource that is held by another task, for all tasks involved up to the last one which is, in turn, waiting for a resource held by the first task.

### Livelock

A **livelock** is similar to a deadlock: it involves tasks that need at least two resources each, however none of them is blocked. Unlike a deadlock, tasks in a livelock are overly polite: they acquire a resource, they test whether another resource is available, they release the first resource if the second one is not available, wait for a given amount of time, then repeat the whole process all over again. If bad timing is involved, none of the tasks involved in a livelock can ever progress. The irony is that livelocks often occur while attempting to correct for deadlocks...

### Starvation

Resource **starvation** occurs when a task never acquires a resource it needs. It can usually be resolved by improving the scheduling algorithm such that tasks that has been waiting for a long time get assigned a higher priority.

### Priority inversion

**Priority inversion** occurs when a task with low priority holds a resource required by a task with high priority. This results in the low-priority task finishing before the high-priority task. It can also get more subtle than this, involving a task with medium priority that preempts the low priority task, thus indirectly blocking the high-priority task indefinitely. [Several protocols][prio-inv] can be used to avoid priority inversion, one of them being priority inheritance. This is how the Mars Pathfinder priority inversion bug from 1997 was [fixed][mars].

## Conclusion

This post takes a bird's eye view of concurrency by:
* Establishing some necessary terminology (task, thread, process, concurrency, parallelism)
* Taking a look at two classes of problems (I/O-bound and CPU-bound) and how they relate to concurrency
* Explaining some common pitfalls in concurrent programming (race conditions, deadlock, livelocks, starvation and priority inversion)

The next posts in this series will illustrate synchronization primitives (for threads and processes), list principles to keep in mind when designing concurrent programs, and show how to evaluate parallel implementations.

## Resources

* [Concurrency vs parallelism][jenkov] (Jakob Jenkov)
* [Concurrency is not parallelism][pike-slides] (Rob Pike, 2012) [slides]
* [Concurrency is not parallelism][pike-talk] (Rob Pike, 2012) [talk]
<!--* [Speed up your Python program with concurrency][realpy] (Jim Anderson @ RealPython, 2019)-->
* [Inter-process communication][ipc] (Wikipedia)
<!--* [Python parallel and concurrent programming (Part 2)][linkedin] (LinkedIn Learning)-->
* [Priority inversion][prio-inv]

<!-- links -->
[jenkov]: http://tutorials.jenkov.com/java-concurrency/concurrency-vs-parallelism.html
[pike-slides]: https://go.dev/talks/2012/waza.slide
[pike-talk]: https://www.youtube.com/watch?v=oV9rvDllKEg
[realpy]: https://realpython.com/python-concurrency
[ipc]: https://en.wikipedia.org/wiki/Inter-process_communication
[linkedin]: https://www.linkedin.com/learning/python-parallel-and-concurrent-programming-part-2
[prio-inv]: http://www.embeddedlinux.org.cn/rtconforembsys/5107final/LiB0101.html
[mars]: https://medium.com/delta-force/the-case-of-mysterious-system-resets-on-mars-pathfinder-b01eab813b69