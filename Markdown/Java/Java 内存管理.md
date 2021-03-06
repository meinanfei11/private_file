---
title: Java 内存管理
tag: Java
---

对于 Java 程序员来说，在虚拟机的自动内存管理机制的帮助下，不再需要为每一个 new 操作区写配对的 `delete/free` 代码，而且不容易出现内存泄漏和内存移除问题，看起来一切由虚拟机管理内存一切都很美好。不过也正是 Java 程序员把内存控制的权力交给了 Java 虚拟机，一旦出现内存泄漏和溢出的问题，如果不了解续集及是怎么使用内存的，排查问题就很艰难。

## 运行时数据区域

![java runtime](https://github.com/xiaomanwong/static_file/blob/master/images/java_runtime_data_area.png?raw=true)

<!-- more -->

### 程序计数器

程序计数器是一块较小的内存空间，作用是当前线程锁执行的字节码的行号指示器。

由于 Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现，再任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）知乎执行一条线程中的指令，因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间的计数器互不影响，独立存储。

此区域是唯一一个 Java 虚拟机规范中没有任何 **OutOfMemoryError** 情况的区域

### Java 虚拟机栈

Java 虚拟机栈也是线程私有的，生命周期与线程相同。虚拟机描述的是 Java 方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作站、动态链接、方法处口等信息。

每一个方法被调用直到执行完成的过程，就对应一个栈帧再虚拟机栈中从入栈到出栈的过程。

**局部变量表** 存放了编译器可知的各种基本数据类型（boolean, byte, char, short, int, float, long, double），对象引用(Reference类型)。



**StackOverflowError**： 如果虚拟机栈可以动态扩展，当扩展时无法申请到足够的内存时会抛出 `StackOverflowError` 异常

### 本地方法栈

本地方法栈与虚拟机栈锁发挥的作用时相似的，其区别是虚拟机栈为虚拟机执行 Java 方法服务。而本地方法栈则是为虚拟机使用到的 Navite 方法服务。

### Java 堆

是 Java 虚拟机所管理的内存中最大的一块。Java 堆被所有线程共享的一块内存区域，再虚拟机启动的时候创建。**此区域唯一的目的就是存放对象实例**，几乎所有的对象实例都再这里分配内存。

Java 堆是垃圾回收器管理的主要区域，因此很多时候也被称做 GC 堆，从内存回收角度看，由于现在收集器基本都是采用 **分代收集算法** ，所以 Java 堆中还可以细分为： 新生代和老生代

如果从内存分配角度看，线程共享的 Java 堆中可能划分出多个线程私有的分配缓冲区

### **方法区**

方法区与 Java 堆一样，是哥哥线程共享的内存区域，**用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译的代码等数据**。

### **运行时常量池**

运行时常量池是方法去的一部分 ，用于存放编译器申城的各种字面量和符号引用，这部分内容在类加载后存放到方法区的运行时常量池中。

## 垃圾回收

**GC** 真正让程序员的生产力得到了释放，但是程序员很难感知到它的存在。在大多数情况下不是很需要关心 GC ，不过如果设计到一些性能优化，问题排查的时候，深入地了解 GC 还是有必要的。

### Java 内存区域

* 虚拟机栈：表述的是方法执行时的内存模型，线程私有化，生命周期和线程相同，每个方法被执行的同时都会创建栈帧，主要保存执行方法时的具不变量表、操作数栈、动态链接和方法返回地址等信息。方法执行时入栈，执行完成出栈，出栈就相当于清空了数据，入栈出栈的实际很明确，**这块区域不需要进行 GC**
* 本地方法栈：与虚拟机栈类似，主要在于虚拟机栈为虚拟机执行Java方法是服务，本地方法栈为虚拟机执行本地方法时服务。**不需要进行 GC**
* 程序计数器：线程独有，可以看作时当前线程执行的字节码行号。**不需要进行 GC**
* 本地内存：线程共享区域，本地内存；主要存储类的信息、长廊、静态变量、即使编译器编译后代码，这部分由于时在堆中实现的，受 GC 管理。Java 8 以后，这个区域也不需要GC
* 堆：对象实例和水族都是在堆上分配的， GC 也主要堆这两类数据进行回收

#### 回收算法

##### 引用计数法

最容易想到的一种方式，就是对象被引用一次，再它的头上就加一次引用次数，如果没有被引用（引用次数为0），则此对象可回收。但这种方式存在一个问题：**循环引用**

```java
public class Test {
    Test instance;
    public Test(String name){}
    
    public static void main(String[] args) {
        // first
        A a = new Test("a");
        B b = new Test("b");
        //second
        a.instance = b;
        b.instance = a;
	    //third
        a = null;
        b = null;
    }
}
```

按照上面的步骤，虽然 a, b 都被置为 null, 但是由于之前他们指向的对象相互引用（引用计数都为1），所以无法收回，也证是无法解决循环引用的问题，现代虚拟机一抛弃这种方法。

##### 可达性算法

以一系列叫做 **GC Root** 的对象为起点出发，引出他们指向的下一个节点，再以下个节点为起点，引出此节点的下一个节点。。。（通过 GC Root 传承的一条线就叫引用链），直到所有的节点都遍历完毕，如果相关对象不再任意一个以 GC Root 为起点的引用链上，则这个对象会被判定为垃圾，进行回收。

但是，一个对象的 `finalize` 方法给了对象一次垂死挣扎的机会，当对象不可达时，发生 GC 时，会先判断对象是否执行了 `finalize` 方法，如果未执行，则会先执行 `finalize` 方法，我们可以再此方法里将当前对象和 GC Root 关联，这样执行 `finalize` 之后，GC 会再次判断对象是否可达，如果不可达，就回收，可达则不回收。

**注意：** `finalize` 方法只会执行一次，如果第一次执行 `finalize` 方法，子对象变成了可达，确定不会回收，但如果对象再次被 GC 则会忽略 `finalize` 方法，对象会被回收。

**GC Root**

那么，什么样的对象可以作为 GC Root 呢

1. 虚拟机栈（栈帧中的本地变量）中的引用对象

2. 方法区中类静态属性引用的对象

3. 方法区中常量引用的对象

4. 本地方法栈中 JNI 引用的对象

**虚拟机栈中的对象**

```java
public class Test {
    public static void main(String[] args) {
        Test a = new Test();
        a = null;
    }
}
```

a 是栈帧中的本地变量，当 a = null 时，由于此时 a  充当了 GC Root 的作用， a 与原来指向的实例 `new Test()` 断开连接，所以对象会被回收。

**方法区中类静态属性引用的对象**

```java
public class Test {
    public static Test instance;
    public static void main(String[] args) {
        Test a = new Test();
        a.instance = new Test();
        a = null;
    }
}
```

当栈帧中的本地变量  `a = null` 时，由于 a 原来指向的对下个与 GC  Root（变量 instance）断开了连接，所以 a 原来的对象会被回收，而由于我们给 `instance` 赋值了变量的引用， `instance` 在此时是类静态属性引用，充当了 GC Root 的作用，它指向的对象依然存活。

**方法区中常量引用的对象**

```java
public class Test {
    public static final Test instance = new Test();
    public static void main(String[] args) {
        Test a = new Test();
        a = null;
    }
}
```

常量 `instance` 指向的对象并不会因为 a 指向的对象被回收而回收

**本地方法栈中的 JNI 引用的对象**

> 所谓本地方法就是一个 java 调用非 java 代码的接口，该方法并非 java 实现的，可能是 C 或 Python 等其他语言。Java 通过 JNI 来调用本地方法，而本地方法是以库文件的形式存放的。

当调用 Java 方法时，虚拟机会创建一个栈帧并压入 Java 栈，而当它调用的是本地方法时，虚拟机会保持 Java 不变，不会再 Java 栈中压入新的帧，虚拟机只是简单的动态连接并直接调用指定的本地方法。

##### 标记清除算法

**步骤**：

1. 先根据可达性算法 **标记** 出相应的可回收对象
2. 对可回收对象进行回收

操作起来很简单，也不需要做数据移动的操作。但是却存在一个问题 –> **内存碎片**

假如我们想在内存中分配一块需要连续内存占用的 4M  或 6M 的内存区域，由于内存碎片的存在，有可能得不到分配。

##### 复制算法

把堆等分成两块区域 A 和 B， 区域 A 负责分配对象， 区域 B 不非陪，对区域 A 使用标记清楚算法把存活的对象标记出来，然后把区域 A 中存活的对象都复制到 B 区域（同时将存活的对象都一次紧邻排列），最后把 A 区域对象全部清理掉释放出空间。

**问题：**

比如给堆分配了 500M 内存，结果只有 250M 可用 ，空间平白无故减少了一半。另外每此回收都要把存活的对象移动到另外一般，效率很低下。

##### 标记整理法

步骤：

1. 先根据可达性算法 **标记** 出相应的可回收对象
2. 对可回收对象进行回收
3. 将所有存活对象都往一端移动，紧邻排列。
