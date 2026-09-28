---
title: CN_0
published: 2026-09-23
pinned: false
description: This is an introduction to computer networks
image: ""
tags:
  - CS
  - SE
  - 自学
category: 软件工程
draft: false
author: 山吹
comment: true
date: 2026-09-23
---

> [!NOTE]
> 本系列是完全自学
### Uses of Computer Neteorks

When Computer Tech is at a young age, the idea of a powerful big machine gathering, computing tasks was  deeply roored in everyone's mind.But days wounld change, nobody would expect many years later we are able to vastly produce those powerful machines in the stamp-size materials.

The old model of **a single computer serving all** of the organization’s computational needs has been replaced by one in which **a large number of separate but interconnected computers** do the job. 

These systems are called **computer networks**.

Then, there is the Network's Age. Many small networks are connected into a bigger one, so-called **Internet**.

#### Access to Information

Much Information is accessed using **Client-Server Model**. It's widely used and form the basis of much network usage.


![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790132426602_image.png)

Then, we dive into the model to see details. **Communication** happens on both sides, so we see two **processes** to our first approximation.

![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790132755222_image.png)

Another model is **peer-to-peer** communication. A great example is **BitTorrent**.

We do not have **a centralized database** in this model, instead, each user maintain **a local database of pieces of the content**.

![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790132981472_image.png)
 
#### Person-to-person Communication

> Person-to-person communication is the 21st century’s answer to the 19th century’s telephone.

This kind of communication is devided into **Instant messaging** and **Twitter** services. Former focus on **one-to-one**, while the latter provides **multi-person** capabilities.

Between person-to-person communications and accessing information are **social network applications**. For example, Facebook.

More loosely, communication can take the form of co-created content,like **WIKI**.

#### Other Usage

- E-commerce

![image.png](https://tu.shanchui.cc/file/blog/wenzhang/1790135242022_image.png)

- Entertainment

**IPTV**, **Eletronic Games**

- IoT

> **Ubiquitous computing** entails computing that is embedded in everyday life, as in the vision of Mark Weiser (1991).

**IoT, Internet of Things**

### Types of Computer Networks

```mermaid
graph TD
A[Computer Netowrks] --> B[Broadband Access Networks]
A --> C[Mobile and Wireless Access Networks]
A --> D[Content Provider Networks]
A --> E[Transit Networks]
A --> F[Enterprise Networks]
```

#### Broadband Access Network
>[!NOTE]
>**Metcalfe's Law** : the value of a network is proportional to the **square** of the number of users

Today, Broadband access Network is **proliferating**.

The mian media of Broadband Access  Network is as followed.

| Texture       | Example        |
| ------------- | -------------- |
| copper        | telephone line |
| coaxial cable | cable          |
| optical fiber | optical fiber  |

#### Mobile and Wireless Access Networks
```mermaid
graph LR
A[Wireless Access Network] --> B[mobile uses]
B --> D[personal uses]
B --> E[Enterprise uses]
A --> C[military uses]
```
- For personal uses
	- **Cellular networks**, a wireless network operated by telephone companies
	- **Wireless hotspots**, **802.11 Standard** 
- For Enterprises uses
	- run process of DiDi 
	- boost "sharing economy", like Uber 
- For military uses
	- troops-owned network

> [!IMPORTANT]
> Wireless networking and mobile computing are often **related**, but **not identical**.

| Wireless | Mobile | Typical applications                     |
| -------- | ------ | ---------------------------------------- |
| No       | No     | Desktop computers in offices             |
| No       | Yes    | A laptop coputer used in a hotel room    |
| Yes      | No     | Networks in unwired buildings            |
| Yes      | Yes    | Store inventory with a handheld computer |
