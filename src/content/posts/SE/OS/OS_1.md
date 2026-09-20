---
title: 操作系统 OS_1
published: 2026-09-15
pinned: false
description: 操作系统的第一节课
image: ""
tags:
  - SE
  - 软件工程
category: 软件工程
draft: false
author: 山吹
comment: true
date: 2026-09-15
---

### Overview 
#### history of OS course in ZJU
- This is the 3.x of OS course
- previously 2 modules including **Principles of OS** and **OS Practice**
#### INTRO for course
- basics of internal design
- history and state-of-the-art 
- design methodologies
- prepare to practice techs in future works and hacking around
#### INTRO for content
- Overview (history and classification)
- Process
- Memory
- Storage
#### WHY OS matters
- Entering post-Moore Era, simply enhancing hardware ability is **not** enough to match the booming needs of computer applications
- Agent Sandbox. The way AI manage your files in sandbox is highly related to OS!
- Multi-agent Management. Limited OS resource V.S. Unlimited number of Agents
#### Recommend  reference book
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789461803340_image.png)
#### Grading Policy
| terms                | grades            |
| -------------------- | ----------------- |
| Final Exam           | 50%               |
| Assignment           | 5%                |
| Quiz                 | 5%                |
| Lab Report           | 10%               |
| Lab Demo             | 20%               |
| Computer-baesd Exams | 10%+***5%BONUS*** |
#### Lab Projects
(TO BE)
### How to learn
- 🤖Learn with AI. They can **speed** your understanding but can't **replace** it.
- 🤔A poem 
```poem
山近月远觉月小，便道此山大于月。
若有人眼大如天，当见山高月更阔。
By 王阳明
```
- 👍Theory & Practice 
- 🚀Find your interest point 

### Chp.1
#### Outline
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789622500069_image.png)
#### What is operating system?
>Let's think about a crossroad.Why there is very less accident on road?
  This is a problem of ***coordination with limited resource for efficient work***.
  Driver is the user, the previlege to pass is resource,less time and accidents is preferred,
  and **YOU** are OS.
  
- So,there is a metaphor.
- OS is a program that acts as an **intermeditary** between a user of computer and the hardware.
- OS is an **allocator** who manages all resources and a **controller** who prevent invalid use to the machine.
#### ***bootstrap*** program
- Bootstrap is program loaded at power-up or reboot.
- It is typically strored in ROM or EPROM, known as firmware.

The startup procedure is as followed.
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789622762253_image.png)

#### Computer System Organization
- Computer-system operation
	- One or more CPUs, device controllers connect through common bus providing access to shared memory
	- **Concurrent execution** of CPUs and devices competing for memory cycles
	- Device *connected* by BUS![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789624690216_image.png)

>![IMPORTANT]
>Parallel v.s. Concurrent
>Parallel : 2 programs run at a specific time
>Concurrent : 2 programs is  high possibly not running together at a specific time

- I/O devices and CPU run concurrently
	-  every devices controller has a local buffer
	- device controller cause an **interrupt** in system bus to inform CPU a job finished
	- To avoid wasting the **EXPENSIVE CPU RESOURCE**
- An OS is **interrupt driven**
	- Interrupt transfers control to the interrupt service routine (**ISR**) generally, through the **interrupt vector**, which contains the addresses of all the service routines.
	- What if multiple interrupts?
		- **DISABLE** is a solution. we can maintain a queue. 
		- Actually, more solutions are available.
	- **Trap**: software-generated interrupt caused by either an error or user request(often referred to as  a system call)
- Interrupt handling
	- OS presevers the state of CPU by stroing **registers** and **program counter** (PC)
	- Then, determine which type of interrupt has occuered:
		- **Polling** by a generic routine
		- **Vectored** interrupt system 
	- Interrupt driven I/O Cycle![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789627918398_image.png)

#### I/O Structure
- 2 I/O Methods
	- After I/O starts, control returns to user program only upon I/O completion.
	- After I/O starts, control returns to user program without waiting for I/O completion.
	- 2 methods showed below![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789628332747_image.png)
