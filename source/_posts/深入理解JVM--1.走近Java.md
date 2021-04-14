---
title: 深入理解JVM--1.走近Java
date: 2020-09-27 23:36:44
tags: 
  - JVM
categories: 
  - 深入理解JVM
description: Java不仅仅是一门编程语言，它还是一个由一系列计算机软件和规范组成的技术体系
comments: true
top_img: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img11.jpg
cover: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img11.jpg
---
### 概述
Java不仅仅是一门编程语言，它还是一个由一系列计算机软件和规范组成的技术体系，这个技术体系提供了完整的用于软件开发和跨平台部署的支持环境，并广泛应用于嵌入式系统、移动终端、企业服务器、大型机等多种场合。
Java优势：
- 结构严谨、面向对象
- 摆脱硬件平台的束缚，实现“一次编写、到处运行“
- 提供相对安全的内存管理和访问机制，避免绝大部分内存泄漏和指针越界问题
- 实现了热点代码检测和运行时编译及优化，使得Java应用能随着运行时间的增长而获得更高的性能
- 有一套完善的应用程序接口，还有无数来自商业机构和开源社区的第三方类库来帮助用户实现各种各样的功能

### Java技术体系
组成部分：
1. Java程序设计语言
2. 各种硬件平台上的Java虚拟机实现
3. Class文件格式
4. Java类库API
5. 来自商业机构和开源社区的第三方Java类库
Java程序设计语言、Java虚拟机、Java类库这三部分统称为**JDK（Java Development Kit），JDK是用于支持Java程序开发的最小环境**。
Java类库API中的Java SE API子集和Java虚拟机这两部分统称为**JRE（Java Runtime Environment），JRE是支持Java程序运行的标准环境**。

产品线：
1. Java Card：支持Java小程序（Applets）运行在小内存设备（如智能卡）上的平台。
2. Java ME(Micro Edition)：支持Java程序运行在移动终端（手机、PDA）上的平台。JDK 6以前被称为J2ME。备注：智能手机上Java开发的Android应用不属于Java ME。
3. Java SE(Standard Edition)：支持面向桌面级应用（如Windows下的应用程序）的Java平台。JDK 6以前被称为J2SE。
4. Java EE(Enterprise Edition)：支持使用多层架构的企业应用（如ERP、MIS、CRM应用）的Java平台。JDK 6以前被称为J2EE，在JDK 10以后被Oracle放弃，捐献给Eclipse基金会管理，此后被称为Jakarta EE。

