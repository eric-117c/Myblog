---
title: 操作系统 OS_2：操作系统结构
published: 2026-09-20
updated: 2026-09-29
pinned: false
description: 操作系统提供的服务、系统调用与系统程序，以及内核结构、虚拟化、系统启动和 RISC-V 内核启动。
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

## 1. Operating-System Services

An operating system sits between applications and hardware. From a user's point of view, it provides a convenient execution environment; from the system's point of view, it manages and protects shared resources.

The services introduced in class can be divided into two groups:

| Goal | Service | What it does |
| --- | --- | --- |
| Help users and programs | User interface | Provides a CLI, GUI, or another way to interact with the system |
| Help users and programs | Program execution | Loads a program into memory, runs it, and handles normal or abnormal termination |
| Help users and programs | I/O operations | Gives programs controlled access to devices |
| Help users and programs | File-system manipulation | Creates, deletes, reads, writes, and organizes files and directories |
| Help users and programs | Communication | Exchanges data between processes, locally or over a network |
| Help users and programs | Error detection | Detects and responds to errors in hardware, I/O, memory, and programs |
| Keep the system efficient and safe | Resource allocation | Shares CPU time, memory, storage, and devices among competing users and processes |
| Keep the system efficient and safe | Accounting | Records resource usage for statistics, quotas, billing, or performance analysis |
| Keep the system efficient and safe | Protection and security | Controls access and defends the system against misuse |

These are not necessarily separate components inside the kernel. They are a useful way to describe what the OS must accomplish.

## 2. User Interfaces

### Command-Line Interface (CLI)

A **command-line interface** accepts commands as text. The command interpreter may be implemented as part of the kernel, but modern systems commonly provide it as a system program, such as a shell.

The shell reads a command, interprets its arguments, and either performs a built-in operation or starts another program. The command itself is therefore not automatically a system call.

### Graphical User Interface (GUI)

A **graphical user interface** presents windows, icons, menus, pointers, and touch controls. It is usually easier to discover and operate than a CLI, while a CLI is often more direct for automation and repeated tasks.

Many operating systems offer both. They are different interfaces to programs and OS services, not different kernels.

### Batch Interface

The service overview also lists **batch processing**. Instead of interacting with each command as it runs, a user prepares a job and its commands in advance. The system executes the batch with little or no interactive input. Batch interfaces remain useful for scheduled and repeatable workloads.

## 3. System Calls

A **system call** is the controlled programming interface through which a process requests a service from the kernel. Application code usually does not invoke the low-level mechanism directly. It calls a library function through an **Application Programming Interface (API)**, and that function invokes a system call when kernel work is required.

The relationship is:

```text
Application → API / library function → system-call interface → kernel service
```

The API is source-level and designed for programmers. The system-call interface is the boundary between user mode and kernel mode. Keeping these layers separate lets libraries offer convenient, portable functions even when kernels use different system-call conventions.

The APIs named in the slides are the **Win32 API**, the **POSIX API** used by UNIX-like systems, and the **Java API** exposed to programs running on the JVM. An API can remain relatively stable even when the underlying system-call set differs.

