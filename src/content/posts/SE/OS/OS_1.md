---
title: 操作系统 OS_1：课程导读与系统概览
published: 2026-09-15
updated: 2026-09-22
pinned: false
description: 课程安排、操作系统的角色、中断与 I/O、存储层次，以及进程、内存和设备管理。
image: ""
tags:
  - SE
  - 软件工程
  - 操作系统
category: 软件工程
draft: false
author: 山吹
comment: true
date: 2026-09-15
---


## 1. Course Overview

### Background & Objectives

My classroom note: this is the **3.x version** of the OS course. Previously, it was split into **Principles of Operating Systems** and **Operating System Practice**.

The course introduces the internal design of operating systems, their history, and the design methods behind them. The goal is to understand the ideas well enough to use them in future work and enjoy hacking around!

The prerequisites are **C programming, data structures, and computer organization**. The main topics form four groups:

| Topic | What it covers |
| --- | --- |
| Overview | The role of an OS and its internal structure |
| Process management | Processes, threads, CPU scheduling, synchronization, and deadlocks |
| Memory management | Main memory and virtual memory |
| Storage management | File systems, mass storage, and I/O systems |

### Why OS Matters

The lecture connects OS concepts to software performance in the post-Moore era: faster hardware alone does not remove the need for better algorithms, resource management, and efficient software.

These are the connections that stood out in my notes:

- **Agent Sandbox.** The way AI manages your files in a sandbox is highly related to OS!
- **Multi-agent Management.** Limited OS resources vs. an ever-growing number of agents: where should they run, how much can they use, and what happens when one goes wrong?

Scheduling, memory management, permissions, and isolation give us the vocabulary to think about these questions.

### Textbooks & References

The course guide lists *Operating System Concepts*, **10th edition**, by Abraham Silberschatz, Peter B. Galvin, and Greg Gagne. Lecture 1 also retains a reference to the 7th edition, so chapter and page numbers should be checked against the edition being used.

Other references mentioned in the supplied slides include *边干边学 Linux 内核指导* and *xv6: a simple, Unix-like teaching operating system*.