#### Device Management
- We preserve device-status in kernel block of memory
- Device-Status Table![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789628634180_image.png)
- Direct Memory Access (DMA) Structure
	- Device controller transfers blocks of data from buffer storage directly to main memory **without** CPU intervention.
	- Only one interrupt is generated **per block**, rather than the one interrupt per byte.
- Storage Structure
	- Storage-Device Hierarchy![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789629505228_image.png)
	- envaluate a storage hardware
		- speed; cost; volatility
	- The most important philosophy of storage is **caching**.
		- Copy information to faster storage system
-  Multiprocessor System![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789630149900_image.png)
- MultiCore System![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789630332514_image.png)
- NUMA Architecture
 ![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789630347116_image.png)
#### Taxonomy
- **Multiprogramming** is designed to maximize CPU utilization
	- Single user cannot keep CPU and I/O devices busy at all times 
	- Multiprogramming organizes jobs (code and data) so CPU always has one to execute A subset of total jobs in system is kept in memory 
	- One job selected and run via job scheduling 
	- When it has to wait (for I/O for example), OS switches to another job
- **Timesharing** (multitasking) CPU switches jobs frequently to enhance responsibility
	- Response time should be < 1 second 
	- Each user has at least one program executing in memory --> process 
	- If several jobs ready to run at the same time --> CPU scheduling 
	- If processes don’t fit in memory, swapping moves them in and out to run Virtual memory allows execution of processes not completely in memory

#### Operating-System Operations
- Interrupts driven by hardware happen at any time
	- Software error or Request creates **exception** or **trap**
	- Other problems : Infinite loops, processes modifying each other or OS
- Then, protection is needed
	- **Dual-mode operation** allows OS to protect itself and other system components
	- **User mode** and **Kernel mode**
	- ![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789895198596_image.png)

#### Time & Interrupt
- **Timer** to prevent infinite loop / process hogging resources 
	- Set interrupt after specific period 
	- Operating system decrements counter 
	- When counter zero generate an interrupt 
	- Set up before scheduling process to regain control or terminate program that exceeds allotted time

#### Process Management

- **Definition** : A process is a program in execution. It's a unit of work within the system.

> Program is a PASSIVE entity.
> Process is an ACTIVE entity.

- Process needs resource
	- CPU, memory, I/O, files
	- Initialization data

- BUT, resource is never eternally owned by the process.
	- resouce is reclaimed when process termination.

- Single-threaded Process has 1 **program counter** to specify location of **next instruction** to execute.
- Muti-threaded Process has  1 program counter **per thread**.

- System has many processes, some user, some OS running concurrently on **one or more** CPUs.
	- Concurrency by **multiplexing the CPUs** among the processes / threads.

#### Process Management Activities
- So,OS is responsible for following activities in connection with management :
	- **create** and **delete** both user and system processes
	- **suspend** and **resume** processes
	- Providing mechanisms for **synchronization, communication** and **deadlock handling**.

> Synchronization is about **timing**; communication is about **data movement**.

#### Memory Management
> Prequisite:
> 	All data must be in memory before and after processing
> 	All instructions must be in memory in order to execute

- So, Memory management determines what is memory to
	- Optimize CPU utilization and computer response 

- Memory management activities
	- **Keeping track** of which parts of memory are currently being used and by whom 
	- Deciding **which processes** (or parts thereof) and data to move into and out of memory 
	- **Allocating and deallocating** memory space as needed

#### Storage Management
- OS should provide a uniform and logical view of inforamtion storage
	- File --> a logical abstract concept of a unit of physical properties
	- Every physical storage medium is controlled by device

- This abstraction leads to **File-System management**
	- Files usually organized into **directories**
	- **Access control** on most systems to determine who can access what

- Storage management activities

#### Mass-Storage Management

- Entire **speed** of computer operation hinges on disk

#### I/O Subsystem
- One purpose of OS is to hide peculiarities of hardware devices from the user – ease of usage & programming 
- I/O subsystem responsible for 
	- Memory management of I/O including buffering (storing data temporarily while it is being transferred), caching (storing parts of data in faster storage for performance), spooling (the overlapping of output of one job with input of other jobs) 
	- General device-driver interface 
	- Drivers for specific hardware devices

### Conclusion
- Abstraction is the base
- Concurrency brings Sharing and Mutiplexing
- Isolation guarantees Security
- Users 
- All for performance