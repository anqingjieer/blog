## 1.什么是jvm虚拟机？

​       以软件的方式模拟具有完整硬件系统功能、运行在一个完全隔离环境中的完整计算机系统，是物理机的软件实现。

​       凡是能够编译成class文件的都可以在jvm里面进行编译。

## 2.编译过程

代码：

```java
class Person{
    private String name;
    private int age;
    private static String address;
    private final static String hobby="Programming";
    public void say(){
        System.out.println("person say...");
    }
	public int calc(int op1,int op2){
		return op1+op2;
	}
}
```



> Person.java -> 词法分析器 -> tokens流 -> 语法分析器 -> 语法树/抽象语法树 -> 语义分析器
> -> 注解抽象语法树 -> 字节码生成器 -> Person.class文件

![image-20200619140438120](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619140438.png)

## 2.类文件加载到虚拟机？

> 类加载器将class文件加载到虚拟机的内存



### 2.1装载（load）

> (1)通过一个类的全限定名获取定义此类的二进制字节流
> (2)将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
> (3)在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口

### 2.2链接

#### 2.2.1 验证

保证被加载类的正确性

- 文件格式验证
- 原数据验证
- 字节码验证
- 符号引用验证

#### 2.2.2 准备

为类的静态变量分配内存，并将其初始化为默认值

#### 2.2.3解析

把类中的符号引用转换为直接引用

### 2.3 初始化

对类的静态变量，静态代码块执行初始化操作

![image-20200619140759708](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619140759.png)

## 4.类加载器

在装载(Load)阶段，其中第(1)步:通过类的全限定名获取其定义的二进制字节流，需要借助类装载
器完成，顾名思义，就是用来装载Class文件的。
(1)通过一个类的全限定名获取定义此类的二进制字节流

### 4.1类加载器进行分类