![Textbooks and references from the original course notes](https://tu.shanchui.cc/file/blog/wenzhang/1789461803340_image.png)

### Grading Policy

| Component | Weight |
| --- | ---: |
| Final exam | 50% |
| Homework | 5% |
| In-class quiz / participation | 5% |
| Lab reports | 10% |
| Lab demos | 20% |
| Computer-based exam | 10% + up to 5% bonus |

The regular components add up to **100%**. The bonus is listed separately in the slides.

### Lab Projects

The labs use **RV64 / RISC-V 64**. The weights below are the distribution **among the labs**, separate from the overall course grading table.

| Lab | Focus | Lab weight |
| --- | --- | ---: |
| Lab 0 | Environment setup and kernel compilation | 5% |
| Lab 1 | Kernel boot and clock interrupts | 15% |
| Lab 2 | Thread / process scheduling | 15% |
| Lab 3 | Virtual memory management | 15% |
| Lab 4 | User mode and system calls | 20% |
| Lab 5 | Page faults and `fork` | 20% |
| Lab 6 | File system | 10% |

The course materials require individual work, a report for each lab, and a demonstration to the teaching assistants. Each report includes **Discussion and Reflection**, covering problems encountered and their solutions. The slides assign this section **20% of the lab report grade** and emphasize detailed code comments.

*Course arrangements above are recorded from the supplied slides. The decks contain different lab-document links, including an older Lab 0 reference, so use the course announcements to identify the assigned version.*

### How to Learn: My Notes

- 🤖 **Learn with AI.** It can **speed up** your understanding but cannot **replace** it.
- 👍 **Theory & Practice.**
- 🚀 **Find your interest point.**

The poem I kept from class:

> 山近月远觉月小，便道此山大于月。<br>
> 若有人眼大如天，当见山高月更阔。
>
> —— 王阳明

## 2. What Is an Operating System?

![Chapter 1 outline from the lecture](https://tu.shanchui.cc/file/blog/wenzhang/1789622500069_image.png)

### My Analogy: A Crossroad

> Let's think about a crossroad. Why aren't there more accidents on the road?
>
> This is a problem of **coordination with limited resources for efficient work**.
>
> The driver is the user. The right to pass is the resource. Less waiting time and fewer accidents are preferred. And **YOU** are the OS.

An **operating system** acts as an intermediary between applications / users and the computer hardware. It makes the machine convenient to use while coordinating hardware use efficiently.

Two roles are especially useful to remember:

| Role | Responsibility | In the crossroad analogy |
| --- | --- | --- |
| **Resource allocator** | Decide how CPU time, memory, storage, and devices are shared | Decide who gets to pass and when |
| **Control program** | Control execution and prevent improper use of the machine | Enforce the rules and prevent conflicting movements |

The system can be viewed as **hardware, operating system, system / application programs, and users**. The **kernel** is the core of the OS. Shells and other system programs provide additional facilities around it.

## 3. Computer Startup & Organization

### Bootstrap Program

At power-on or reboot, the processor begins executing startup code. The initial **bootstrap code** resides in firmware, traditionally described as ROM or EPROM in the slides.

The simplified startup sequence is:

1. Initialize the hardware needed for startup.
2. Locate and load the operating-system kernel, possibly through additional boot-loader stages.
3. Transfer control to the kernel so it can initialize the system and start services.

![Computer startup and bootstrap sequence](https://tu.shanchui.cc/file/blog/wenzhang/1789622762253_image.png)

### CPUs, Controllers & Shared Memory

In the lecture's simplified model, one or more CPUs and device controllers connect through a **shared bus** to access main memory. CPUs and devices can operate concurrently, competing for access to memory and the interconnect.

![CPUs and device controllers connected to shared memory](https://tu.shanchui.cc/file/blog/wenzhang/1789624690216_image.png)

> [!IMPORTANT]
> **Concurrency vs. Parallelism**
>
> **Concurrency** means multiple tasks make progress during overlapping periods. Their execution may be interleaved on a single CPU core.
>
> **Parallelism** means tasks execute at the same instant on different execution resources. Concurrent tasks may also run in parallel when the hardware allows it.

A device controller manages a particular kind of device and may have a **local buffer**. The CPU can initiate an I/O operation and do other work while the device operates. An interrupt can notify the CPU when attention is required, such as when an operation completes.

## 4. Interrupts & I/O

### Interrupts, Exceptions & Traps

An interrupt transfers control to a handler, often called an **interrupt service routine (ISR)**. In a vectored design, the interrupt source or number helps select the appropriate handler.

| Term | Main idea | Example |
| --- | --- | --- |
| Hardware interrupt | An event asynchronous to the current instruction stream | A timer event or I/O completion |
| Exception | An event caused by the instruction currently executing | A page fault or an illegal instruction |
| System call | A deliberate request for an OS service through a controlled entry into the kernel | A request to read from a file |

The slides use **trap** for a software-generated transfer caused by an error or a service request. Terminology depends on the architecture: in **RISC-V**, *trap* is the umbrella term for a transfer to a handler caused by either an interrupt or an exception. See the [RISC-V privileged architecture introduction](https://docs.riscv.org/reference/isa/priv/priv-intro.html).

### Interrupt Handling

A simplified handling sequence is:

1. Save the return address and the execution state needed to resume. Hardware and the OS handler share this work.
2. Identify the event using a vector / cause value or by checking possible sources.
3. Run the appropriate handler and update device or process state.
4. Restore the chosen execution context and resume execution. Scheduling may select a different process.

**What if multiple interrupts arrive?** Masking some interrupts during critical parts of handling is one option. Priorities and controlled nesting are also possible. Whether events remain pending, and how they are recorded, depends on the interrupt hardware. Disabling interrupts alone does not create a software queue.

![Interrupt-driven I/O cycle](https://tu.shanchui.cc/file/blog/wenzhang/1789627918398_image.png)

The purpose is to avoid wasting **expensive CPU time** repeatedly waiting for a device. The CPU can run another task while I/O is in progress.

### Waiting for I/O

The slides contrast two simplified approaches:

| Approach | When the caller continues | What happens while I/O is pending |
| --- | --- | --- |
| Wait for completion | After the requested operation completes | The calling task waits. A multitasking OS can schedule another ready task. |
| Continue before completion | After the operation has been submitted | The caller does other work and later checks or receives completion information. |

![Synchronous and asynchronous I/O in the lecture model](https://tu.shanchui.cc/file/blog/wenzhang/1789628332747_image.png)

**A waiting process does not imply an idle CPU.** Also distinguish the caller's interface from the device mechanism: an operation that blocks its caller can still use interrupts internally.

### Device Status & DMA

The OS tracks devices in kernel data structures. A simplified **device-status table** records a device's type, address, and state, along with information needed to manage requests.

![Device-status table with device types, addresses, and states](https://tu.shanchui.cc/file/blog/wenzhang/1789628634180_image.png)

**Direct Memory Access (DMA)** allows a controller to transfer data between a device and main memory without making the CPU copy each byte.

- The CPU / driver sets up the transfer, including the memory area and amount of data.
- The controller performs the data transfer.
- Completion can be reported by an interrupt, after which the OS handles the result.

The slides illustrate **one interrupt per block instead of per byte**. This explains the reduction in CPU overhead, rather than prescribing an interrupt count for every device. DMA still requires CPU involvement in setup and completion handling.

## 5. Storage Hierarchy & Caching

### Storage Hierarchy

Storage is organized into levels with different **speed, capacity, cost per bit, and volatility**.

![Storage-device hierarchy](https://tu.shanchui.cc/file/blog/wenzhang/1789629505228_image.png)

| Level | Role | Typical characteristic |
| --- | --- | --- |
| Registers and CPU caches | Keep immediately needed values near execution units | Small, fast, and volatile |
| Main memory | Hold the working code and data of running programs | Larger than caches and typically volatile |
| Secondary storage | Retain files and data beyond program execution or power loss | Larger, persistent, and slower to access |

For the execution model used in this course, code and data stored on disk must be brought into memory before the CPU can use them, with caches and registers serving the immediate execution path.

### Caching

One important philosophy of storage is **caching**: temporarily copy information into a faster storage level so repeated accesses cost less.

- **Cache hit:** the needed information is already in the cache.
- **Cache miss:** the information must be fetched from a slower level.
- **Replacement policy:** decides what to evict when the cache is full.

Because a cache is smaller than the storage behind it, deciding **what to keep** matters. Multiple copies also raise a correctness question: which copy reflects the current value? In multiprocessor systems, cache-coherence mechanisms help keep cached copies consistent. Shared-program data still needs appropriate synchronization.

## 6. Multiprocessors, Multicore & NUMA

These terms describe different aspects of hardware organization and can overlap in one machine.

| Concept | What to notice | Why the OS cares |
| --- | --- | --- |
| Multiprocessor / SMP | Multiple processors share memory, each with its own execution state | Work can be scheduled across processors |
| Multicore | Multiple processing cores are integrated into one chip | Cores provide parallel execution resources and may share caches |
| NUMA | Memory access cost depends on the processor and memory location | Scheduling and memory placement affect access latency |

**Multiprocessor system**

![Symmetric multiprocessing with shared physical memory](https://tu.shanchui.cc/file/blog/wenzhang/1789630149900_image.png)

**Multicore system**

![Multiple processor cores on a chip](https://tu.shanchui.cc/file/blog/wenzhang/1789630332514_image.png)

**NUMA architecture**

![NUMA nodes and their system interconnect](https://tu.shanchui.cc/file/blog/wenzhang/1789630347116_image.png)

With **Non-Uniform Memory Access**, local memory is generally faster to access than remote memory. A process can therefore be affected by both **where it runs** and **where its data resides**.

## 7. Multiprogramming & Time Sharing

**Multiprogramming** aims to improve CPU utilization. Several jobs are available in memory, so when one waits for I/O, the OS can run another ready job.

**Time sharing** extends this idea to interactive use. The CPU switches among runnable tasks frequently enough to provide responsive service to users.

| | Multiprogramming | Time sharing |
| --- | --- | --- |
| Main goal | Keep the CPU busy | Provide interactive response |
| Central idea | Run another job when the current one waits | Share CPU time through frequent scheduling |
| Main concern | Resource utilization | Responsiveness as well as utilization |

The lecture gives **less than one second** as an illustrative response-time target for interaction. It is not a universal guarantee for every OS or workload.

If several processes are ready, **CPU scheduling** selects which runs next. **Virtual memory** allows a process to execute without its entire address space being resident in physical memory. Swapping and paging are related memory-management techniques to study later.

## 8. Protection, Privilege & Timers

### User Mode & Kernel Mode

An OS must deal with faulty programs, infinite loops, and attempts to access another process's memory or the kernel itself.

**Dual-mode operation** introduces a hardware-enforced distinction:

- **User mode:** applications execute with restricted privileges.
- **Kernel mode:** trusted OS code can perform privileged operations.

A system call provides a controlled entry into the kernel. The kernel checks the request, performs the permitted operation, and returns control appropriately. Privilege modes work together with memory protection and access checks to enforce isolation.

![Transition from user mode to kernel mode during a system call](https://tu.shanchui.cc/file/blog/wenzhang/1789895198596_image.png)

### Timer Interrupts

A **timer interrupt** lets the OS regain control even if a user process never voluntarily yields the CPU.

1. The OS arranges a future timer event.
2. The process runs while hardware keeps track of time.
3. A timer interrupt transfers control to the handler.
4. The OS updates timekeeping and decides whether to reschedule.

The timer does not require the OS to continuously run a countdown loop. For example, the RISC-V machine timer compares `mtime` with `mtimecmp` and makes an interrupt pending when the threshold is reached. See the [RISC-V privileged specification](https://docs.riscv.org/reference/isa/v20240411/_attachments/riscv-privileged.pdf).

An expired time slice normally leads to a scheduling decision, not automatic process termination.

## 9. What the OS Manages

### Process Management

A **process** is a program in execution and a unit of work in the system.

> A program is a **passive entity**.<br>
> A process is an **active entity**.

A process needs CPU time, memory, I/O resources, files, and initialization data. These resources are not owned forever: reusable resources should be reclaimed when the process terminates.

A single-threaded process has one instruction stream and **program counter (PC)**. A multithreaded process has a separate execution state, including a PC, **for each thread**. Threads and processes can run concurrently on one or more CPUs.

The OS is responsible for:

- **Creating and deleting** user and system processes.
- **Suspending and resuming** execution.
- Providing mechanisms for **synchronization** and **communication**.
- Providing support for **deadlock handling**, according to the system's design.

> **Synchronization is about timing; communication is about data movement.**

Here, “timing” includes controlling ordering and access to shared resources. Synchronization can ensure that one operation finishes before another begins, or that only one thread enters a critical section at a time.

### Memory Management

Memory management determines **what is in memory, where it is placed, and who may access it**.

The main activities are:

- Track which regions are in use and which process owns them.
- Decide which process pages and data should reside in memory.
- Allocate and reclaim memory as needed.

The aim is to support correct execution while making good use of memory and maintaining responsiveness.

### File-System Management

The OS provides a uniform, logical view of stored information through the **file** abstraction. Files are organized into **directories**, and access controls determine who can perform which operations.

File-system responsibilities include creating and deleting files and directories, providing read / write and other operations, mapping files onto storage, and supporting persistent storage and backup facilities.

### Mass-Storage Management

Secondary storage holds data that does not fit in main memory or must persist for a long time. Storage performance can strongly affect **I/O-bound workloads**, though the dominant bottleneck depends on the program.

The OS manages **free space**, **storage allocation**, and **I/O request scheduling**. The details depend on the medium: a magnetic disk and an SSD have different performance characteristics.

### I/O Subsystem

The I/O subsystem hides device-specific details behind common interfaces and drivers.

| Technique | Purpose | Example |
| --- | --- | --- |
| **Buffering** | Temporarily hold data during a transfer to accommodate different speeds or transfer sizes | An input buffer accumulating incoming data |
| **Caching** | Retain a copy to make later access faster | Frequently read file contents kept in memory |
| **Spooling** | Queue work for a device that serves requests in sequence | Print jobs waiting in a spool |

These techniques can coexist. Device drivers translate the common interface into the operations required by a particular device.

## 10. Conclusion: My Takeaways

- **Abstraction is the base.**
- **Concurrency brings sharing and multiplexing.**
- **Isolation supports security.**
- **Users matter.**
- **All for performance.**

The performance goal sits alongside correctness, fairness, protection, and ease of use. The interesting part is how an OS balances them when resources are limited.