![Relationship among an API, the system-call interface, and the operating system](https://tu.shanchui.cc/file/blog/wenzhang/1790068124696_image.png)

### Example: Copying a File

Copying a file is one user-level task but requires a sequence of OS services:

1. Obtain the source and destination file names.
2. Open the source file.
3. Create or open the destination file.
4. Repeatedly read a block from the source and write it to the destination.
5. Handle read, write, permission, or storage errors.
6. Close both files and report the result.

![A possible system-call sequence for copying a file](https://tu.shanchui.cc/file/blog/wenzhang/1790067690129_image.png)

### Example: `printf()` and `write()`

In C, `printf()` is a library function rather than a system call. It formats data in user space and normally uses buffering. When output must be sent to the operating system, the library may eventually invoke a call such as `write()`.

```text
printf() → C library formatting and buffering → write() → kernel I/O path
```

One `printf()` call does not always correspond to exactly one `write()` call because buffering and implementation details can change when output reaches the kernel.

![A C program calling printf, which may invoke write](https://tu.shanchui.cc/file/blog/wenzhang/1790068319501_image.png)

### System-Call Dispatch

Each system call is identified by a number. A simplified dispatch path is:

1. The program places the call number and arguments where the ABI expects them.
2. A special instruction transfers control to the kernel.
3. The system-call interface uses the number to select an entry in a dispatch table.
4. The kernel validates the request and performs the operation.
5. The kernel returns a result, status, or error code and restores user execution.

The application does not need to know how the kernel implements the service. This abstraction is one of the main benefits of the interface.

### Parameter Passing

The call number alone is not enough; most calls also need arguments such as an address, a buffer, or a length. Three common parameter-passing methods are:

| Method | Idea | Main limitation or cost |
| --- | --- | --- |
| Registers | Put arguments directly in CPU registers | The number and size of registers are limited |
| Memory block / table | Put arguments in memory and pass the block's address | The kernel must safely access and validate user memory |
| Stack | Push arguments onto a stack for the OS to retrieve | Requires an agreed layout and safe boundary handling |

Real ABIs can combine these methods. For example, small arguments may go in registers while a register points to a larger user-space buffer.

![Passing system-call parameters through a table in memory](https://tu.shanchui.cc/file/blog/wenzhang/1790068854396_image.png)

### Connection to Privilege Modes

This is the practical connection to the user-mode / kernel-mode distinction from OS_1: a system call is an intentional, restricted transition into kernel mode. It does not give the application unrestricted kernel privileges. The kernel still checks the call number, arguments, permissions, and memory addresses before acting.

## 4. Types of System Calls

System calls are commonly grouped by purpose:

| Category | Typical responsibilities |
| --- | --- |
| Process control | Create and terminate processes, load programs, wait, signal events, and manage memory |
| File management | Create, delete, open, close, read, write, reposition, and query file attributes |
| Device management | Request or release devices, perform I/O, and read or set device attributes |
| Information maintenance | Get or set time, system information, and process or file attributes |
| Communication | Create communication channels and send, receive, or share data |
| Protection | Check or change permissions, ownership, identities, and access controls |

The categories overlap. For example, a file can be both stored data and an interface to a device, depending on the OS design.

The slides compare representative Windows and UNIX calls:

| Category | Windows examples | UNIX examples |
| --- | --- | --- |
| Process control | `CreateProcess()`, `ExitProcess()` | `fork()`, `exit()`, `wait()` |
| File management | `CreateFile()`, `ReadFile()`, `WriteFile()` | `open()`, `read()`, `write()`, `close()` |
| Device management | `SetConsoleMode()`, `ReadConsole()` | `ioctl()`, `read()`, `write()` |
| Information maintenance | `GetCurrentProcessID()`, `SetTimer()` | `getpid()`, `alarm()`, `sleep()` |
| Communication | `CreatePipe()`, `CreateFileMapping()` | `pipe()`, `shm_open()`, `mmap()` |
| Protection | `SetFileSecurity()` | `chmod()`, `umask()`, `chown()` |

The names differ, but both systems must solve the same broad classes of problems.

## 5. System Programs

**System programs** provide a convenient environment for program development and execution. Unlike system calls, they are ordinary programs outside the kernel, although they use system calls to do their work.

| Category | Examples of work performed |
| --- | --- |
| File manipulation | Copy, move, rename, print, search, and edit files |
| Status information and debugging | Display time, users, resource use, logs, and diagnostic data |
| File modification | Text editors and tools that transform file contents |
| Programming-language support | Compilers, assemblers, interpreters, libraries, and debuggers |
| Program loading and execution | Loaders, linkers, launchers, and runtime support |
| Communications | Messaging, remote access, browsers, and network utilities |
| Background services | Printing, logging, networking, scheduling, and other daemons |

The boundary to remember is:

```text
System program: user-space tool that provides a convenient environment
System call: controlled entry point used to request kernel work
Kernel: privileged core that manages hardware and protected resources
```

Users often judge an operating system by its system programs and interfaces even though those programs are not the kernel itself.

## 6. Operating-System Design & Implementation

There is no single correct way to design an operating system. The design begins with goals and requirements, then depends on the target hardware, workloads, reliability requirements, and type of system.

### User Goals & System Goals

| Perspective | Typical goals |
| --- | --- |
| User goals | Easy to learn, convenient, responsive, reliable, and safe |
| System goals | Easy to design, implement, maintain, adapt, test, and operate efficiently |

These goals can conflict. Additional abstraction may improve usability and maintainability but add overhead; aggressive optimization may improve performance but make a system harder to reason about.

### Policy vs. Mechanism

- **Policy:** What should be done?
- **Mechanism:** How can it be done?

For CPU scheduling, the timer and context-switch code are mechanisms; the rule that selects the next process is a policy. Separating them allows the scheduling policy to change without rebuilding the underlying switching mechanism.

This principle appears throughout operating systems: access-control rules vs. permission checks, page-replacement choices vs. page-table hardware, and resource limits vs. accounting mechanisms.

The card-key example from class makes the separation concrete. Card readers, electronic locks, and the connection to a security server form the **mechanism**. The database rules deciding who may enter each room and at what time form the **policy**. Changing a person's access should require a policy update, not replacement of every lock.

### Reading Notes: The Early Principles of X

The X Window System provides a stronger example of placing mechanism below policy. The principles quoted in the slides emphasize:

- add functionality only when a real application cannot be completed without it;
- define what the system deliberately does not do, then keep it extensible;
- avoid generalizing before enough examples exist;
- avoid committing to a solution before the problem is understood;
- prefer a simple solution when it obtains most of the desired effect for much less work;
- isolate complexity;
- provide mechanism while leaving user-interface policy to clients.

The lesson is restraint: a small, stable mechanism can support several policies, while policy built into the core becomes harder to replace.

### Implementation Notes

Early operating systems were often written largely in assembly. Modern kernels use a systems language such as C or C++ for most code, with assembly retained where direct hardware control or architecture-specific entry code is necessary.

Higher-level languages generally improve portability, readability, and maintainability. The tradeoff is less direct control over some machine details, although compiler quality means that language level alone does not determine performance.

## 7. Operating-System Structures

An OS structure determines where services run, how components communicate, and how strongly failures are isolated. The labels below describe design tendencies; real operating systems often combine several of them.

### Simple Structure: MS-DOS

MS-DOS was designed to provide substantial functionality in very little space. Its components were not strongly separated into well-protected modules, which kept the design small but made isolation and maintenance difficult.

![Simplified MS-DOS structure](https://tu.shanchui.cc/file/blog/wenzhang/1790070183808_image.png)

### Monolithic Structure: Traditional UNIX

Traditional UNIX can be viewed as two broad parts:

- system programs in user space;
- a kernel containing file systems, CPU scheduling, memory management, device drivers, and the system-call interface.

Kernel services can call one another directly, which offers efficient communication but also creates a large trusted code base.

![Traditional UNIX system structure](https://tu.shanchui.cc/file/blog/wenzhang/1790070375793_image.png)

### Layered Approach

A layered system divides the OS into levels. Each layer uses services from lower layers and provides services to higher ones.

The structure improves modularity and makes interfaces easier to reason about. The difficult part is defining clean layers: some operations do not fit neatly into one direction, and crossing many layers can add overhead.

### Microkernel Structure

A **microkernel** keeps only a small set of essential mechanisms in kernel space, typically including low-level address-space management, thread management, and interprocess communication. Other services run as user-space servers and communicate by messages.

This can improve isolation, extensibility, and portability. Its main challenge is the cost and design complexity of communication and context switching across protection boundaries.

![Communication through a microkernel](https://tu.shanchui.cc/file/blog/wenzhang/1790070459254_image.png)

### Loadable Kernel Modules

A modular kernel has a core kernel plus components that can be loaded when needed. Modules use defined interfaces while still running in kernel space.

This design is flexible and avoids putting every driver into the initial kernel image. It is not the same as a microkernel: a loaded module normally shares the kernel's address space and privileges, so a faulty module can still compromise the whole kernel.

### Hybrid Structure: Darwin / macOS / iOS

Hybrid kernels combine ideas from multiple structures. Darwin's XNU kernel, for example, combines Mach-derived facilities with BSD subsystems and an I/O framework. Calling a kernel “hybrid” describes this mixture; it does not by itself predict performance or reliability.

#### Darwin
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790671619437_image.png)
#### MacOS/iOS
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790671695946_image.png)
### Loadable Kernel Module (LKM)
- implment kernel module which are loadable as needed within the kernel
- ![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790672353991_image.png)
### Exokernel and Unikernel

- An **exokernel** securely multiplexes hardware resources while moving many traditional abstractions into application-level libraries. Applications gain control, but the programming model becomes more specialized.
- A **unikernel** statically links an application with only the OS components it needs into a single-purpose image. It can be small and boot in tens of milliseconds, as the slide notes, but it gives up the familiar general-purpose, multi-user environment.

### Comparison

| Structure | Where most services run | Main strength | Main challenge |
| --- | --- | --- | --- |
| Simple / weakly structured | Little separation | Small and direct | Poor modularity and protection |
| Monolithic | Kernel space | Fast direct calls | Large trusted code base |
| Layered | Ordered layers | Clear organization | Defining layers and avoiding overhead |
| Microkernel | User-space servers around a small kernel | Isolation and extensibility | IPC and boundary-crossing cost |
| Modular | Loadable components in kernel space | Flexible extension with direct calls | Modules retain kernel privilege |
| Hybrid | Mixed | Practical balance for system requirements | More complex architecture |

The most important comparison is not “good vs. bad.” It is **performance, modularity, isolation, compatibility, and complexity** under a particular workload.

## 8. Virtual Machines

**Virtualization** presents a software-defined version of computing resources. It allows several isolated execution environments to share the same physical machine.

### Virtualization & Resource Sharing

A **Virtual Machine Monitor (VMM)**, or **hypervisor**, allocates CPU time, memory, storage, and devices to guest systems. Each guest behaves as though it owns a machine, while the hypervisor mediates access to the real hardware.
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790673091173_image.png)
Virtualization offers isolation, consolidation, snapshots, migration, and reproducible environments. It also introduces implementation challenges: privileged instructions, memory translation, device emulation, I/O performance, and fair resource allocation.

### Types of Hypervisors

| Type                | Placement                           | Typical characteristic                                                 |
| ------------------- | ----------------------------------- | ---------------------------------------------------------------------- |
| Type 1 / bare-metal | Runs directly on hardware           | Controls guest systems without a general-purpose host OS underneath    |
| Type 2 / hosted     | Runs as an application on a host OS | Reuses the host's drivers and services but adds another software layer |
|                     |                                     |                                                                        |
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790673204819_image.png)
This classification describes placement, not an absolute ranking of security or performance.

### The Java Virtual Machine

The **Java Virtual Machine (JVM)** is a process virtual machine: it provides a managed execution environment for one program or language runtime. A system virtual machine instead presents enough virtual hardware to run an entire guest operating system.

```text
JVM: Java program → bytecode runtime → host OS
System VM: applications → guest OS → virtual hardware → hypervisor → hardware
```

### Different Techniques

- **Full virtualization:** presents a complete virtual hardware interface to an unmodified guest OS.
- **Paravirtualization:** lets the guest cooperate with the hypervisor through virtualization-aware interfaces.
- **Hardware-assisted virtualization:** uses processor support to execute guests efficiently while trapping sensitive operations.
- **OS-level virtualization / containers:** isolates user-space environments that share the host kernel; a container is therefore not a full virtual machine.

The comparison diagram can be summarized as follows:

| Technique | Kernel arrangement | Main implication |
| --- | --- | --- |
| Virtual machine | Each VM contains a complete guest OS and kernel above a hypervisor | Strong machine-level isolation with a larger image and startup cost |
| Linux container | Containers share the host OS kernel | Lightweight startup and deployment, but all containers depend on one kernel |
| Unikernel | Each VM contains one application linked with a specialized kernel | Small single-purpose image with less general compatibility |
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790673797326_image.png)
The QEMU-based lab environment connects these ideas to practice: emulation or virtualization gives us a controlled RISC-V machine, while the guest kernel still manages the resources it sees inside that environment.

## 9. Operating-System Generation & System Boot

### System Generation (SYSGEN)

Historically, **SYSGEN** means configuring or generating an OS for a particular machine: available CPUs, memory, devices, drivers, and required features. Modern general-purpose systems detect much of this dynamically, but build-time and boot-time configuration still serve the same purpose.

### Firmware, Bootstrap Loader & Kernel

These components have different responsibilities:

| Component | Responsibility |
| --- | --- |
| Firmware | Performs the earliest platform initialization and selects a boot target |
| Bootstrap / boot loader | Finds the kernel, loads it into memory, supplies boot information, and transfers control |
| Kernel | Initializes memory management, interrupts, devices, scheduling, and the rest of the OS environment |

### Simplified Boot Sequence

```text
Power on / reset
        ↓
Firmware initialization
        ↓
Boot loader
        ↓
Load kernel image and boot data
        ↓
Kernel entry and hardware initialization
        ↓
Start system services and the first user-space process
```

The exact chain depends on the architecture and platform. A real machine can contain several boot-loader stages rather than one.

## 10. Lab Connection: RISC-V Kernel Startup

The RISC-V lab turns the general boot sequence into concrete privilege transitions and control-register operations.

### Startup Path & Privilege Transition

A typical teaching-kernel path is:

1. Execution begins in **machine mode (M-mode)** after reset or after firmware transfers control.
2. Early code establishes a stack and minimal per-hart state.
3. The kernel configures machine control and status registers.
4. It delegates the intended interrupts and exceptions to supervisor mode.
5. It writes the supervisor entry address, `main`, to `mepc`.
6. It prepares `mstatus` so that `mret` returns to **supervisor mode (S-mode)**.
7. It programs the clock to generate timer interrupts.
8. `mret` performs the privilege transition and changes the program counter to `main`.

This is a controlled privilege transition, not an ordinary function return.

### Registers & Initialization

| Register / facility | Role in the startup notes |
| --- | --- |
| `mstatus` | Holds machine-mode status and the privilege mode to return to |
| `mepc` | Holds the address where execution resumes after `mret` |
| `satp` | Selects the supervisor address-translation and protection configuration |
| Trap-vector registers | Point to handlers for interrupts and exceptions at the relevant privilege level |
| Timer facilities | Arrange future timer interrupts for timekeeping and scheduling |

If address translation is disabled during early startup, `satp` is commonly cleared before page tables are ready. The kernel must also initialize trap handling before enabling interrupts that could arrive.

The exact startup path depends on the xv6 or course-kernel version, so register values and delegation details should be checked against the code used in the lab.

## 11. Class Research Topic: OS Support for LLM Inference

The final topic in the slides is an optional research presentation about accelerating LLM inference from an operating-system perspective. The assignment asks for a presentation of about **12 slides**, covering at least one of these directions:

| Inference concern | OS connection |
| --- | --- |
| Model weights and KV cache | Memory allocation, virtual memory, locality, and pressure control |
| CPU, GPU, and accelerators | Heterogeneous resource scheduling and synchronization |
| Pipeline / tensor parallel work | Communication, placement, and load balancing |
| Loading large model files | Storage bandwidth, caching, prefetching, and I/O scheduling |
| Multiple users or agents | Isolation, quotas, accounting, and fairness |

The useful question is not whether an LLM runtime *is* an OS. It is which OS mechanisms and design principles can help it coordinate expensive, shared resources.

The slides describe this as voluntary work for possible classroom discussion. They specify a roughly **15-minute** presentation, submission before **24:00 on November 17**, and possible participation credit. The exact arrangements should still be confirmed against the latest course announcement.

## 12. Takeaways

1. Users usually access OS functionality through interfaces, libraries, and system programs; the system call is the controlled boundary into the kernel.
2. An API call and a system call are not the same thing. A library may do work in user space, buffer operations, or combine several low-level calls.
3. Separating policy from mechanism makes a system easier to change and reuse.
4. Kernel structures trade off performance, modularity, isolation, compatibility, and implementation complexity.
5. Virtual machines add another resource-management layer, while containers isolate environments that still share one kernel.
6. Booting is a staged transfer of control from firmware to a boot loader and then to the kernel; the RISC-V lab exposes the privilege transition behind that process.
