---
title: 操作系统 OS_4：线程
published: 2026-10-08
pinned: false
description: 线程与进程的区别、并发与并行、用户级与内核级线程、多对一/一对一/多对多/两级模型与班级比赛模型，以及 fork/exec 语义、线程取消、信号处理、线程池、TLS、调度器激活和 Pthreads、Windows、Linux、Java 实现。
image: ""
tags:
  - SE
  - 软件工程
  - 操作系统
category: 软件工程
draft: false
author: 山吹
comment: true
date: 2026-10-08
---

## 1. Why Threads: The Responsiveness Problem

The lecture opens with a Web browser instead of a definition. A single-threaded browser has one main loop, so it does everything in sequence:

```c
Main() {
    while (1) {
        RetrieveData();    // Block for 1 second
        DisplayData();     // Block for 1 second
        GetInputEvents();  // Block for 1 second
    }
}
```

Every call blocks, so the loop can spend up to three seconds before it looks at the keyboard. The first fix is to cut each operation into smaller pieces:

```c
Main() {
    while (1) {
        RetrieveALittleData();  // Block for 0.1 second
        DisplayALittleData();   // Block for 0.1 second
        GetAFewInputEvents();   // Block for 0.1 second
    }
}
```

Smaller pieces help, but they are still not responsive enough, and the next attempt makes the code worse by polling:

```c
Main() {
    while (1) {
        if (CheckData() == True) {
            RetrieveALittleData();  // 0.1 second
            DisplayALittleData();   // 0.1 second
        }
        if (CheckInputEvents() == True) {
            GetAFewInputEvents();   // 0.1 second
        }
    }
}
```

The slide states the problem directly: **a lot of checks, not efficient, and still not responsive**. The underlying tension is:

- To be responsive enough, the program must break operations into very, very small pieces.
- To be efficient enough, it wants to execute code in large pieces.
- These two requirements **conflict with each other**.

The resolution is not to find a better constant for the chunk size. It is to *schedule* the operations explicitly in the program, and to leave the tedious work to the OS, which schedules them as **threads**:

```c
Main() {
    CreateThread(RetrieveData());
    CreateThread(DisplayData());
    CreateThread(GetInputEvents());
    WaitForThreads();
}

void RetrieveData() {
    while (1) {
        retrieveData();
        ...
    }
}

void DisplayData() {
    while (1) {
        displayData();
        ...
    }
}
```

Each thread routine now holds its own loop and runs at its own pace. The browser no longer has to interleave three activities by hand, because the scheduler interleaves them on the CPU.

> [!IMPORTANT]
> The lesson of the example is not that polling is always wrong. It is that a program containing several independent activities needs a **unit of scheduling** smaller than the process. Once that unit exists, the OS can decide who runs, when, and on which core.

## 2. What Is a Thread?

A **thread** is the basic unit of CPU utilization. It consists of:

- a **thread ID**;
- a **program counter**;
- a **register set**;
- a **stack**.

It shares with the other threads of the same process the **code section**, the **data section**, and other operating-system resources such as open files and signals. A traditional, heavyweight process has a single thread of control; a multithreaded process has one program counter per thread, so the threads can be executing different code at the same time.

