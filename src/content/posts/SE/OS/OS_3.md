---
title: 操作系统 OS_3：进程
published: 2026-09-29
pinned: false
description: 进程的内存布局、状态与 PCB，进程调度和创建，以及共享内存、消息传递、Socket、RPC 与 RMI。
image: ""
tags:
  - SE
  - 软件工程
  - 操作系统
category: 软件工程
draft: false
author: 山吹
comment: true
date: 2026-09-29
---

## 1. Process Concept

An operating system executes many kinds of work. A batch system runs jobs, while a time-sharing system runs interactive user programs or tasks. The textbook often uses *job* and *process* almost interchangeably, but **process** is the central term in this chapter.

A **program** is passive code stored in a file. A **process** is an active instance of a program in execution. Several processes can run the same program while maintaining different execution states and data.

> [!IMPORTANT]
> A process is more than its executable code. It also includes its current CPU state, address space, open resources, and the information the OS needs to manage it.

### Process Address Space

A process normally contains these memory regions:

| Region | Main contents |
| --- | --- |
| Text | Executable instructions, usually read-only |
| Data | Global and static variables |
| Heap | Memory allocated dynamically while the program runs |
| Stack | Function parameters, return addresses, and local variables |

The process also has a **program counter**, which identifies the next instruction to execute, and CPU registers that hold its current computation state.

![process-memory.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674887795_process-memory.png)

The diagram shows the heap growing toward higher addresses and the stack growing toward lower addresses. This is a common conceptual layout, not a promise that every platform uses exactly the same addresses or growth directions.

## 2. Process States

A process moves through several states during its lifetime:

| State | Meaning |
| --- | --- |
| New | The OS is creating the process |
| Ready | The process is in memory and waiting for a CPU |
| Running | A CPU is executing the process's instructions |
| Waiting / blocked | The process cannot continue until an event occurs, often I/O completion |
| Terminated | The process has finished execution |

![process-state.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674578929_process-state.png)

The transitions explain an important distinction:

- **Ready** means the process could run but does not currently have a CPU.
- **Waiting** means the process cannot run yet, even if a CPU is available.

A scheduler dispatch moves a ready process to running. An interrupt or expired time slice can return it to ready. An I/O request moves it from running to waiting, and the completion event makes it ready again.

### The Sports-Meet Analogy

The lecture uses a sports meet to make the states concrete:

| Operating-system concept | Sports-meet model |
| --- | --- |
| Process | Athlete |
| Process ID | Athlete number |
| CPU core | Running track |
| Running | Athlete occupies a track and runs |
| Waiting for I/O | Athlete leaves to get water and cannot race yet |
| Ready | Athlete can run but waits for a free track |
| Saved context | The recorded position needed to resume the race |

Each track can hold one athlete at a time, while a stadium may contain one or several tracks. Other athletes can use a track while one athlete waits for water. This is the same reason an OS schedules another ready process when the current process blocks for slow I/O.

![sports-meet-model.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674600410_sports-meet-model.png)

## 3. Process Control Block & Context Switch

### Process Control Block

The OS stores the information associated with a process in a **Process Control Block (PCB)**. A PCB typically records:

- process state and process identifier;
- program counter and CPU registers;
- CPU-scheduling information;
- memory-management information;
- accounting and resource-usage information;
- I/O status, including open files and allocated devices.

The PCB is the kernel's representation of the process. When the process stops using a CPU, the OS needs this information to continue it later.

### Context Switch

A **context switch** changes the CPU from one process to another:

1. An interrupt, exception, or system call transfers control to the kernel.
2. The kernel saves the current process's CPU state into its PCB.
3. The scheduler selects another runnable process.
4. The kernel restores that process's saved state from its PCB.
5. Execution resumes from the restored program counter.

![context-switch.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674634157_context-switch.png)

