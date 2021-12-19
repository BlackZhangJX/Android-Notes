- [JVM](#jvm)
  - [JVM 工作流程](#jvm-工作流程)
  - [运行时数据区（Runtime Data Area）](#运行时数据区runtime-data-area)
    - [程序计数器](#程序计数器)
    - [Java 虚拟机栈](#java-虚拟机栈)
    - [本地方法栈](#本地方法栈)
    - [Java 堆](#java-堆)
    - [方法区](#方法区)
  - [方法指令](#方法指令)
  - [类加载器](#类加载器)
  - [垃圾回收 gc](#垃圾回收-gc)
    - [对象存活判断](#对象存活判断)
    - [垃圾收集算法](#垃圾收集算法)
    - [垃圾收集器](#垃圾收集器)
    - [内存模型与回收策略](#内存模型与回收策略)
- [Object](#object)
  - [equals 方法](#equals-方法)
  - [hashCode 方法](#hashcode-方法)
- [static](#static)
- [final](#final)
- [String、StringBuffer、StringBuilder](#stringstringbufferstringbuilder)
- [异常处理](#异常处理)
- [内部类](#内部类)
  - [匿名内部类](#匿名内部类)
- [多态](#多态)
- [抽象和接口](#抽象和接口)
- [集合框架](#集合框架)
  - [HashMap](#hashmap)
    - [结构图](#结构图)
    - [HashMap 的工作原理](#hashmap-的工作原理)
    - [HashMap 与 HashTable 对比](#hashmap-与-hashtable-对比)
  - [ConcurrentHashMap](#concurrenthashmap)
    - [Base 1.7](#base-17)
    - [Base 1.8](#base-18)
  - [ArrayList](#arraylist)
  - [LinkedList](#linkedlist)
  - [CopyOnWriteArrayList](#copyonwritearraylist)
- [反射](#反射)
- [单例](#单例)
  - [饿汉式](#饿汉式)
  - [双重检查模式](#双重检查模式)
  - [静态内部类模式](#静态内部类模式)
- [线程](#线程)
  - [属性](#属性)
  - [状态](#状态)
  - [状态控制](#状态控制)
- [volatile](#volatile)
- [synchronized](#synchronized)
  - [根据获取的锁分类](#根据获取的锁分类)
  - [原理](#原理)
- [Lock](#lock)
  - [锁的分类](#锁的分类)
    - [悲观锁、乐观锁](#悲观锁乐观锁)
    - [自旋锁、适应性自旋锁](#自旋锁适应性自旋锁)
    - [死锁](#死锁)
- [引用类型](#引用类型)
- [动态代理](#动态代理)
- [元注解](#元注解)
# JVM
## JVM 工作流程
![](https://user-gold-cdn.xitu.io/2019/6/23/16b833f4a4906226?w=448&h=592&f=jpeg&s=44057)

## 运行时数据区（Runtime Data Area）
![](https://user-gold-cdn.xitu.io/2019/6/23/16b833f4a499f6fe?w=868&h=497&f=webp&s=46378)

### 程序计数器
**程序计数器（Program Counter Register）** 是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。

字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

由于 Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，**在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）都只会执行一条线程中的指令**。

因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

- 如果线程正在执行的是一个 Java 方法，这个计数器记录的是正在执行的**虚拟机字节码指令的地址**。
- 如果线程正在执行的是一个 Native 方法，这个计数器值则为空（Undefined）。

**此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。**

### Java 虚拟机栈
**Java 虚拟机栈（Java Virtual Machine Stacks**）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是 Java 方法执行的内存模型，每个方法在执行的同时都会创建一个**栈帧（Stack Frame）** 用于存储**局部变量表、操作数栈、动态链接、方法出口**等消息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

**局部变量表**存放了编译器可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）和 returnAddress 类型（指向了一条字节码指令的地址）。

其中 64 位长度的 long 和 double 类型的数据会占用两个局部变量空间（Slot），其余的数据类型只占用一个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

在 Java 虚拟机规范中，对这个区域规定了两种异常状态：
- 如果线程请求的栈深度大于虚拟机所允许的的深度，将抛出 **StackOverflowError** 异常。
- 如果虚拟机栈可以动态扩展（当前大部分的Java虚拟机都可动态扩展，只不过Java虚拟机规范中也允许固定长度的虚拟机栈），如果扩展时无法申请到足够的内存，就会抛出 **OutOfMemoryError** 异常。

### 本地方法栈
**本地方法栈（Native Method Stack）** 与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。

在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（例如：Sun HotSpot虚拟机）直接就把虚拟机栈和本地方法栈合二为一。与虚拟机栈一样，本地方法栈区域也会抛出 StackOverflowError 和 OutOfMemoryError 异常。

### Java 堆
对于大多数应用来说，**Java 堆（Java Heap）** 是 Java 虚拟机所管理的的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。

Java堆是垃圾收集器管理的主要区域，从内存回收的角度来看，由于现在收集器基本采用分代收集算法，所以Java堆中还可以细分为：新生代和老年代；再细致一点的有 Eden 空间、From Survivor 空间、To Survivor 空间等。

从内存分配的角度来看，线程共享的Java堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB）。不过无论如何划分，都与存放内容无关，无论哪个区域，存储的仍然是对象实例，进一步划分的目的是为了更好地回收内存，或者更快地分配内存。

### 方法区
**方法区（Method Area**）与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

**运行时常量池（Runtime Constant Pool）** 是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是**常量池（Constant Pool Table）**，用于存放编译器生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时就会抛出 OutOfMemoryError 异常。

## 方法指令
| 指令 | 说明 |                     
|----------|-----|
| invokeinterface | 用以调用接口方法 |
| invokevirtual | 指令用于调用对象的实例方法 |
| invokestatic | 用以调用类/静态方法 |
| invokespecial | 用于调用一些需要特殊处理的实例方法，包括实例初始化方法、私有方法和父类方法 | 

## 类加载器
| 类加载器 | 说明 |                     
|----------|-----|
| BootstrapClassLoader | Bootstrap 类加载器负责加载 rt.jar 中的 JDK 类文件，它是所有类加载器的父加载器。Bootstrap 类加载器没有任何父类加载器，如果你调用 String.class.getClassLoader()，会返回 null，任何基于此的代码会抛出 NUllPointerException 异常。Bootstrap 加载器被称为初始类加载器 |
| ExtClassLoader | 而 Extension 将加载类的请求先委托给它的父加载器，也就是Bootstrap，如果没有成功加载的话，再从 jre/lib/ext 目录下或者 java.ext.dirs 系统属性定义的目录下加载类。Extension 加载器由 sun.misc.Launcher$ExtClassLoader 实现 |
| AppClassLoader | 第三种默认的加载器就是 System 类加载器（又叫作 Application 类加载器）了。它负责从 classpath 环境变量中加载某些应用相关的类，classpath 环境变量通常由 -classpath 或 -cp 命令行选项来定义，或者是 JAR 中的 Manifest 的 classpath 属性。Application 类加载器是 Extension 类加载器的子加载器 |

| 工作原理 | 说明 |                     
|----------|------|
| 委托机制 | 加载任务委托交给父类加载器，如果不行就向下传递委托任务，由其子类加载器加载，保证 java 核心库的安全性 |
| 可见性机制 | 子类加载器可以看到父类加载器加载的类，而反之则不行 |
| 单一性机制 | 父加载器加载过的类不能被子加载器加载第二次 |

## 垃圾回收 gc
### 对象存活判断
- **引用计数**

每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。 

- **可达性分析**

从 GC Roots 开始向下搜索，搜索所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是不可用的。不可达对象。

> 在Java语言中，GC Roots包括：
> - 虚拟机栈中引用的对象。
> - 方法区中类静态属性实体引用的对象。
> - 方法区中常量引用的对象。
> - 本地方法栈中 JNI 引用的对象。

### 垃圾收集算法
- **标记 -清除算法**
  
“标记-清除”（Mark-Sweep）算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其缺点进行改进而得到的。

它的主要缺点有两个：一个是效率问题，标记和清除过程的效率都不高；另外一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致，当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

- **复制算法**
  
“复制”（Copying）的收集算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

这样使得每次都是对其中的一块进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为原来的一半，持续复制长生存期的对象则导致效率降低。

- **标记-整理算法**
  
复制收集算法在对象存活率较高时就要执行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。

根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

- **分代收集算法**

GC 分代的基本假设：绝大部分对象的生命周期都非常短暂，存活时间短。

“分代收集”（Generational Collection）算法，把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记-清理”或“标记-整理”算法来进行回收。

### 垃圾收集器
- **CMS收集器**
  
> CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。目前很大一部分的 Java 应用都集中在互联网站或B/S系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。

从名字（包含“Mark Sweep”）上就可以看出CMS收集器是基于“标记-清除”算法实现的，它的运作过程相对于前面几种收集器来说要更复杂一些，整个过程分为4个步骤，包括：

- 初始标记（CMS initial mark）
- 并发标记（CMS concurrent mark）
- 重新标记（CMS remark）
- 并发清除（CMS concurrent sweep）

其中初始标记、重新标记这两个步骤仍然需要“Stop The World”。初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，并发标记阶段就是进行GC Roots Tracing的过程，而重新标记阶段则是为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。

由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以总体上来说，CMS收集器的内存回收过程是与用户线程一起并发地执行。老年代收集器（新生代使用ParNew）

- **G1收集器**
  
与CMS收集器相比G1收集器有以下特点：

1、空间整合，G1收集器采用标记整理算法，不会产生内存空间碎片。分配大对象时不会因为无法找到连续空间而提前触发下一次GC。

2、可预测停顿，这是G1的另一大优势，降低停顿时间是G1和CMS的共同关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为N毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒，这几乎已经是实时 Java（RTSJ）的垃圾收集器的特征了。

使用G1收集器时，Java堆的内存布局与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔阂了，它们都是一部分（可以不连续）Region 的集合。

G1的新生代收集跟 ParNew 类似，当新生代占用达到一定比例的时候，开始出发收集。和 CMS 类似，G1 收集器收集老年代对象会有短暂停顿。

### 内存模型与回收策略
![](https://mmbiz.qpic.cn/mmbiz_png/qdzZBE73hWsbhfAng9ibqfcbjrqgyRWqAKiaJ2U75SGYwQhs2tuNbXtu8KIpaUsBOaHRKXf7esuuFoMjELFxibIVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Java 堆（Java Heap）是JVM所管理的内存中最大的一块，堆又是垃圾收集器管理的主要区域，Java 堆主要分为2个区域-年轻代与老年代，其中年轻代又分 Eden 区和 Survivor 区，其中 Survivor 区又分 From 和 To 2个区。

- **Eden 区**
  
大多数情况下，对象会在新生代 Eden 区中进行分配，当 Eden 区没有足够空间进行分配时，虚拟机会发起一次 Minor GC，Minor GC 相比 Major GC 更频繁，回收速度也更快。
通过 Minor GC 之后，Eden 会被清空，Eden 区中绝大部分对象会被回收，而那些无需回收的存活对象，将会进到 Survivor 的 From 区（若 From 区不够，则直接进入 Old 区）。

- **Survivor 区**

Survivor 区相当于是 Eden 区和 Old 区的一个缓冲，类似于我们交通灯中的黄灯。Survivor 又分为2个区，一个是 From 区，一个是 To 区。每次执行 Minor GC，会将 Eden 区和 From 存活的对象放到 Survivor 的 To 区（如果 To 区不够，则直接进入 Old 区）。Survivor 的存在意义就是减少被送到老年代的对象，进而减少 Major GC 的发生。Survivor 的预筛选保证，只有经历16次 Minor GC 还能在新生代中存活的对象，才会被送到老年代。

- **Old 区**
  
老年代占据着2/3的堆内存空间，只有在 Major GC 的时候才会进行清理，每次 GC 都会触发“Stop-The-World”。内存越大，STW 的时间也越长，所以内存也不仅仅是越大就越好。由于复制算法在对象存活率较高的老年代会进行很多次的复制操作，效率很低，所以老年代这里采用的是标记——整理算法。


# Object
## equals 方法
对两个对象的地址值进行的比较（即比较引用是否相同）
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

## hashCode 方法
hashCode() 方法给对象返回一个 hash code 值。这个方法被用于 hash tables，例如 HashMap。

它的性质是：
- 在一个Java应用的执行期间，如果一个对象提供给 equals 做比较的信息没有被修改的话，该对象多次调用 hashCode() 方法，该方法必须始终如一返回同一个 integer。

- 如果两个对象根据 equals(Object) 方法是相等的，那么调用二者各自的 hashCode() 方法必须产生同一个 integer 结果。

- 并不要求根据 equals(Object) 方法不相等的两个对象，调用二者各自的 hashCode() 方法必须产生不同的 integer 结果。然而，程序员应该意识到对于不同的对象产生不同的 integer 结果，有可能会提高 hash table 的性能。

在 JDK 中，Object 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法直接返回对象的 内存地址。在 String 类，重写了 hashCode 方法
```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

# static
- static关键字修饰的方法或者变量不需要依赖于对象来进行访问，只要类被加载了，就可以通过类名去进行访问。
- 静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。
- 能通过 this 访问静态成员变量吗?
所有的静态方法和静态变量都可以通过对象访问（只要访问权限足够）。
- static是不允许用来修饰局部变量

# final
- 可以声明成员变量、方法、类以及本地变量
- final 成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误
- final 变量是只读的
- final 申明的方法不可以被子类的方法重写
- final 类通常功能是完整的，不能被继承
- final 变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销
- final 关键字提高了性能，JVM 和 Java 应用都会缓存 final 变量，会对方法、变量及类进行优化
- 方法的内部类访问方法中的局部变量，但必须用 final 修饰才能访问

# String、StringBuffer、StringBuilder
- String 是 final 类，不能被继承。对于已经存在的 Stirng 对象，修改它的值，就是重新创建一个对象
- StringBuffer 是一个类似于 String 的字符串缓冲区，使用 append() 方法修改 Stringbuffer 的值，使用 toString() 方法转换为字符串，是线程安全的
- StringBuilder 用来替代于 StringBuffer，StringBuilder 是非线程安全的，速度更快

# 异常处理
- Exception、Error 是 Throwable 类的子类
- Error 类对象由 Java 虚拟机生成并抛出，不可捕捉  
- 不管有没有异常，finally 中的代码都会执行
- 当 try、catch 中有 return 时，finally 中的代码依然会继续执行

| 常见的Error | | |
|------|-----|-----|
| OutOfMemoryError | StackOverflowError | NoClassDeffoundError |

| 常见的Exception | | |
|------|-----|-----|
| 常见的非检查性异常 |  |
| ArithmeticException | ArrayIndexOutOfBoundsException | ClassCastException |
| IllegalArgumentException | IndexOutOfBoundsException | NullPointerException |
| NumberFormatException | SecurityException | UnsupportedOperationException |
| 常见的检查性异常 |  |
| IOException | CloneNotSupportedException | IllegalAccessException |
| NoSuchFieldException | NoSuchMethodException | FileNotFoundException |

# 内部类
- 非静态内部类没法在外部类的静态方法中实例化。
- 非静态内部类的方法可以直接访问外部类的所有数据，包括私有的数据。
- 在静态内部类中调用外部类成员，成员也要求用 static 修饰。
- 创建静态内部类的对象可以直接通过外部类调用静态内部类的构造器；创建非静态的内部类的对象必须先创建外部类的对象，通过外部类的对象调用内部类的构造器。

## 匿名内部类
- 匿名内部类不能定义任何静态成员、方法
- 匿名内部类中的方法不能是抽象的
- 匿名内部类必须实现接口或抽象父类的所有抽象方法
- 匿名内部类不能定义构造器
- 匿名内部类访问的外部类成员变量或成员方法必须用 final 修饰

# 多态
- 父类的引用可以指向子类的对象
- 创建子类对象时，调用的方法为子类重写的方法或者继承的方法
- 如果我们在子类中编写一个独有的方法，此时就不能通过父类的引用创建的子类对象来调用该方法

# 抽象和接口
- 抽象类不能有对象（不能用 new 关键字来创建抽象类的对象）
- 抽象类中的抽象方法必须在子类中被重写
- 接口中的所有属性默认为：public static final ****；
- 接口中的所有方法默认为：public abstract ****；

# 集合框架
![](https://user-gold-cdn.xitu.io/2019/6/23/16b833f4a86db5e6?w=643&h=611&f=gif&s=22445)
- List接口存储一组不唯一，有序（插入顺序）的对象, Set接口存储一组唯一，无序的对象。

## HashMap
### 结构图
- **JDK 1.7 HashMap 结构图**

![](https://user-gold-cdn.xitu.io/2019/6/23/16b833f4ac8f44fd?w=1636&h=742&f=png&s=88323)

- **JDK 1.8 HashMap 结构图**

![](https://user-gold-cdn.xitu.io/2018/7/23/164c47f32f9650ba?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### HashMap 的工作原理
HashMap 基于 hashing 原理，我们通过 put() 和 get() 方法储存和获取对象。当我们将键值对传递给 put() 方法时，它调用键对象的 hashCode() 方法来计算 hashcode，让后找到 bucket 位置来储存 Entry 对象。当两个对象的 hashcode 相同时，它们的 bucket 位置相同，‘碰撞’会发生。因为 HashMap 使用链表存储对象，这个 Entry 会存储在链表中，当获取对象时，通过键对象的 equals() 方法找到正确的键值对，然后返回值对象。

**如果 HashMap 的大小超过了负载因子(load factor)定义的容量，怎么办？**  
默认的负载因子大小为 0.75，也就是说，当一个 map 填满了 75% 的 bucket 时候，和其它集合类(如 ArrayList 等)一样，将会创建原来 HashMap 大小的两倍的 bucket 数组，来重新调整 map 的大小，并将原来的对象放入新的 bucket 数组中。这个过程叫作 rehashing，因为它调用 hash 方法找到新的 bucket 位置。

**为什么 String, Interger 这样的 wrapper 类适合作为键?**  
因为 String 是不可变的，也是 final 的，而且已经重写了 equals() 和 hashCode() 方法了。其他的 wrapper 类也有这个特点。不可变性是必要的，因为为了要计算 hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的 hashcode 的话，那么就不能从 HashMap 中找到你想要的对象。不可变性还有其他的优点如线程安全。如果你可以仅仅通过将某个 field 声明成 final 就能保证 hashCode 是不变的，那么请这么做吧。因为获取对象的时候要用到 equals() 和 hashCode() 方法，那么键对象正确的重写这两个方法是非常重要的。如果两个不相等的对象返回不同的 hashcode 的话，那么碰撞的几率就会小些，这样就能提高 HashMap 的性能。

### HashMap 与 HashTable 对比
HashMap 是非 synchronized 的，性能更好，HashMap 可以接受为 null 的 key-value，而 Hashtable 是线程安全的，比 HashMap 要慢，不接受 null 的 key-value。

``HashMap.java``
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    ···

    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    ···

    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    ···
}
```

``HashTable.java``
```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    ···

    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        ···
        addEntry(hash, key, value, index);
        return null;
    }
    ···

    public synchronized V get(Object key) {
        HashtableEntry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (HashtableEntry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }
    ···
}
```

## ConcurrentHashMap
### Base 1.7

ConcurrentHashMap 最外层不是一个大的数组，而是一个 Segment 的数组。每个 Segment 包含一个与 HashMap 数据结构差不多的链表数组。

![](http://www.jasongj.com/img/java/concurrenthashmap/concurrenthashmap_java7.png)

在读写某个 Key 时，先取该 Key 的哈希值。并将哈希值的高 N 位对 Segment 个数取模从而得到该 Key 应该属于哪个Segment，接着如同操作 HashMap 一样操作这个 Segment。

Segment 继承自 ReentrantLock，可以很方便的对每一个 Segmen 上锁。

对于读操作，获取 Key 所在的 Segment 时，需要保证可见性。具体实现上可以使用volatile关键字，也可使用锁。但使用锁开销太大，而使用volatile时每次写操作都会让所有CPU内缓存无效，也有一定开销。ConcurrentHashMap 使用如下方法保证可见性，取得最新的Segment：
```java
Segment<K,V> s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)
```

获取 Segment 中的 HashEntry 时也使用了类似方法：
```java
HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
  (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE)
```

对于写操作，并不要求同时获取所有 Segment 的锁，因为那样相当于锁住了整个Map。它会先获取该 Key-Value 对所在的 Segment 的锁，获取成功后就可以像操作一个普通的 HashMap 一样操作该 Segment，并保证该 Segment 的安全性。同时由于其它 Segment 的锁并未被获取，因此理论上可支持 concurrencyLevel（等于Segment的个数）个线程安全的并发读写。

获取锁时，并不直接使用 lock 来获取，因为该方法获取锁失败时会挂起。事实上，它使用了自旋锁，如果 tryLock 获取锁失败，说明锁被其它线程占用，此时通过循环再次以 tryLock 的方式申请锁。如果在循环过程中该 Key 所对应的链表头被修改，则重置 retry 次数。如果 retry 次数超过一定值，则使用 lock 方法申请锁。

这里使用自旋锁是因为自旋锁的效率比较高，但是它消耗 CPU 资源比较多，因此在自旋次数超过阈值时切换为互斥锁。

### Base 1.8  
1.7 已经解决了并发问题，并且能支持 N 个 Segment 这么多次数的并发，但依然存在 HashMap 在 1.7 版本中的问题：查询遍历链表效率太低。因此 1.8 做了一些数据结构上的调整。

![](https://user-gold-cdn.xitu.io/2018/7/23/164c47f3756eb206?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

其中抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。

``ConcurrentHashMap.java``
```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        ···
                    }
                    else if (f instanceof TreeBin) {
                       ···
                    }
                    else if (f instanceof ReservationNode)
                        throw new IllegalStateException("Recursive update");
                }
            }
            ···
    }
    addCount(1L, binCount);
    return null;
}
```

## ArrayList
ArrayList 本质上是一个动态数组，第一次添加元素时，数组大小将变化为 DEFAULT_CAPACITY 10，不断添加元素后，会进行扩容。删除元素时，会按照位置关系把数组元素整体（复制）移动一遍。

``ArrayList.java``
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
    ···

    // 增加元素
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    ···

    // 删除元素
    public E remove(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        modCount++;
        E oldValue = (E) elementData[index];

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
    ···

    // 查找元素
    public E get(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        return (E) elementData[index];
    }
    ···
}
```

## LinkedList
LinkedList 本质上是一个双向链表的存储结构。

``LinkedList.java``
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    ····

    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    ···
    
    // 增加元素
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
    ···

    // 删除元素
    E unlink(Node<E> x) {
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
    ···

    // 查找元素
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
    ···
}
```

对于元素查询来说，ArrayList 优于 LinkedList，因为 LinkedList 要移动指针。对于新增和删除操作，LinedList 比较占优势，因为 ArrayList 要移动数据。

## CopyOnWriteArrayList
CopyOnWriteArrayList 是线程安全容器(相对于 ArrayList)，增加删除等写操作通过加锁的形式保证数据一致性，通过复制新集合的方式解决遍历迭代的问题。

``CopyOnWriteArrayList.java``
```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
 
    final transient Object lock = new Object();
    ···

    // 增加元素
    public boolean add(E e) {
        synchronized (lock) {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        }
    }
    ···

    // 删除元素
    public E remove(int index) {
        synchronized (lock) {
            Object[] elements = getArray();
            int len = elements.length;
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                setArray(newElements);
            }
            return oldValue;
        }
    }
    ···
    
    // 查找元素
    private E get(Object[] a, int index) {
        return (E) a[index];
    }
}
```

# 反射
```java
try {
    Class cls = Class.forName("com.jasonwu.Test");
    //获取构造方法
    Constructor[] publicConstructors = cls.getConstructors();
    //获取全部构造方法
    Constructor[] declaredConstructors = cls.getDeclaredConstructors();
    //获取公开方法
    Method[] methods = cls.getMethods();
    //获取全部方法
    Method[] declaredMethods = cls.getDeclaredMethods();
    //获取公开属性
    Field[] publicFields = cls.getFields();
    //获取全部属性
    Field[] declaredFields = cls.getDeclaredFields();
    Object clsObject = cls.newInstance();
    Method method = cls.getDeclaredMethod("getModule1Functionality");
    Object object = method.invoke(null);
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (InstantiationException e) {
    e.printStackTrace();
} catch (NoSuchMethodException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

# 单例
## 饿汉式
```java
public class CustomManager {
    private Context mContext;
    private static final Object mLock = new Object();
    private static CustomManager mInstance;

    public static CustomManager getInstance(Context context) {
        synchronized (mLock) {
            if (mInstance == null) {
                mInstance = new CustomManager(context);
            }

            return mInstance;
        }
    }

    private CustomManager(Context context) {
        this.mContext = context.getApplicationContext();
    }
}
```
## 双重检查模式
```java
public class CustomManager {
    private Context mContext;
    private volatile static CustomManager mInstance;

    public static CustomManager getInstance(Context context) {
        // 避免非必要加锁
        if (mInstance == null) {
            synchronized (CustomManger.class) {
                if (mInstance == null) {
                    mInstacne = new CustomManager(context);
                }
            }
        }

        return mInstacne;
    }

    private CustomManager(Context context) {
        this.mContext = context.getApplicationContext();
    }
}
```
## 静态内部类模式
```java
public class CustomManager{
    private CustomManager(){}
 
    private static class CustomManagerHolder {
        private static final CustomManager INSTANCE = new CustomManager();
    }
 
    public static CustomManager getInstance() {
        return CustomManagerHolder.INSTANCE;
    } 
}
```
静态内部类的原理是：

当 SingleTon 第一次被加载时，并不需要去加载 SingleTonHoler，只有当 getInstance() 方法第一次被调用时，才会去初始化 INSTANCE，这种方法不仅能确保线程安全，也能保证单例的唯一性，同时也延迟了单例的实例化。getInstance 方法并没有多次去 new 对象，取的都是同一个 INSTANCE 对象。

虚拟机会保证一个类的 ``<clinit>()`` 方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的 ``<clinit>()`` 方法，其他线程都需要阻塞等待，直到活动线程执行 ``<clinit>()`` 方法完毕

缺点在于无法传递参数，如Context等

# 线程
线程是进程中可独立执行的最小单位，也是 CPU 资源（时间片）分配的基本单位。同一个进程中的线程可以共享进程中的资源，如内存空间和文件句柄。

## 属性
| 属性 | 说明 |
|--|--|
| id | 线程 id 用于标识不同的线程。编号可能被后续创建的线程使用。编号是只读属性，不能修改 |
| name | 名字的默认值是 Thread-(id) |
| daemon | 分为守护线程和用户线程，我们可以通过 setDaemon(true) 把线程设置为守护线程。守护线程通常用于执行不重要的任务，比如监控其他线程的运行情况，GC 线程就是一个守护线程。setDaemon() 要在线程启动前设置，否则 JVM 会抛出非法线程状态异常，可被继承。 |
| priority | 线程调度器会根据这个值来决定优先运行哪个线程（不保证），优先级的取值范围为 1~10，默认值是 5，可被继承。Thread 中定义了下面三个优先级常量：<br>- 最低优先级：MIN_PRIORITY = 1<br>- 默认优先级：NORM_PRIORITY = 5<br>- 最高优先级：MAX_PRIORITY = 10 |

## 状态
![](https://pic2.zhimg.com/80/v2-326a2be9b86b1446d75b6f52f54c98fb_hd.jpg)

| 状态 | 说明 |
|--|--|
| New | 新创建了一个线程对象，但还没有调用start()方法。 |
| Runnable | Ready 状态 线程对象创建后，其他线程(比如 main 线程）调用了该对象的 start() 方法。该状态的线程位于可运行线程池中，等待被线程调度选中 获取 cpu 的使用权。Running 绪状态的线程在获得 CPU 时间片后变为运行中状态（running）。 |
| Blocked | 线程因为某种原因放弃了cpu 使用权（等待锁），暂时停止运行 |
| Waiting | 线程进入等待状态因为以下几个方法：<br>- Object#wait()<br>- Thread#join()<br>- LockSupport#park() |
| Timed Waiting | 有等待时间的等待状态。 |
| Terminated | 表示该线程已经执行完毕。 |

## 状态控制
- wait() / notify() / notifyAll()
  
``wait()``，``notify()``，``notifyAll()`` 是定义在Object类的实例方法，用于控制线程状态，三个方法都必须在synchronized 同步关键字所限定的作用域中调用，否则会报错 ``java.lang.IllegalMonitorStateException``。

| 方法 | 说明 |
|--|--|
| ``wait()`` | 线程状态由 的使用权。Running 变为 Waiting, 并将当前线程放入等待队列中 |
| ``notify()`` | notify() 方法是将等待队列中一个等待线程从等待队列移动到同步队列中 |
| ``notifyAll() `` | 则是将所有等待队列中的线程移动到同步队列中 |

被移动的线程状态由 Running 变为 Blocked，notifyAll 方法调用后，等待线程依旧不会从 wait() 返回,需要调用 notify() 或者 notifyAll() 的线程释放掉锁后，等待线程才有机会从 wait() 返回。

- join() / sleep() / yield()
  
在很多情况，主线程创建并启动子线程，如果子线程中需要进行大量的耗时计算，主线程往往早于子线程结束。这时，如果主线程想等待子线程执行结束之后再结束，比如子线程处理一个数据，主线程要取得这个数据，就要用 ``join()`` 方法。

``sleep(long)`` 方法在睡眠时不释放对象锁，而 ``join()`` 方法在等待的过程中释放对象锁。

``yield()`` 方法会临时暂停当前正在执行的线程，来让有同样优先级的正在等待的线程有机会执行。如果没有正在等待的线程，或者所有正在等待的线程的优先级都比较低，那么该线程会继续运行。执行了yield方法的线程什么时候会继续运行由线程调度器来决定。


# volatile
当把变量声明为 volatile 类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，JVM 保证了每次读变量都从内存中读，跳过 CPU cache 这一步，因此在读取 volatile 类型的变量时总会返回最新写入的值。

![](https://user-gold-cdn.xitu.io/2019/6/23/16b833f4a48b216e?w=550&h=429&f=png&s=21448)

当一个变量定义为 volatile 之后，将具备以下特性：
- 保证此变量对所有的线程的可见性，不能保证它具有原子性（可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的）
- 禁止指令重排序优化
- volatile 的读性能消耗与普通变量几乎相同，但是写操作稍慢，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行

AtomicInteger 中主要实现了整型的原子操作，防止并发情况下出现异常结果，其内部主要依靠 JDK 中的 unsafe 类操作内存中的数据来实现的。volatile 修饰符保证了 value 在内存中其他线程可以看到其值得改变。CAS（Compare and Swap）操作保证了 AtomicInteger 可以安全的修改value 的值。

# synchronized
当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。

在 Java 中，每个对象都会有一个 monitor 对象，这个对象其实就是 Java 对象的锁，通常会被称为“内置锁”或“对象锁”。类的对象可以有多个，所以每个对象有其独立的对象锁，互不干扰。针对每个类也有一个锁，可以称为“类锁”，类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。

Monitor 是线程私有的数据结构，每一个线程都有一个可用 monitor record 列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个 monitor 关联，同时 monitor 中有一个 Owner 字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。Monitor 是依赖于底层的操作系统的 Mutex Lock（互斥锁）来实现的线程同步。


## 根据获取的锁分类
**获取对象锁**
- synchronized(this|object) {}  
- 修饰非静态方法  

**获取类锁**
- synchronized(类.class) {}  
- 修饰静态方法

## 原理
**同步代码块：**
- monitorenter 和 monitorexit 指令实现的

**同步方法**
- 方法修饰符上的 ACC_SYNCHRONIZED 实现

# Lock
```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;  
    boolean tryLock();  
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;  
    void unlock();  
    Condition newCondition();
}
```
| 方法 | 说明 |
|--|--|
| ``lock()`` | 用来获取锁，如果锁被其他线程获取，处于等待状态。如果采用 Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一般来说，使用Lock必须在 try{}catch{} 块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。 |
| ``lockInterruptibly()`` | 通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。 |
| ``tryLock()`` | tryLock 方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回 true，如果获取失败（即锁已被其他线程获取），则返回 false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。 |
| ``tryLock(long，TimeUnit)`` | 与 tryLock 类似，只不过是有等待时间，在等待时间内获取到锁返回 true，超时返回 false。 |


## 锁的分类
![](https://user-gold-cdn.xitu.io/2019/6/18/16b69b50c9d340a5?w=1372&h=1206&f=png&s=142754)

### 悲观锁、乐观锁
悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。Java 中，synchronized 关键字和 Lock 的实现类都是悲观锁。悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确。

而乐观锁认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。乐观锁在 Java 中是通过使用无锁编程来实现，最常采用的是 CAS 算法，Java 原子类中的递增操作就通过 CAS 自旋实现。乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。

### 自旋锁、适应性自旋锁
阻塞或唤醒一个 Java 线程需要操作系统切换 CPU 状态来完成，这种状态转换需要耗费处理器时间。如果同步代码块中的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长。

在许多场景中，同步资源的锁定时间很短，为了这一小段时间去切换线程，线程挂起和恢复现场的花费可能会让系统得不偿失。如果物理机器有多个处理器，能够让两个或以上的线程同时并行执行，我们就可以让后面那个请求锁的线程不放弃CPU的执行时间，看看持有锁的线程是否很快就会释放锁。

而为了让当前线程“稍等一下”，我们需让当前线程进行自旋，如果在自旋完成后前面锁定同步资源的线程已经释放了锁，那么当前线程就可以不必阻塞而是直接获取同步资源，从而避免切换线程的开销。这就是自旋锁。

自旋锁本身是有缺点的，它不能代替阻塞。自旋等待虽然避免了线程切换的开销，但它要占用处理器时间。如果锁被占用的时间很短，自旋等待的效果就会非常好。反之，如果锁被占用的时间很长，那么自旋的线程只会白浪费处理器资源。所以，自旋等待的时间必须要有一定的限度，如果自旋超过了限定次数（默认是 10 次，可以使用 -XX:PreBlockSpin 来更改）没有成功获得锁，就应当挂起线程。

自旋锁的实现原理同样也是 CAS，AtomicInteger 中调用 unsafe 进行自增操作的源码中的 do-while 循环就是一个自旋操作，如果修改数值失败则通过循环来执行自旋，直至修改成功。

### 死锁
当前线程拥有其他线程需要的资源，当前线程等待其他线程已拥有的资源，都不放弃自己拥有的资源。

# 引用类型
强引用 > 软引用 > 弱引用 

| 引用类型 | 说明 |
|------|------|
| StrongReferenc（强引用）| 当一个对象具有强引用，那么垃圾回收器是绝对不会的回收和销毁它的，**非静态内部类会在其整个生命周期中持有对它外部类的强引用**|
| WeakReference （弱引用）| 在垃圾回收器运行的时候，如果对一个对象的所有引用都是弱引用的话，该对象会被回收 |
| SoftReference（软引用）| 如果一个对象只具有软引用，若内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，才会回收这些对象的内存|
| PhantomReference（虚引用） | 一个只被虚引用持有的对象可能会在任何时候被 GC 回收。虚引用对对象的生存周期完全没有影响，也无法通过虚引用来获取对象实例，仅仅能在对象被回收时，得到一个系统通知（只能通过是否被加入到 ReferenceQueue 来判断是否被GC，这也是唯一判断对象是否被 GC 的途径）。|

# 动态代理

示例：

```java
// 定义相关接口
public interface BaseInterface {
    void doSomething();
}

// 接口的相关实现类
public class BaseImpl implements BaseInterface {
    @Override
    public void doSomething() {
        System.out.println("doSomething");
    }
}

public static void main(String args[]) {
    BaseImpl base = new BaseImpl();
    // Proxy 动态代理实现
    BaseInterface proxyInstance = (BaseInterface) Proxy.newProxyInstance(base.getClass().getClassLoader(), base.getClass().getInterfaces(), new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            if (method.getName().equals("doSomething")) {
                method.invoke(base, args);
                System.out.println("do more");
            }
            return null;
        }
    });

    proxyInstance.doSomething();
}
```

``Proxy.java``
```java
public class Proxy implements java.io.Serializable {

    // 代理类的缓存
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
    ···

    // 生成代理对象方法入口
    public static Object newProxyInstance(ClassLoader loader,
                                        Class<?>[] interfaces,
                                        InvocationHandler h)
    throws IllegalArgumentException
    {
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        // 找到并生成相关的代理类
        Class<?> cl = getProxyClass0(loader, intfs);

        // 调用代理类的构造方法生成代理类实例
        try {
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                cons.setAccessible(true);
            }
            return cons.newInstance(new Object[]{h});
        } 
        ···
    }
    ···
    
    // 定义和返回代理类的工厂类
    private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
    {
        // 所有代理类的前缀
        private static final String proxyClassNamePrefix = "$Proxy";

        //  用于生成唯一代理类名称的下一个数字
        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

            Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
            ···

            String proxyPkg = null;     // 用于定义代理类的包名
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

            // 确保所有 non-public 的代理接口在相同的包里
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();
                if (!Modifier.isPublic(flags)) {
                    accessFlags = Modifier.FINAL;
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }

            if (proxyPkg == null) {
                // 如果没有 non-public 的代理接口，使用默认的包名
                proxyPkg = "";
            }

            {
                List<Method> methods = getMethods(interfaces);
                Collections.sort(methods, ORDER_BY_SIGNATURE_AND_SUBTYPE);
                validateReturnTypes(methods);
                List<Class<?>[]> exceptions = deduplicateAndGetExceptions(methods);

                Method[] methodsArray = methods.toArray(new Method[methods.size()]);
                Class<?>[][] exceptionsArray = exceptions.toArray(new Class<?>[exceptions.size()][]);

                // 生成代理类的名称
                long num = nextUniqueNumber.getAndIncrement();
                String proxyName = proxyPkg + proxyClassNamePrefix + num;

                // Android 特定修改：直接调用 native 方法生成代理类
                return generateProxy(proxyName, interfaces, loader, methodsArray,
                                     exceptionsArray);

                // JDK 使用的 ProxyGenerator.generateProxyClas 方法创建代理类
                byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                    proxyName, interfaces, accessFlags);
                try {
                    return defineClass0(loader, proxyName,
                                        proxyClassFile, 0, proxyClassFile.length);
                } ···
        }
    }
    ···

    // 最终调用 native 方法生成代理类
    @FastNative
    private static native Class<?> generateProxy(String name, Class<?>[] interfaces,
                                                 ClassLoader loader, Method[] methods,
                                                 Class<?>[][] exceptions);

}
```

``ProxyGenerator.java``
```java
public static byte[] generateProxyClass(final String name,
                                        Class[] interfaces)
{
    ProxyGenerator gen = new ProxyGenerator(name, interfaces);
    final byte[] classFile = gen.generateClassFile();

    if (saveGeneratedFiles) {
        java.security.AccessController.doPrivileged(
        new java.security.PrivilegedAction<Void>() {
            public Void run() {
                try {
                    FileOutputStream file =
                        new FileOutputStream(dotToSlash(name) + ".class");
                    file.write(classFile);
                    file.close();
                    return null;
                } catch (IOException e) {
                    throw new InternalError(
                        "I/O exception saving generated file: " + e);
                }
            }
        });
    }

    return classFile;
}
```

# 元注解
@Retention：保留的范围，可选值有三种。

| RetentionPolicy | 说明 |
|----|----|
| SOURCE | 注解将被编译器丢弃（该类型的注解信息只会保留在源码里，源码经过编译后，注解信息会被丢弃，不会保留在编译好的class文件里），如 @Override |
| CLASS | 注解在class文件中可用，但会被 VM 丢弃（该类型的注解信息会保留在源码里和 class 文件里，在执行的时候，不会加载到虚拟机中），请注意，当注解未定义 Retention 值时，默认值是 CLASS。 |
| RUNTIME | 注解信息将在运行期 (JVM) 也保留，因此可以通过反射机制读取注解的信息（源码、class 文件和执行的时候都有注解的信息），如 @Deprecated |

@Target：可以用来修饰哪些程序元素，如 TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER等，未标注则表示可修饰所有

@Inherited：是否可以被继承，默认为false  

@Documented：是否会保存到 Javadoc 文档中