![single-and-multithreaded-processes.png](https://tu.shanchui.cc/file/blog/wenzhang/1791430795834_single-and-multithreaded-processes.png)

The figure shows the essential asymmetry. The single-threaded process has one register set and one stack. The multithreaded process keeps one copy of `code`, `data`, and `files`, but each thread has its own `registers` and `stack`.

| Aspect | Process | Thread |
| --- | --- | --- |
| Creation cost | Heavy: new address space, new PCB, new page tables | Light: a stack, a register set, and a thread ID in the same address space |
| Address space | Private and isolated | Shared with all peer threads |
| Context switch | Must change the address space (page tables, TLB, caches) | Stays in the same address space, so it is cheaper |
| Communication | Requires IPC: shared memory, messages, sockets | Directly through shared data, which still needs synchronization |
| Fault isolation | A bad pointer in one process normally does not damage another | A bad pointer in one thread can kill the whole process |
| Scheduling unit | The kernel schedules the process | The kernel schedules the thread when kernel threads are used |

> [!NOTE]
> Sharing is exactly what makes threads cheap, and exactly what makes them dangerous. Because the data section is shared, two threads that update the same variable need mutual exclusion, just like the bounded-buffer producer and consumer in OS_3. Nothing about threads removes the need for synchronization.

## 3. Benefits

The lecture lists four reasons to use threads:

| Benefit | Explanation |
| --- | --- |
| Responsiveness | In an interactive application, one thread can keep serving the user while another blocks on slow work |
| Resource sharing | Threads share memory for code and data by default, so no extra mechanism is needed to share a large structure |
| Economy | Creating and switching processes is more expensive than creating and switching threads |
| Utilization of multiprocessor architectures | Multithreading increases concurrency, so the work can actually be spread over several cores |

The economy argument is worth stating precisely. Sharing memory avoids duplicating an address space, and because a thread switch does not change the address space, the kernel does not have to reload page-table base registers or flush translation caches.

The last benefit is the one that made threads mainstream. On a single core, threads only improve structure and responsiveness. On a multicore machine they are the ordinary way to turn idle cores into throughput, which is why "one thread per core" is a common starting point for server design.

> [!NOTE]
> More threads do not automatically mean more throughput. If a large part of the work is serial, or if all threads contend for the same lock, adding threads only adds switching and synchronization overhead. The useful question is always which part of the work can actually proceed in parallel.

## 4. Concurrency vs. Parallelism

These two words are often used interchangeably, and the lecture separates them with two figures.

**Concurrency** means that more than one thread is making progress during the same interval of time. On a single-core system the threads take turns, so execution is interleaved rather than simultaneous.

![concurrency-vs-parallelism-1.png](https://tu.shanchui.cc/file/blog/wenzhang/1791878924130_concurrency-vs-parallelism-1.png)


**Parallelism** means that more than one thread is executing at the same instant, which requires more than one core.

![concurrency-vs-parallelism-2.png](https://tu.shanchui.cc/file/blog/wenzhang/1791878962173_concurrency-vs-parallelism-2.png)

| | Concurrency | Parallelism |
| --- | --- | --- |
| Definition | Several activities progress within the same time span | Several activities execute at the same instant |
| Hardware needed | One core is enough | More than one core |
| Mechanism | Interleaving by the scheduler | Simultaneous execution on separate cores |
| Property of | The structure of the program | The execution platform |

The textbook adds a second pair of terms for the way parallel work is split:

- **Data parallelism** distributes the same operation over subsets of the data, for example adding two arrays element by element on two cores.
- **Task parallelism** distributes distinct operations over the threads, for example one thread computing statistics while another renders the result.

A concurrent program may never run in parallel, and that is not a defect: the structure is what makes responsiveness and resource sharing possible. The reverse case is the interesting one: a program cannot become parallel just because the machine has more cores. Somebody, usually the programmer, has to create the threads that the extra cores can run.

## 5. User Threads and Kernel Threads

Threads are supported at two levels, and the distinction explains most of the models in the next section.

**User threads** are managed by a user-level thread library, without kernel knowledge. The library handles creation, scheduling, and switching inside the process, and the kernel still sees one process with one execution state.

**Kernel threads** are supported and scheduled directly by the kernel. The kernel can schedule each thread of a process separately, on the same core or on different cores.

| | User threads | Kernel threads |
| --- | --- | --- |
| Managed by | A thread library in user space | The kernel |
| Visible to the kernel | No, the kernel sees one process | Yes, each thread is a schedulable entity |
| Switching cost | Very low: a library call, no mode switch | Higher: involves the kernel |
| Blocking system call | Blocks the whole process | Blocks only the calling thread |
| True parallelism | Not with the classic many-to-one design | Yes, threads can run on several cores |

The lecture lists typical examples on both sides:

| Level | Examples from the lecture |
| --- | --- |
| User-level libraries | POSIX Pthreads, Win32 threads, Java threads |
| Kernel threads | Windows XP/2000, Solaris, Linux, Tru64 UNIX, Mac OS X |

> [!NOTE]
> Do not confuse an **API** with an **implementation level**. Pthreads, Win32 threads, and Java threads are programming interfaces. The lecture asks this directly on the Pthreads slide: *is it a user- or kernel-level library?* The answer is that it can be either. Linux, for example, implements the Pthreads API with kernel threads, so a "Pthreads thread" is a kernel thread there, and the true many-to-one model survives mainly in historical or special-purpose libraries.

## 6. Multithreading Models

The relationship between user threads and kernel threads has to be defined by some mapping. The lecture presents four:

- many-to-one;
- one-to-one;
- many-to-many;
- two-level.

### Many-to-One Model

Many user-level threads are mapped to a single kernel thread. Thread management is done entirely in user space, so it is efficient.

![many-to-one-model.png](https://tu.shanchui.cc/file/blog/wenzhang/1791878975899_many-to-one-model.png)

Examples from the lecture: **Solaris Green Threads** and **GNU Portable Threads**.

The weakness follows from the mapping. Because the kernel knows only one schedulable entity, a single blocking system call blocks the entire process, and no two threads of the process can run on two cores. The comparison slide states it plainly: management is efficient, but the process blocks on a system call, and the kernel can schedule only one thread at a time.

### One-to-One Model

Each user-level thread maps to one kernel thread, so the kernel sees every thread.

![one-to-one-model.png](https://tu.shanchui.cc/file/blog/wenzhang/1791878991620_one-to-one-model.png)

Examples from the lecture: **Windows NT/XP/2000**, **Linux**, and **Solaris 9 and later**.

This model gives more concurrency, because a blocking call blocks only its own thread and several threads can run on several cores. The price is that creating a user thread requires creating a kernel thread, which is expensive, and the number of threads in a system has to be limited to avoid exhausting kernel resources.

### Many-to-Many Model

Many user-level threads are multiplexed onto a smaller or equal number of kernel threads.

![many-to-many-model.png](https://tu.shanchui.cc/file/blog/wenzhang/1791878996937_many-to-many-model.png)

Examples from the lecture: **Solaris prior to version 9**, and **Windows NT/2000 with the ThreadFiber package**.

The kernel creates a sufficient number of kernel threads, and the thread library multiplexes the user threads onto them. The model is the most flexible of the three: the developer can create as many user threads as the problem needs, and the blocking problem of many-to-one disappears because a blocked user thread does not have to occupy the kernel thread that another user thread needs. The cost is complexity, because the library and the kernel must cooperate to keep the number of kernel threads appropriate for the workload.

### Two-Level Model

The two-level model is many-to-many plus binding: a user thread may be bound to a single kernel thread.

![two-level-model.png](https://tu.shanchui.cc/file/blog/wenzhang/1791879001254_two-level-model.png)

Examples from the lecture: **IRIX**, **HP-UX**, **Tru64 UNIX**, and **Solaris 8 and earlier**.

Binding is useful for a thread that needs a dedicated kernel thread, for example a thread that must be able to run while its peers block. The unbound threads still share the remaining kernel threads.

### Comparison

![multithreading-models-comparison.png](https://tu.shanchui.cc/file/blog/wenzhang/1791879007929_multithreading-models-comparison.png)

| Model | Concurrency | Cost | Main risk |
| --- | --- | --- | --- |
| Many-to-One | Low: one kernel thread per process | Cheapest thread operations | One blocking system call blocks the process; no multicore use |
| One-to-One | High: one kernel thread per user thread | Expensive thread creation | Too many threads exhaust kernel resources |
| Many-to-Many | High, with the number of kernel threads under control | Complex coordination | Requires kernel-library communication to stay correct |
| Two-Level | High, with selected threads bound | Complex, plus binding policy | Same as many-to-many, plus deciding what to bind |

> [!NOTE]
> These examples come from the era of the lecture's textbook. Most contemporary general-purpose systems converged on one-to-one, because kernel threads became cheap enough and because programming a many-to-many library correctly is hard. The models are still worth understanding: they describe a real design space, and the coordination problem they raise is the reason the scheduler-activation mechanism exists.

## 7. The Class-Race Model: Threads in the Sports Meet

OS_3 used a sports meet to explain process states: an athlete is a process, a bib number is a process ID, and a track is a CPU core. This chapter extends the same picture by adding classes, which turns it into an analogy for threads.

| Sports-meet concept | Operating-system concept |
| --- | --- |
| Class (班级) | Process |
| Class number (班级号) | Process ID |
| Class member (班级成员) | Thread |
| Bib worn on the track (号码簿) | Thread ID, or LWP ID when the kernel schedules it |
| Track (跑道) | CPU core |
| One athlete per track | One thread per core at a time |
| A stadium has one or more tracks | A machine has one or more cores |

The class as a whole is the process, one member of the class is one thread, and the number of bibs issued to a class is the number of kernel threads available to that process. The three competition rules in the slides are exactly the three multithreading models:

| Rule from the slides | Model | Meaning |
| --- | --- | --- |
| The class gets **one** bib; members take turns wearing it, and the kernel schedules whoever wears it | Many-to-one | Many user threads, one kernel thread |
| **Every** member may apply for a bib; several members of one class can be on the track at the same time | One-to-one | One kernel thread per user thread |
| The class is issued **a limited number of** bibs, for example at most two on the track; the class decides internally who wears them | Many-to-many | Many user threads multiplexed onto fewer kernel threads |

The bib is the key to the analogy, because it marks exactly what the kernel can schedule. In the many-to-one case the class has to choose one runner, so the other members are invisible to the kernel and cannot make progress in parallel. In the one-to-one case everybody can be on the track, which is why the model is the most concurrent and the most expensive. In the many-to-many case the bib is labelled by class and number, such as `一班01` or `一班02`, which is the same idea as an LWP identifier: the kernel schedules the labelled runner, while the class decides which member wears the label at any moment.

> [!IMPORTANT]
> The analogy makes the division of labour visible. The kernel only ever schedules entities that hold a bib, that is, kernel threads or LWPs. Everything above that line, including which user thread runs next, is decided by the thread library inside the process.

## 8. Threading Issues

Once a process contains several threads, a set of questions appear that do not exist for a single-threaded process. The lecture lists six.

### Semantics of `fork()` and `exec()`

The `fork()` system call creates a new process. If the calling process is multithreaded, does the new process duplicate **only the calling thread** or **all threads**?

- Duplicating all threads gives the child the same parallelism as the parent, but the other threads may be in states that make no sense to copy, for example blocked inside a system call or holding a lock whose owner is not duplicated.
- Duplicating only the calling thread keeps the child simple, but the child loses the parallelism of the parent.

Some UNIX systems therefore provide two versions of `fork()`, one that duplicates all threads and one that duplicates only the invoking thread. The usual pattern is that a program calls `fork()` and then immediately calls `exec()`: since `exec()` replaces the entire process image, duplicating the other threads in that case is wasted work, which is why the thread-only variant exists.

`exec()` itself is unambiguous: it replaces the whole process, so all threads of the calling process disappear and the new program starts with a single thread.

### Thread Cancellation

**Thread cancellation** terminates a thread before it has finished; the thread being cancelled is called the target thread. The lecture names two general approaches:

| Approach | Behavior | Trade-off |
| --- | --- | --- |
| Asynchronous cancellation | Terminates the target thread immediately | Fast, but the cancelled thread may hold a lock, an open file, or a partially updated structure, and it never gets a chance to clean up |
| Deferred cancellation | The target thread periodically checks a flag and decides whether it should be cancelled | Safe: the thread can release resources first, and cancellation happens at defined cancellation points |

Deferred cancellation is the default in the Pthreads model. A thread that must not be interrupted while it is updating shared data can temporarily disable cancellation, and a thread that is cancelled can register a cleanup handler that runs before it exits. The cost is latency: a thread that never reaches a cancellation point cannot be cancelled.

### Signal Handling

**Signals** are the UNIX mechanism for notifying a process that a particular event has occurred. A signal is **generated** by a specific event, **delivered** to a process, and then **handled**, either by a default action or by a signal handler registered by the program.

In a multithreaded process the question is where the signal should be delivered. The lecture lists four options:

| Option | When it makes sense |
| --- | --- |
| Deliver the signal to the thread to which the signal applies | Synchronous signals such as a divide-by-zero or an invalid memory access, which are caused by one specific thread |
| Deliver the signal to every thread in the process | The event concerns every thread, so all handlers should run |
| Deliver the signal to certain threads in the process | A policy is needed to select the threads |
| Assign a specific thread to receive all signals for the process | A dedicated signal-handling thread, which keeps the other threads free of this concern |

The method of delivery depends on the type of the signal: a synchronous fault has an obvious target thread, while an asynchronous signal such as a timer or a user pressing Ctrl+C has none.

### Thread Pools

Instead of creating a thread per request and destroying it afterwards, a server can create a **pool** of threads at start-up and let them wait for work. When a request arrives, it is handed to a thread that is already waiting; when the thread finishes, it returns to the pool.

The two advantages from the lecture:

- it is usually slightly faster to service a request with an existing thread than to create a new thread;
- the number of threads in the application is bounded by the size of the pool, so the server cannot be flooded into creating unlimited threads.

A pool also has a trade-off that is easy to miss: because the same thread handles different requests over time, it cannot rely on data left in its own stack or registers between requests. That is the problem the next issue solves.

### Thread-Specific Data

**Thread-local storage (TLS)** allows each thread to have its own copy of data. In some ways it is similar to static data, except that the copy is unique to each thread rather than shared by the process.

The lecture notes when this matters: it is useful when the program does not control the thread creation process, for example when using a thread pool, because the thread that runs a piece of code may be a different thread each time.

### Scheduler Activations

Both the many-to-many and the two-level models need communication between the kernel and the thread library to keep the appropriate number of kernel threads allocated to the application. Consider what happens when a user thread is about to block inside the kernel. If the kernel lets it block while it holds the process's only kernel thread, the application stalls even though it has other runnable user threads. The library has no way to know, because it cannot see into the kernel.

The mechanism that solves this is the **scheduler activation**, and it is built on two ideas:

- An **LWP** (lightweight process) is a virtual processor attached to a kernel thread, and it is the object the kernel schedules for a user-level library.
- **Upcalls** are a communication mechanism from the kernel to the thread library, handled by an **upcall handler** in the library.

With upcalls, the kernel can tell the library that a thread is about to block, and the library can then move the runnable user thread to another LWP instead of leaving the process stuck. This is what allows an application to maintain the correct number of kernel threads while the workload changes.

## 9. Thread Libraries and Implementations

The chapter ends with concrete systems, which is a useful reminder that the models above are implemented somewhere.

### Pthreads

**Pthreads** is a POSIX standard (IEEE 1003.1c) API for thread creation and synchronization. The standard specifies the behavior of the thread library, while the implementation is left to the developers of the library. It is common on UNIX systems, including Solaris, Linux, and macOS.

The slide's own question and answer are the important part: *is it a user- or kernel-level library?* It can be either, because Pthreads specifies an interface rather than an implementation.

A typical program creates a thread, lets it compute, and waits for the result:

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

int sum = 0;  /* shared by both threads */

void *runner(void *param) {
    int i, upper = atoi(param);
    sum = 0;
    for (i = 1; i <= upper; i++)
        sum += i;
    pthread_exit(0);
}

int main(int argc, char *argv[]) {
    pthread_t tid;
    pthread_attr_t attr;

    pthread_attr_init(&attr);
    pthread_create(&tid, &attr, runner, argv[1]);
    pthread_join(tid, NULL);

    printf("sum = %d\n", sum);
}
```

`pthread_create()` starts the new thread in `runner()`, and `pthread_join()` makes the parent wait for it. The global `sum` is shared between the two threads, which is safe here only because the parent reads it after the join rather than concurrently.

### Windows XP Threads

Windows XP implements the **one-to-one** mapping. Every thread contains:

- a thread ID;
- a register set;
- separate user and kernel stacks;
- a private data storage area.

The register set, the stacks, and the private storage area together are called the **context** of the thread. The primary data structures are:

| Structure | Role |
| --- | --- |
| ETHREAD | Executive thread block |
| KTHREAD | Kernel thread block |
| TEB | Thread environment block, accessed from user space |

The split between these structures mirrors the split of the analogy above: the kernel-side blocks are what the scheduler works with, while the TEB is the part a user-level library and the thread itself can reach.

### Linux Threads

Linux refers to threads as **tasks** rather than threads. Thread creation is done through the `clone()` system call, which allows a child task to share the address space of the parent task. The flags passed to `clone()` select what is shared, so the same call can produce a process-like task or a thread-like task:

```c
/* a conventional process: nothing is shared */
clone(0, /* ... */);

/* a thread-like task: share the address space */
clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, /* ... */);
```

This is why Linux can treat processes and threads uniformly: both are tasks described by the same structures, and the difference is the set of shared resources. On current Linux systems the Pthreads library is implemented on top of kernel threads in a one-to-one mapping.

### Java Threads

Java threads are managed by the **JVM**. A Java thread may be created in two ways:

- by extending the `Thread` class and overriding `run()`;
- by implementing the `Runnable` interface and passing the object to a `Thread`.

```java
/* 1. Extending Thread */
class Task extends Thread {
    public void run() { /* work */ }
}
new Task().start();

/* 2. Implementing Runnable */
class Work implements Runnable {
    public void run() { /* work */ }
}
new Thread(new Work()).start();
```

Because the JVM is a specification rather than a single implementation, the mapping from Java threads to kernel threads is up to the JVM. On most modern JVMs a Java thread corresponds to one kernel thread, which is the one-to-one model.

Finally, Java defines its own thread states, which do not map exactly onto the process states from OS_3.

![java-thread-states.png](https://tu.shanchui.cc/file/blog/wenzhang/1791879031677_java-thread-states.png)

| State | Meaning | Transition out |
| --- | --- | --- |
| new | The thread object has been created but not started | `start()` moves it to runnable |
| runnable | The thread is eligible to run, which includes the notion of actually running | `sleep()` or waiting on I/O moves it to blocked; finishing `run()` moves it to dead |
| blocked | The thread cannot proceed until an event occurs | The I/O event becomes available, and the thread returns to runnable |
| dead | The thread has finished its `run()` method | Terminal state |

The Java `runnable` state covers both the ready and the running states of the process model. The reason is that Java does not expose the boundary between them: the JVM, not the Java program, decides when a runnable thread is actually executing.

## 10. Connections to Previous Chapters

- The **sports-meet model** from OS_3 described processes. This chapter adds classes to the same picture, so the analogy now covers threads as well.
- A **context switch** between threads of one process is cheaper than a switch between processes, because the address space does not change. Everything OS_3 said about saving and restoring state still applies, only with less to save.
- The **program counter per thread** is the concrete reason a process can have more than one point of execution. OS_1 already noted that a multithreaded process has a separate execution state, including a PC, for each thread.
- A **blocking system call** interacts with the multithreading model. Entering the kernel on behalf of one thread must not silently stall the other threads, which is the problem that kernel threads and scheduler activations solve.
- `fork()`, `exec()`, and `wait()` were introduced in OS_3 as process-control system calls. This chapter shows that `fork()` needs an extra rule once the parent has several threads.
- The course lab on **thread and process scheduling** is where these models become measurable: thread creation cost, switch cost, and the speedup from several cores are exactly the quantities the models predict.

## 11. Takeaways

1. A thread is the basic unit of CPU utilization: a thread ID, a program counter, a register set, and a stack, sharing the code and data of its process.
2. Threads make an application responsive without hand-written polling, and they let one process share memory, create work cheaply, and use several cores.
3. Concurrency is interleaved progress and needs one core; parallelism is simultaneous execution and needs more than one.
4. User threads are managed by a library and are invisible to the kernel; kernel threads are scheduled by the kernel and can run on different cores.
5. The mapping between the two levels defines the four models: many-to-one is cheap but blocks, one-to-one is concurrent but expensive, many-to-many is flexible but complex, and the two-level model adds binding.
6. The class-race analogy shows what the kernel actually schedules: only the threads holding a bib, that is, kernel threads or LWPs.
7. Multithreading creates new questions rather than removing old ones: `fork()` semantics, cancellation, signal delivery, pools, thread-local storage, and kernel-library coordination.
8. Deferred cancellation, thread pools, and TLS all exist because threads are reused and share state; the safe design is the one that defines who owns which data, and when it may be released.
9. Real systems are concrete answers to these questions: Pthreads is an API that can be implemented at either level, Windows XP is one-to-one, Linux calls threads tasks and creates them with `clone()`, and Java leaves the mapping to the JVM.