Context-switch time is overhead because the CPU is managing execution rather than advancing either user's computation. Its cost depends on the architecture, kernel path, cache effects, and amount of state that must be saved. The lecture slide uses milliseconds as an illustrative scale, so it should not be treated as a universal value for modern systems.

> [!NOTE]
> A mode switch and a context switch are related but different. Entering the kernel does not always change the running process. The kernel may handle a short system call and return to the same process without scheduling another one.

## 4. Process Scheduling

Multiprogramming keeps several processes available so that the CPU can run useful work when another process waits. The scheduler chooses among those processes.

### Scheduling Queues

The slides introduce three queue concepts:

| Queue | Processes it contains |
| --- | --- |
| Job queue | All processes submitted to or known by the system |
| Ready queue | Processes in main memory that are ready to execute |
| Device queue | Processes waiting for a particular I/O device or event |

Processes migrate among queues as their state changes. A running process might request I/O, exhaust its time slice, create and wait for a child, or wait for an interrupt.

![scheduling-queues.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674761810_scheduling-queues.png)

### Long-, Short-, and Medium-Term Schedulers

| Scheduler | Decision | Frequency and concern |
| --- | --- | --- |
| Long-term / job scheduler | Which jobs enter memory and the ready queue | Runs infrequently and controls the degree of multiprogramming |
| Short-term / CPU scheduler | Which ready process runs next | Runs very frequently, so it must be fast |
| Medium-term scheduler | Which processes should be temporarily swapped out or brought back | Reduces memory pressure and adjusts the active process set |

The lecture notes that classic UNIX and Windows systems do not use a distinct long-term scheduler in the textbook sense. The concept still helps explain admission control in systems that decide which jobs may enter active execution.

![medium-term-scheduling.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674772057_medium-term-scheduling.png)

### CPU-Bound vs. I/O-Bound Processes

- An **I/O-bound process** performs many short CPU bursts and frequently waits for I/O.
- A **CPU-bound process** performs fewer but longer CPU bursts.

A useful workload mix prevents either the CPU or I/O devices from sitting idle unnecessarily. The long-term scheduling model therefore considers both the number and the behavior of admitted processes.

## 5. Operations on Processes

### Process Creation

A process can create child processes, producing a process tree. The creator is the **parent**, and each created process is a **child**.

The parent and child relationship raises two separate design questions:

| Question | Possible choices |
| --- | --- |
| Resource sharing | Share all resources, share selected resources, or share none |
| Execution | Run concurrently, or make the parent wait for the child |
| Address space | Duplicate the parent's address space, or load a different program |

### `fork()`, `exec()`, and `wait()`

UNIX separates process creation from program loading:

- `fork()` creates a child process based on the calling process.
- An `exec`-family call replaces the current process image with a new program.
- `wait()` lets a parent wait for a child to terminate and collect its status.
- `exit()` terminates the calling process.

![process-creation.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674804657_process-creation.png)

The key point is that `exec()` does **not** create another process. After a successful `exec()`, the same process identity continues with a replaced program image.

A simplified version of the lecture example is:

```c
pid_t pid = fork();

if (pid < 0) {
	fprintf(stderr, "fork failed\n");
	exit(EXIT_FAILURE);
} else if (pid == 0) {
	execlp("/bin/ls", "ls", NULL);
	exit(EXIT_FAILURE); // reached only if exec fails
} else {
	wait(NULL);
	printf("child complete\n");
}
```

Both parent and child continue after `fork()`, but they observe different return values. The child takes the `pid == 0` branch and tries to load `ls`; the parent waits and continues after the child finishes.

### Process Termination

A process normally terminates by calling `exit()` or returning from its entry function. The OS then releases most of its resources. The parent can use `wait()` to receive the child's termination status.

A parent or the OS may also terminate a child because the child exceeded a resource limit, its work is no longer required, or the surrounding job is being stopped. Some systems perform **cascading termination** when a parent exits.

If a parent terminates while its child continues, another system process can adopt the **orphaned process**. This differs from a **zombie**: a zombie has already terminated, but its parent has not yet collected its status.

