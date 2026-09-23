---
title: "操作系统 OS_2：操作系统结构"
published: 2026-09-20
pinned: false
description: "ch2 写作框架：操作系统服务、系统调用、系统程序、内核结构、虚拟机与系统启动，正文待补充。"
image: ""
tags:
  - SE
  - 软件工程
  - 操作系统
category: 软件工程
draft: true
author: 山吹
comment: true
---

> [!IMPORTANT]
> **Writing outline** · 对应《2-ch2-new-o》：Operating-System Structures。以下保留章节与写作提示，正文和个人思考待补充。
## 1. Operating System Services

- User Interface
- Program excution
- I/O operations
- File-system manipulation
- Communication
- Error dection
- Resource allocation
- Accounting
- Protection and Security

## 2. User Operating System Interface
### Command-Line Interface (CLI)

CLI allows direct command entry, Somtimes implemented in **kernel**, sometimes by **system program**

### Graphical User Interface (GUI)

GUi is **user-friendly** with the use of icons,mouse etc.

## 3. System Calls

- **Programming interface** to services provided by the OS
- Mostly accessed by programs via a high-level Application Program Interface (so-called,**API**)

### Example: Copying a File

System call sequence to copy the contents of one file to another file
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790067690129_image.png)

### Example: printf() and write()

C program invoking printf() library call, which calls write() system call
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790068319501_image.png)

### System Call Implementation

- Every **system-call** is associated with **a number**, Systlem Call Interface maintains a **table**
- The system call interface invokes intended system call in OS kernel and **returns status of the system call** and any return values
- Details are hidden

API - System Call - OS Relationship
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790068124696_image.png)


### Parameter Passing

Systemcall needs to pass more information

- Threee general methods used to pass parameters to OS
	- pass the parameters in **registers**
	- Parameters stored in a **block**, or table, in memory
	- Parameters placed, or pushed, onto the stack by the program and popped off the **stack** by the operating system

Paramater Passing via Table
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790068854396_image.png)
### My Thoughts

<!-- 留给自己：与第一篇 user mode / kernel mode 的联系。 -->

## 4. Types of System Calls

-  Process Control
-  File Management
-  Device Management
-  Information Maintenance
-  Communications
-  Protection

## 5. System Programs

System programs provide a convenient environment for program development and execution.

System Programs Classification
-  File Manipulation & Modification
- Status Information & Debugging
- Programming-Language Support
- Program Loading & Execution
- Communications

### My Thoughts

<!-- 留给自己：system programs、system calls 和 kernel 的边界。 -->

## 6. Operating System Design & Implementation

The Design Philosophy of OS is open. 

First,start with goals and specifications

Then, Affected by choice of hardware,type of system

### User Goals & System Goals

User Goals: easy to learn, convenient to use
System Goals: easy to design, implement and maintain

### Policy vs. Mechanism

**Policy:**   What will be done? 
**Mechanism:**  How to do it?

### Reading Notes: The Early Principles of X

<!-- 待写：选择 slide 33 中感兴趣的原则，记录自己的理解。 -->

### My Thoughts

<!-- 留给自己：一个自己遇到过的“策略与机制分离”的例子。 -->

## 7. Operating System Structures

### Simple Structure: MS-DOS

MS-DOS – written to provide the most functionality in the least space

![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790070183808_image.png)

### Monolithic Structure: UNIX

- UNIX consists of 2 separate parts:
	- System programs
	- The Kernel

![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790070375793_image.png)

### Microkernel Structure

![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790070459254_image.png)

### Hybrid Structure: Darwin / macOS / iOS

### Layered Approach

### Loadable Kernel Modules

### Exokernel

### Unikernel

### Comparison & My Thoughts

<!-- 留给自己：完成各节后再整理比较表；区分模块化与微内核，避免只给结构贴“好 / 坏”标签。 -->

## 8. Virtual Machines

<!-- 课件 slides 50–56。 -->

### Virtualization & Resource Sharing

### Types of Hypervisors

<!-- 待写：根据 slide 53 补全分类及结构图。 -->

### The Java Virtual Machine

<!-- 待写：与运行完整操作系统的系统虚拟机作区分。 -->

### Different Virtualization Techniques

<!-- 待写：结合 slide 55 的图整理技术差异。 -->

### Benefits, Isolation & Implementation Challenges

### My Thoughts

<!-- 留给自己：联系 QEMU 实验环境，以及第一篇提到的 sandbox。 -->

## 9. Operating System Generation & System Boot

<!-- 课件 slides 58、60。 -->

### System Generation (SYSGEN)

### Firmware, Bootstrap Loader & Kernel

### Boot Sequence

<!-- 待写：系统如何定位、加载并进入内核；可画启动顺序图。 -->

### My Thoughts

## 10. Lab Connection: RISC-V Kernel Startup

<!-- 课件 slides 64–65，为课末实验衔接内容。 -->

### Startup Path & Privilege Transition

<!-- 待写：结合实际实验代码记录启动路径、machine mode / supervisor mode 以及 mret。 -->

### Registers & Initialization

<!-- 待写：记录课件示例中的 mstatus、mepc、satp，以及中断、异常和时钟初始化。区分课件 xv6 示例与自己使用的内核版本。 -->

### Debugging Notes & My Thoughts

<!-- 留给自己：实际断点、观察结果、遇到的问题与解决过程。 -->

## 11. Optional Reading: OS & LLM Inference

<!-- 课件 slides 61–62 的拓展研讨主题，仅保留选题位置。 -->

### Memory Management & KV Cache

### Heterogeneous Scheduling & Pipeline Parallelism

### Storage I/O & Model Loading

### My Questions

<!-- 留给自己：有兴趣再选一个方向展开。 -->

## 12. My Takeaways

<!-- 留给自己：学完本章后的总结、最有启发的设计思想、仍没弄懂的问题。 -->

## References

- 《2-ch2-new-o》：Chapter 2, Operating-System Structures。

<!-- 后续补充实际阅读过的书籍、文档和文章。 -->
