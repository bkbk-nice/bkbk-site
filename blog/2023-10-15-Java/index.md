---
slug: Java
title:  Java 
authors: [bkbk]
tags: [Java]
---

:::note 目录
**Java**   
   
::: -->
<!--truncate-->
### 从软件一般实践原则理解Java

:::info
The Feel of Java - James Gosling 1997   
   Java is a blue collar language. It's not PhD thesis material but a 
languagefor a job.Java feels very familiar to many different programmers
 becausewe preferred tried-and-tested things.  
-  Distributed objects,Architecture neutral, Garbage collection,0bject orientation with latehindino PFREORMANICE(JIT) Java Virtual Machine foruced on safety
:::

对抗偶然复杂度(Accidental Complexity)-面向对象&函数式编程 
:::info
No Silver Bullet—Essence and Accident in Software Engineering
Fred Brooks，1986
No Silver Bullet
—Essence and Accident in Software Engineering
redierick PBrooks. Jr.
aiverisity ofNorth Carolina at Clapel Itil
Abstract1
All software construction involves essential tasks,the fashioning of the complexconceptual structures that compose the abstract software entity,and accidental tasks, therepresentation of these abstract entities in programming languages and the mapping ofthese onto machine languages within space and speed constraints. Most of the big past   
本质复杂度(Essential Complexity)与偶然复杂度(Accidental Complexity)

·最早1966 or 1967，Alan Kay提出了term“object oriented programming”的概念
OOP的强项(非常适合Large Scale软件开发)Abstraction(抽象)–对抗软件复杂性的有效工具

.OOP不擅长的地方. shared mutable statco
. Non-deterministic，not guaranteed to get the same output given the
same inputs.
·大量固定模式代码(signal-to-noise ratio, boilerplate code)

函数编程 
-  A mathematical function a relationship between two
sets(input & output sets). 
- lt is a mapping from one to the other. 
.Mathematical function - for any given input it always gives
you the same output.
 - Java 8的重要改进:
The stream APl and lambda expressions are the new features that move us closer tofunctional programming.



:::

软件工程实践-无所不在的Tradeoffs

:::info
软件工程实践的重要特征之一:    
limited time, knowledge, and resources force decisions on tradeoffs.

Productivity/Performance之间取舍
- 1 Java是statically typed语言，但允许:
Dynamic class(type) loading
.Dynamic method(code) modification(class/jar aroessentially data, not code)Metaprogramming: Reflection(like RTTl in C++)/Annotation
- 2 Java被认为是为“safe, managed runtime”而设计，但留了好多“backdoors”:  
  Java Native lnterface(JNI)  
 sun.misc.Unsafe

Java Runtime Tradeoff
垃圾回收器
"Infinite memory" & Stop-the-WorldJust-in-Time编译
Native performance speed & Time to warmup

:::
内存安全与Java 
:::info
典型的Memory Safety问题Buffer overflow
- 内存泄漏
-  Use memory that has been freed·使用未初始化的变量
Security/Safety是Java初始设计的动机之一，Java也被称为Memory-safe的语言
- Runtime error detection
(例如ArrayIlndexOutOfBoundsException)
- Pointer restrictions
:::

### Java工具  
  
JDK自带工具： 
:::info 
JDK自带的Diagnostic工具
Attach(Local Access)
jcmd - send diagnostic commands(since from JDK 7)- VM.version
- Thread.print
- Gc.class_histogram- GC.heap_dump
jstack - obtain Java and native stack informationjmap -generate a heap dump
 jinfo - gets configuration information from a running Java process

JDK自带的Diagnostic工具(2)
Remote Access
. JMX
- jmc, jsisualvm, jconsole
- Access information from java.lang.management Mbeans- Enable startup: -Dcom.sun.management.jmxremote- Enable runtime: jcmd ManagementAgent.start
jstatd
- Daemon that runs on remote machine- jstat can connect to daemon

:::


JFR简介：
:::info
JFR:构建JVM可观测性(Observability）基础
·初始是BEA's JRockit JVM的功能;
. Oracle 收购BEA, Jrockit 与HotSpot合并的结果，作为OracleJDK商用版本的重要功能;. openJDK 11开源;
Alibaba 与社区合作，将其贡献给OpenJDK8u;
. OpenJDK14引入JFR Event Streaming,构建监控以及可观测性的重要的JVM工具技术。
::: 


内存分析-Eclipse MAT&Jifa： 

升级工具一EMT4J：

 