The final macOS example in the slides identifies `launchd` as the PID 1 service manager that fills the role traditionally associated with `init`.

## 6. Cooperating Processes

An **independent process** cannot affect or be affected by other processes. A **cooperating process** shares information or coordinates execution with another process.

Cooperation is useful for:

- sharing information;
- speeding up suitable work across several CPUs;
- organizing a system into modules;
- allowing related tasks to work together conveniently.

Cooperation also introduces correctness problems. Once processes share data or depend on message order, the system must define synchronization and failure behavior.

### Producer-Consumer Problem

The **producer** creates data and the **consumer** uses it. A buffer lets them run at different speeds.

| Buffer model | Producer waits when | Consumer waits when |
| --- | --- | --- |
| Unbounded buffer | Never because of capacity | The buffer is empty |
| Bounded buffer | The buffer is full | The buffer is empty |

The lecture uses a circular array with indices `in` and `out`:

```c
#define BUFFER_SIZE 10

item buffer[BUFFER_SIZE];
int in = 0;
int out = 0;
```

The basic condition is:

```c
// empty
in == out

// full when one slot is deliberately left unused
(in + 1) % BUFFER_SIZE == out
```

Leaving one slot empty separates the full and empty states, so an array of size `BUFFER_SIZE` stores at most `BUFFER_SIZE - 1` items.

### Why the First Solution Is Incomplete

The slide pseudocode uses loops that repeatedly test whether the buffer is full or empty. This is **busy waiting** and wastes CPU time while no progress can be made.

The final classroom slides explore two tempting changes:

1. Moving `buffer[in] = item` before the full-buffer check can overwrite an item that the consumer has not removed.
2. Adding a shared `count` distinguishes full from empty and allows all slots to be used, but plain `count++` and `count--` operations can overlap and lose an update.

The example therefore leads directly to synchronization. Shared state needs atomic operations, locks, semaphores, or another correct coordination mechanism. Source order alone does not make concurrent access safe.

## 7. Interprocess Communication

**Interprocess Communication (IPC)** provides mechanisms for processes to exchange information and synchronize their actions. The two main models are shared memory and message passing.

![ipc-models.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674813557_ipc-models.png)

| Model | How data moves | Main responsibility |
| --- | --- | --- |
| Shared memory | Processes read and write a mapped memory region | Processes must coordinate concurrent access |
| Message passing | Processes use `send` and `receive`; the OS or runtime transfers messages | The communication mechanism defines delivery and buffering behavior |

Shared memory can avoid repeated kernel-mediated copies after setup and works well for high-volume local communication. Message passing offers a clearer communication boundary and also works across machines, but the implementation may add copying, queueing, and context-switch costs.

### Communication-Link Questions

An IPC design must answer:

- How is a link established?
- Can more than two processes use one link?
- Can a pair of processes share several links?
- Does a link have finite capacity?
- Are message sizes fixed or variable?
- Is communication unidirectional or bidirectional?

### Direct Communication

With **direct communication**, processes explicitly name one another:

```text
send(P, message)
receive(Q, message)
```

The system can establish the link automatically between the named pair. The textbook model associates one link with one pair of processes, usually allowing communication in both directions.

### Indirect Communication

With **indirect communication**, processes send to and receive from a mailbox or port:

```text
send(A, message)
receive(A, message)
```

A link exists when processes share mailbox `A`. One mailbox may connect several processes, and two processes may share several mailboxes.

Shared mailboxes create a choice: if one process sends while several processes are waiting to receive, which receiver gets the message? Possible rules include restricting a link to two processes, allowing only one active receiver, or letting the system choose and report the selected receiver.

### Synchronization

Message operations can be blocking or non-blocking:

| Operation | Blocking / synchronous behavior | Non-blocking / asynchronous behavior |
| --- | --- | --- |
| Send | Sender waits until the message is accepted or received | Sender submits the message and continues |
| Receive | Receiver waits until a message arrives | Receiver returns immediately with a message or an empty result |

The exact completion point of a blocking send depends on the IPC API. “Accepted into a queue” and “consumed by the receiver” are different guarantees.

### Buffering

Messages waiting on a link may use one of three capacities:

| Capacity | Behavior |
| --- | --- |
| Zero | No message can wait in the link; sender and receiver must rendezvous |
| Bounded | Up to `n` messages can wait; a sender blocks or fails when the queue is full |
| Unbounded | The conceptual queue has no fixed limit; the sender does not wait for capacity |

Real systems always have finite resources, so “unbounded” describes the interface model rather than physically infinite storage.

## 8. Client-Server Communication

The chapter closes with three mechanisms used in client-server systems: **sockets**, **Remote Procedure Calls (RPC)**, and Java **Remote Method Invocation (RMI)**.

### Sockets

A **socket** is a communication endpoint. In Internet networking, an endpoint is commonly identified by an IP address and a port number. Communication occurs between a pair of sockets.

![socket-communication.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674833219_socket-communication.png)

The server's well-known port identifies a service, while the client usually uses a temporary port. The socket abstraction exposes communication but does not by itself define the application's message format or semantics.

### Remote Procedure Call

**RPC** makes a request to another process look similar to a local procedure call. The resemblance is useful, but a remote call still has network latency, partial failures, serialization limits, and different retry semantics.

A typical RPC path is:

1. The client calls a local **stub**.
2. The stub marshals the procedure identifier and arguments into a message.
3. The communication layer locates the server and transmits the request.
4. A server stub unmarshals the request and calls the server procedure.
5. The result follows the reverse path back to the client.

![rpc-execution.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674848032_rpc-execution.png)

**Marshalling** converts parameters into a representation that can cross an address-space or machine boundary. The receiver performs **unmarshalling** to reconstruct usable values. This step must account for data formats, byte order, object identity, and values that cannot be transferred directly, such as a raw pointer into the sender's memory.

### Remote Method Invocation

Java **RMI** applies a similar idea to remote objects. A Java program invokes a method through a local proxy, and the runtime communicates with the remote object. Unlike a basic procedural RPC interface, RMI is object-oriented and works with remote object references.

![rmi-marshalling.png](https://tu.shanchui.cc/file/blog/wenzhang/1790674855312_rmi-marshalling.png)

The slide uses the historical term **skeleton** for the server-side dispatcher and notes that Java 2 version 1.2 removed the requirement for generated skeleton classes in the standard RMI model.

## 9. Connections to Previous Chapters

This chapter links several ideas from OS_1 and OS_2:

- A timer interrupt can move a running process back to ready so the scheduler can select another process.
- A system call enters the kernel on behalf of the current process. It may return to the same process or block and trigger a context switch.
- The PCB contains the saved state required to resume execution after an interrupt or scheduling decision.
- IPC uses protected OS mechanisms to let isolated processes cooperate without giving unrestricted access to one another.
- `fork()`, `exec()`, `wait()`, and `exit()` are concrete examples of the process-control system calls introduced in OS_2.

## 10. Takeaways

1. A process is a running program together with its execution state, address space, and OS-managed resources.
2. Ready and waiting are different: a ready process needs a CPU, while a waiting process needs an event.
3. The PCB makes suspension and resumption possible; a context switch saves one PCB state and restores another.
4. Scheduling queues describe how processes move among CPU execution, I/O waiting, and temporary suspension.
5. UNIX separates process creation with `fork()` from program replacement with `exec()`.
6. Shared-memory performance does not remove the need for synchronization. The bounded-buffer example shows how small reorderings can introduce lost data or races.
7. Message passing must define naming, synchronization, and buffering semantics.
8. Sockets expose communication endpoints, while RPC and RMI add higher-level call abstractions and parameter marshalling.