1）Bootstrap ClassLoader 负责加载$JAVA_HOME中 jre/lib/rt.jar 里所有的class或
Xbootclassoath选项指定的jar包。由C++实现，不是ClassLoader子类。
2）Extension ClassLoader 负责加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中
jre/lib/*.jar 或 -Djava.ext.dirs指定目录下的jar包。
3）App ClassLoader 负责加载classpath中指定的jar包及 Djava.class.path 所指定目录下的类和
jar包。
4）Custom ClassLoader 通过java.lang.ClassLoader的子类自定义加载class，属于应用程序根据
自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader。

### 4.2图解

![image-20200707210830265](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200707210830.png)

双亲委派机制

如果一个类加载器在接到加载类的请求时，它首先不会自己尝试去加载这个类，而是把这个请求任务委托给父类加载器去完成，

依次递归，如果父类加载器可以完成类加载任务，就成功返回；

只有父类加载器无法完成此加载任务时，才自己去加载。

优势：Java类随着加载它的类加载器一起具备了一种带有优先级的层次关系。比如，Java中的Object类，它存放在rt.jar之中,无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的启动类加载器进行加载，因此Object在各种类加载环境中都是同一个类。如果不采用双亲委派模型，那么由各个类加载器自己取加载的话，那么系统中会存在多种不同的Object类。

破坏：可以继承ClassLoader类，然后重写其中的loadClass方法，其他方式大家可以自己了解拓展一下。

## 5.jvm内存划分

### 5.1 图解

![image-20200619142656469](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619142656.png)

### 5.2各个区域的理解

- ### 方法区

  方法区是各个线程共享的内存区域，在虚拟机启动时创建。**用于存储已被虚拟机加载的类信息**、**常量**、**静态变量**、**即时编译器编译后的代码等数据**。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却又一个别名叫做Non-Heap(非堆)，目的是与Java堆区分开来。

  当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

  ![image-20200619143126423](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619143126.png)

  > (1)方法区在JDK 8中就是Metaspace，在JDK6或7中就是Perm Space
  > (2)Run-Time Constant Pool

- ### 堆

  Java堆是Java虚拟机所管理内存中最大的一块，在虚拟机启动时创建，被所有线程共享。
  Java对象实例以及数组都在堆上分配。

  ![image-20200619144550730](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619144550.png)

- ### 程序计数器

- ### 本地方法栈和虚拟机栈

  1.虚拟机栈是一个线程执行的区域，保存着一个线程中方法的调用状态。换句话说，一个Java线程的运行状态，由一个虚拟机栈来保存，所以虚拟机栈肯定是线程私有的，独有的，随着线程的创建而创建。

  2.每一个被线程执行的方法，为该栈中的栈帧，即每个方法对应一个栈帧。
  3.调用一个方法，就会向栈中压入一个栈帧；一个方法调用完成，就会把该栈帧从栈中弹出。

![image-20200619150501544](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619150501.png)

### 5.3结合字节码指令理解Java虚拟机栈和栈针

栈针：每一个栈针对应一个被调用的方法，可以理解为一个方法的运行空间

- 局部变量表:方法中定义的局部变量以及方法的参数存放在这张表中。局部变量表中的变量不可直接使用，如需要使用的话，必须通过相关指令将其加载至操作数栈中作为操作数使用。

- 操作数栈:以压栈和出栈的方式存储操作数的
- 动态链接:每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接(Dynamic Linking)。
- 方法返回地址：遇到方法方法返回的指令；一种是遇见异常，并且这个方法没有再方法体内得到处理。

### 5.4内存模型

> 一块是非堆区，一块是堆区。
> 堆区分为两大块，一个是Old区，一个是Young区。
> Young区分为两大块，一个是Survivor区（S0+S1），一块是Eden区。 Eden:S0:S1=8:1:1
> S0和S1一样大，也可以叫From和To。

![image-20200619151937368](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619151937.png)

#### 5.4.1对象创建所在区域

一般情况下，新创建的对象都会被分配到Eden区，一些特殊的大的对象会直接分配到Old区。

比如有对象A，B，C等创建在Eden区，但是Eden区的内存空间肯定有限，比如有100M，假如已经使用了
100M或者达到一个设定的临界值，这时候就需要对Eden内存空间进行清理，即垃圾收集(Garbage Collect)，
这样的GC我们称之为Minor GC，Minor GC指得是Young区的GC。

经过GC之后，有些对象就会被清理掉，有些对象可能还存活着，对于存活着的对象需要将其复制到Survivor
区，然后再清空Eden区中的这些对象。



survivor区详解

在同一个时间点上，S0和S1只能有一个区有数据，另外一个是空的。


> 接着上面的GC来说，比如一开始只有Eden区和From中有对象，To 中是空的。
> 此时进行一次GC操作，From区中对象的年龄就会+1，我们知道Eden区中所有存活的对象会被复制到To 区，
> From区中还能存活的对象会有两个去处。
>
> 1.若对象年龄达到之前设置好的年龄阈值，此时对象会被移动到Old区，
>
> 2.如果Eden区和From区没有达到阈值的对象会被复制到To 区。此时Eden区和From区已经被清空(被GC的对象肯定没了，没有被GC的对象都有了各自的去处)。



old区详解

从上面的分析可以看出，一般Old区都是年龄比较大的对象，或者相对超过了某个阈值的对象。在Old区也会有GC的操作，Old区的GC我们称作为Major GC。



对象的一辈子：

我是一个普通的Java对象,我出生在Eden区,在Eden区我还看到和我长的很像的小兄弟,我们在Eden区中玩了挺长时间。有一天Eden区中的人实在是太多了,我就被迫去了Survivor区的“From”区,自从去了Survivor区,我就开始漂了,有时候在Survivor的“From”区,有时候在Survivor的“To”区,居无定所。直到我18岁的时候,爸爸说我成人了,该去社会上闯闯了。于是我就去了年老代那边,年老代里,人很多,并且年龄都挺大的,我在这里也认识了很多人。在年老代里,我生活了20年(每次GC加一岁),然后被回收。

![image-20200619153108362](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619153108.png)

### 5.5常见的问题？

####       1.如何理解Minor/Major/Full GC？

> ​        Minor GC:新生代
> ​		Major GC:老年代
> ​		Full GC:新生代+老年代

####      2 .为什么需要Survivor区?只有Eden不行吗？

如果没有Survivor,Eden区每进行一次Minor GC,并且没有年龄限制的话，存活的对象就会被送到老年代。这样一来，老年代很快被填满,触发Major GC(因为Major GC一般伴随着Minor GC,也可以看做触发了Full GC)。老年代的内存空间远大于新生代,进行一次Full GC消耗的时间比Minor GC长得多。执行时间长有什么坏处?频发的Full GC消耗的时间很长,会影响大型程序的执行和响应速度。

所以Survivor的存在意义,就是减少被送到老年代的对象,进而减少Full GC的发生,Survivor的预筛选保证,只有经历16
次Minor GC还能在新生代中存活的对象,才会被送到老年代。

####       3.为什么需要两个Survivor?

最大的好处就是解决了碎片化。

​       刚刚新建的对象在Eden中,一旦Eden满了,触发一次Minor GC,Eden中的存活对象就会被移动到Survivor区。这样继续循环下去,下一次Eden满了的时候,问题来了,此时进行Minor GC,Eden和Survivor各有一些存活对象,如果此时把Eden区的存活对象硬放到Survivor区,很明显这两部分对象所占有的内存是不连续的,也就导致了内存碎片化。
永远有一个Survivor space是空的,另一个非空的Survivor space无碎片。

新生代中Eden:S1:S2为什么是8:1:1？

新生代中的可用内存：复制算法用来担保的内存为9：1
可用内存中Eden：S1区为8：1
即新生代中Eden:S1:S2 = 8：1：1



Stack Space用来做方法的递归调用时压入Stack Frame(栈帧)。所以当递归调用太深的时候，就有可能耗尽Stack
Space，爆出StackOverflow的错误。



线程栈的大小是个双刃剑，如果设置过小，可能会出现栈溢出，特别是在该线程内有递归、大的循环时出现溢出的可能性更大，如果该值设置过大，就有影响到创建栈的数量，如果是多线程的应用，就会出现内存溢出的错误。

## 6.GC(垃圾回收)

### 1.如何确定一个对象是垃圾

### 1.1引用计数法

​          对于某个对象而言，只要应用程序中持有该对象的引用，就说明该对象不是垃圾，如果一个对象没有任
何指针对其引用，它就是垃圾。

如果AB相互持有引用，导致永远不能被回收。

### 1.2可达性分析

通过GC Root的对象，开始向下寻找，看某个对象是否可达

> 能作为GC Root:类加载器、Thread、虚拟机栈的本地变量表、static成员、常量引用、本地方法
> 栈的变量等。

### 2.垃圾回收算法

1. 标记清除   会产生空间碎片

2. 复制算法  空间利用率降低。

3. 标记整理算法

4. 分代回收算法

   Young: 复制算法

   old区：标记清除或标记整理

   ![image-20200619161213169](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619161213.png)

常见的垃圾收集器

#### Seriallh收集器

它是一种单线程收集器，不仅仅意味着它只会使用一个CPU或者一条收集线程去完成垃圾收集工作，更
重要的是其在进行垃圾收集的时候需要暂停其他线程。

> 优点：简单高效，拥有很高的单线程收集效率
> 缺点：收集过程需要暂停所有线程
> 算法：复制算法
> 适用范围：新生代

![image-20200619161259493](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619161259.png)

#### ParNew收集器

可以理解为Seriall收集器

优点：在多CPU时，比serial效率高

![image-20200619161515925](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619161515.png)

#### Parallel Scavenge收集器

Parallel Scavenge收集器是一个新生代收集器，它也是使用复制算法的收集器，又是并行的多线程收集
器，看上去和ParNew一样，但是Parallel Scanvenge更关注系统的吞吐量。

#### CMS收集器

![image-20200619161628221](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619161628.png)

优点：并发收集、低停顿
缺点：产生大量空间碎片、并发阶段会降低吞吐量

#### G1收集器

并行与并发
分代收集（仍然保留了分代的概念）
空间整合（整体上属于“标记-整理”算法，不会导致空间碎片）
**可预测的停顿（比CMS更先进的地方在于能让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集**
**上的时间不得超过N毫秒）**

> 初始标记（Initial Marking） 标记一下GC Roots能够关联的对象，并且修改TAMS的值，需要暂停用户线程
> 并发标记（Concurrent Marking） 从GC Roots进行可达性分析，找出存活的对象，与用户线程并发执行
> 最终标记（Final Marking） 修正在并发标记阶段因为用户程序的并发执行导致变动的数据，需暂停用户线程
> 筛选回收（Live Data Counting and Evacuation） 对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间制定回收计划

![image-20200619161818651](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200619161818.png)

### 3.垃圾回收器

- 串行收集器->Serial和Serial Old

- 并行收集器【吞吐量优先]】Parallel Scanvenge、Parallel Old

- 并发收集器[停顿时间优先]->CMS、G1 

  用户线程和垃圾收集线程同时执行(但并不一定是并行的，可能是交替执行的)，垃圾收集线程在执行的
  时候不会停顿用户线程的运行

### 4.理解吞吐量和停顿时间

- 停顿时间->垃圾收集器 进行 垃圾回收终端应用执行响应的时间
- 吞吐量->运行用户代码时间/(运行用户代码时间+垃圾收集时间)

### 5.如何开启需要的垃圾回收器

> （1）串行
> -XX：+UseSerialGC
> -XX：+UseSerialOldGC
> （2）并行(吞吐量优先)：
> -XX：+UseParallelGC
> -XX：+UseParallelOldGC
> （3）并发收集器(响应时间优先)
> -XX：+UseConcMarkSweepGC
> -XX：+UseG1GC

