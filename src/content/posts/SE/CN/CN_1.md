---
title: "计算机网络 CN_1 Introduction to CN: basics"
published: 2026-09-16
pinned: false
description: 计算机网络（自学）的第一节课
image: ""
tags:
  - SE
  - CS
  - 网络
category: 软件工程
draft: true
author: 山吹
comment: true
date: 2026-09-16
---
### Before basics
#### DEFINITION
- **Computer Network**: a collection of **interconnected** and **autonomous computing** devices
	- **Interconnect**: the state of computers can **exchange** message (This should be a 2-way road)
- **Internet**: This is the most well-known network of networks
### What uses of computer networks?
>![WARNING]
>There are huge amount of Concepts and Terminology
>Please get ready
#### Client-Server Model
- **Definition**: a client explicitly request information from the server who holds the information
![2026-09-16 11-14-38.png](https://tu.shanchui.cc/file/blog/wenzhang/1789528814359_2026-09-16_11-14-38.png)

- The **Workflow** is simple:
	- Client process sends a request message over the network and wait for reply
	- Server process gets the request, processes it and sends back a reply

- My insight: 
	- Network is about the transmission of information, so there is **sender** and **receiver**
	- so,Work is designed in a **symmetry** way.
	- Like buying coffee, you need to tell the lady about your flavour. Then, the brista can get ready your coffee. (I think it's a very good simile🤗)
![2026-09-16 11-22-36.png](https://tu.shanchui.cc/file/blog/wenzhang/1789529053192_2026-09-16_11-22-36.png)
- **Fallback**:
	- Every design has fallbacks, including Clien-Server Model
		- Let's consider you have 2 clients. EZ😄
		- What about 10000 clients. Maybe🤔
		- Then, 10000000000 clients. Not so good😢
	- So, when too many client request overloaded the server. The Network fails. 
	- Some specific example like, DDoS attack.
- **Solution**: 
	- How can we improve the problem brought by **single point failure**.
		- Multiple Servers 
		- OR, clients help clients 🤔
	- Further question, is there a network working without servers?
		- Yes,and now we are about to introduce it.
#### Peer-to-Peer Communication
- **Definition**: every person in the group can communicate with each other. That is to say, Every machine is both client and server
![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1789535623271_image.png)

- **Example**: BitTorrent. This is a network where every client holds a **local database** of content to **relief the stress of Server**. So, request is sent to not only Repository Server but also client devices who have information pieces.
![2026-09-16 12-59-04.png](https://tu.shanchui.cc/file/blog/wenzhang/1789535497158_2026-09-16_12-59-04.png)
