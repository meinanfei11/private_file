---
title: Java 中堆和栈的区别
tag: Java
---

## 区别

Java 中堆和栈的区别具体由一下几点



### 各司其职

**栈：** 用来存储局部变量和方法调用

**堆： ** 用来存储 Java 中的对象，无论是成员变量，局部变量，还是类，他们指向的对象都存储再堆内存中。

<!-- more -->

### 内存：

**栈：** 内存归属于单个线程，每个线程都会由一个栈内存，其存储的变量只能再其所属线程中可见，即占内存可以理解成线程的私有内存

**堆： ** 内存中的对象堆所有线程可见。堆内存中的对象可以被所有线程访问。

### 异常错误

如果栈内存没有可用空间存储方法调用和局部变量， JVM 会抛出 `java.lang.StackOverFlowError` 

如果堆内存没有可用空间存储 生成的对象，  JVM 会抛出 `Java.lang.OutOfMemoryError`

### 空间大小

栈的内存要远远小于堆内存，如果使用递归的话，那么栈很快就会充满。如果递归没有及时跳出，很可能发生 `StackOverFlowError` 问题

