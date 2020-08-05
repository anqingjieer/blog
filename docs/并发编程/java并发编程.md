# java并发编程

## 一.线层的基础

### 1.什么是线程？

![](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200514103426.png)																																***线程池的生命周期***





### 2.实现线程的三种方式

#### 继承Thread

```java
    public static void main(String[] args) {
        ThreadExtend threadExtend = new ThreadExtend();
        threadExtend.start();
    }
    @Override
    public void run() {
        System.out.println("我是真的爱上你");
    }
```



#### 实现Runnable

```java
public static void main(String[] args) {
    Thread a = new Thread(new ThreadRunable());
    a.start();
}

@Override
public void run() {
    System.out.println("我是真的爱上你");
}
```

#### 实现Callable

```java
@Override
public String  call() throws Exception {
    return "我是真的爱上你";
}
public static void main(String[] args) throws ExecutionException, InterruptedException {
    FutureTask futureTask=new FutureTask<>(new ThreadCallable());
    new Thread(futureTask).start();
    System.out.println(futureTask.get());
}

```

> 实现Runnable接口相比继承 Thread类有如下优势
> 	1）可以避免由于 Java的单继承特性而带来的局限
> 	2）增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的
> 	3）线程池只能放入实现 Runable或 Callable类线程，不能直接放入继承 Thread的类
> 实现Runnable接口和实现 Callable接口的区别
> 	1）Runnable是自从 java1.1就有了，而 Callable是 1.5之后才加上去的
> 	2）实现 Callable接口的任务线程能返回执行结果，而实现 Runnable接口的任务线程不能返回结果
>  	3 Callable接口的 call()方法允许抛出异 常，而 Runnable接口的 run()方法的异常只能在内部消化，不能继
> 续上抛
> 	 4）加入线程池运行 Runnable使用 ExecutorService的 execute方法， Callable使用 submit方法
> 注： Callable接口支持返回执行结果，此时需要调用 FutureTask.get()方法实现，此方法会阻塞主线程直到获
> 取返回结果，当不调用此方法时，主线程不会阻塞

#### 2.1 线层的启动和终止

 启动的话，thread.start（）方法进行启动的

终止的话，调用的是thread.interrupt（）方法，表示向当前线程打个招呼，告诉他可以中断线程的执行了 ，至于什么时候中断，取决于当前线程自己。线程通过检查资深是否被中断来进行相应，可以通过i sInterrupted() 来判断是否被中断。

#### 2.2 线程的安全性

对于线程安全性，本质上是管理数据对于数据状态的访问，而且这个状态是 同享的、可变的。

共享是这多个线程对一个资源进行访问。

可变指的是这个变量在生命周期内可以进行改变的。



一个线程是否安全，取决于他是否可以被多个线程访问，在不需额外的同步以及调用端不做任何改变的时候，这个共享状态是正确的。（意味着和我们想要的结果是一样的）

### 3.如何通过命令查看线程情况，

#### **jsp>jstack**

### 4.理解线程的上下文切换。

### 5.线程的执行顺序。

下面程序输出的是无序的，随机的。因为start方法只是让线程的状态是Runable，此时CPU没有进行分配内存。

```java
public static void main(String[] args) {
    Thread t = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("1");
        }
    });
    Thread t2 = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("2");
        }
    });
    Thread t3 = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("3");
        }
    });
    t.start();
    t2.start();
    t3.start();
}

```

加入join方法即可，其join方法的执行原理是如下图，内部调用wait

```java
millis=0
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            //如果线程还存活，进行等待0秒
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

![](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200514224703.png)



### 6.threadlocal

- threadlocal的实现原理

  里面维护的是一个CurrentMap去实现的，其中key为当前线程，valve为锁修饰的值

- threadlocal的缺点？

  使用threadlocal的话，使用完成之后一定要调用 threadlocal.remove

  因为其value是一个强引用，jvm不会去回收它，。如果不回收的话，会造成内存溢出。

## 二.并发线程基础

### 1.什么是多并发编程？

- 并发

	- 同一时间段内，多个任同时运行  。多个线程访问一个cpu.

- 并行

	- 单位时间内，多个任务同时运行。就是一个线程对应一个cpu。

### 2.java线程安全性问题？

- 3.java的内存模型

  ![](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200511153600.png)

- 2.指当多个线程同时读写一个共享资源并且没有任何同步措施时，导致出现脏数据或者其他不可预见的结果的问题

### 4.内存可见性问题

- 实际，多了一个了缓存和l2缓存。

  ![](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200511153630.png)

- 将所有的变量放在主内存中，当线程使用变量时，需要将主内存里面的变量复制到自己的工作内存，然后对自己的工作内存进行处理，处理完成之后更新到主内存

- 如果两个线程同时操作一个共享变量，可能就会出现这种问题

### 5.synchronized

- 线程的执行代码在进入synchronized 代码块前会自动获取内部锁，这时候其他线程访问该同步代码块时会被阻塞挂起

  重量级锁、重入锁、jvm级别锁    

  底层原理：monitorenter\monitorexit

  底层调用了JNI方法，调用c++的代码实现。

  在1.6之后引用了偏向锁和轻量级锁

  解析：里面是一个监视器
  
  1. 对于普通方法来说，作用给给当前实例进行加锁，进入代码前，要先获得锁
  2. 对于静态方法，锁的是当前类的Class
  3. 对于同步方法快，指定加锁对象，对给定对象加锁，进入同步代码库之前要获得给定对象的锁。

![监视器](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200516120221.png)

​		

- ![ssss](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200515205842.png)

### 6.volatile



![vol](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200515205916.png)

- 实现可见性的原理：是保证volatitle修饰的变量，自始至终一直保存在共享变量（主内存）中，工作内存为空，所以只能从主内存取数据

### 7.java的原子性操作

- 要不都执行，要不都不执行
- 使用synchronize可以保证线程的安全性，原理是：它是一个独占锁

### 8.CAS操作

- volatile 只能保证共享变量的可见性，不能解决读写一致性的原子性问题。

- Compa re and Swap 比较和交换。

  boolean compareAndSwapLong(Object obj ,long valueOffset,long expect, long update）
  里面有四个参数，当valueOsset达到expect的时候进行更新。则使用update的值替换旧的值expect。

- ABA问题 A-B-A这是一个环形操作，不能保证最后操纵的数据就是原先的A，可能中间有人将A变化B在变化为A

  假如线程I 使用CAS 修改初始值为A 的变量X ， 那么线程I 会首先去获取当前变量X 的值（为A 〕， 然后使用CAS 操作尝
  试修改X 的值为B ， 如果使用CAS 操作成功了， 那么程序运行一定是正确的吗？其实未必，这是因为有可能在线程I 获取变量X 的值A 后，在执行CAS 前，线程II 使用CAS 修改了变量X 的值为B ，然后又使用CAS 修改了变量X 的值为A 。所以虽然线程I 执行CAS时X 的值是A ， 但是这个A 己经不是线程I 获取时的A 了。
  
  这就是AB A 问题。

### 9.锁

- 乐观锁和悲观锁

	- 悲观锁

		- 悲观锁指对数据被外界修改持保守态度，认为数据很容易就会被其他线程修改，所以
在数据被处理前先对数据进行加锁

	- 乐观锁

		- 认为数据在一般情况下不会发生冲突，在提交的时候进行判断是否冲突.

- 公平锁和非公平锁

	- 按照线程请求锁的时间顺序来决定    ReentrantLock

- 独占锁和共享锁

  - synchronize和reentrantlocak都是独占锁，readwritelook，读写锁是共享锁，一个锁可以被多人共享。
  - 被单个线程持有还是能被多个线程共同持有

- 偏向锁

  -  前面说过，大部分情况下，锁不仅仅不存在多线程竞争，而是总是由同一个线程多次获得， 为了让线程获取锁的代价更低就引入了偏向锁的概念。 怎么理解偏向锁呢？
  - 当一个线程访问加了同步锁的代码块时，会在对象头中存储当前线程的 ID ，后续这个线程进入和退出这段加了同步锁的代码 块时，不需要再次加锁和释放锁。而是直接比较对象头里面是否存储了指向当前线程的偏向锁。如果相等表示偏向锁是偏向于当前线程的，就不需要再尝试获得锁了

- 轻量级锁

- 可重入锁

  - 就是一个线程可以多次访问这个锁，就是可重入锁。锁的线程再次获取锁时发现锁拥有者是自己，直接执行不用阻塞

    ```java
    private synchronized void helloA(){
        System.out.println("helloA");
    }
    private synchronized  void helloB(){
        helloA();
        System.out.println("helloB");
    }
    
    public static void main(String[] args) {
        ReentryThread r = new ReentryThread();
        r.helloB();
    }
    ```

- 自旋锁

  - 当访问独占锁的资源，线程不会立即堵塞，而是一直进行重复操作，等这个独占锁释放。

## 三、Thread Local的原理和bug

> 如果你创建了一个ThreadLocal 变量，那么访问这个变量的每个线程都会有这个变量的一个本地副本。当多
> 个线程操作这个变量时，实际操作的是自己本地内存里面的变量，从而避免了线程安全问题

**使用案例：**

```java
static void print(String string) {
        //打印当前缓存中threaLocal的值
        System.out.println(string+"===="+threadLocal.get());
        //清除当前变量的值
        // threadLocal.remove();
    }
    static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                threadLocal.set("thread");
                print("threadOne");
                System.out.println("one++++"+threadLocal.get());
            }
        });
        Thread threadTwo=new Thread(new Runnable() {
            @Override
            public void run() {
                //此时这个线程是访问不了线程一的值的，因为是单独存放
                System.out.println("two+++++"+threadLocal.get());
                threadLocal.set("threadTwo");
                print("two");
            }
        });
        threadOne.start();
        threadTwo.start();
    }
}


输出结果：
        threadOne====thread
        two+++++null
        one++++null
        two====threadTwo

```

源码：

```java
/**
* 可以看到底层是一个定制的threadMap,key为线程，value为当前值
*/
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}

// set先判断当前值存不存在，如果存在，直接set，不存在以当前线程为key,值为value构建ThreadMap
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

### 关于强引用、弱引用、软引用

1.    <u>强引用：普通的引用，强引用指向的对象不会被回收；</u>

   ```java
   String str="abc";
   // 只要某个对象与强引用关联，那么JVM在内存不足的情况下，宁愿抛出outOfMemoryError错误，也不会回收此类对象。
   //将str置为null，那么JVM就会在合适的时间回收此对象。
   ```

   

2.    <u>软引用：仅有软引用指向的对象，只有发生gc且内存不足，才会被回收；</u>

   ```java
   
   //如果内存足的话，就不回被回收
   public static void main(String args[]) {
           SoftReference<String> str = new SoftReference<String>(new String("abc"));
           System.out.println(str.get());
           //通知JVM进行内存回收
           System.gc();
           System.out.println(str.get());
   
   }
   输出结果：
       abc
       abc
   ```

   

3.    <u>弱引用：仅有弱引用指向的对象，只要发生gc就会被回收。</u>

   如果一个对象只具有弱引用，无论内存充足与否，Java GC后对象如果只有弱引用将会被自动回收。

   ```java
   Object obj = new Object();
   ReferenceQueue queue = new ReferenceQueue();
   WeakReference reference = new WeakReference(obj, queue);
    //强引用对象滞空，保留软引用
    obj = null;
   ```

   

4.    虚引用

- [ ] [*![图片fff](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200512104205.png)*]()

> 照片来源：https://blog.csdn.net/l540675759/article/details/73733763

| 引用类型 | GC或者JVM充足 |   不足   |
| :------: | :-----------: | :------: |
|  强引用  |   不被回收    | 不被回收 |
|  弱引用  |    被回收     |  被回收  |
|  软引用  |   不被回收    |  被回收  |

> key是线程是弱引用，value是强引用，每次set,get,会将这个之前的线程进行回收，但是不将value回收，这样就回出现内存溢出
> 1.使用ThreadLocal，建议用static修饰 static ThreadLocal<HttpHeader> headerLocal = new ThreadLocal();
> 2.使用完ThreadLocal后，执行**<u>*remove*</u>**操作，避免出现内存溢出情况。

## 四、线程池的使用

### 1.java有哪几种线程池。

1. 普通线程池  ThreadPoolExecutor
2. 定时线程池  ScheduledThreadPoolExecutor

### 2线程池的继承关系

![](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200514225255.png)

### 3. 线程池执行原理

执行类图

![类图](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200514225808.png)

构造器

corePoolSize ：池中所保存的线程数，包括空闲线程
• maximumPoolSize：池中允许的最大线程数
• keepAliveTime        当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间
• unit keepAliveTime     参数的时间单位
• workQueue ：   执行前用于保持任务的队列。此队列仅保持由 execute方法提交的 Runnable任务
• threadFactory：   执行程序创建新线程时使用的工厂
• handler ：       由于超出线程范围和队列容量而使执行

核心数量，任务队列容器，存活时间，线程工厂，处理器。

如果当前池大小poolSize 小于 corePoolSize ，则创建新线程执行任务
如果当前池大小poolSize 大于 corePoolSize ，且等待队列未满，则进入等待队列
如果当前池大小poolSize 大于 corePoolSize 且小于 maximumPoolSize ，且等待队列已满，则创建新线程
执行任务
如果当前池大小poolSize 大于 corePoolSize 且大于 maximumPoolSize ，且等待队列已满，则调用拒绝策
略来处理该任务

```java
//固定线程池
ExecutorService executorService = Executors.newFixedThreadPool(2);
executorService.execute(new Runnable() {
      public void run() {

    }
});//runnable接口
executorService.submit(new Callable<String>() {
    public String call() throws Exception {
        return "abc";
    }
});//callable接口

```

初始化构造器

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

java.util.concurrent.ThreadPoolExecutor#execute

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    //判断是否小于核心数量，是直接新增work成功后直接退出 
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();// 增加失败后继续获取标记
    }
    //判断是运行状态并且扔到workQueue里成功后
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
    //再次check判断运行状态如果是非运行状态就移除出去&reject掉
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0) //否则发现可能运行线程数是0那么增加一个null的worker。
            addWorker(null, false);
    }
    else if (!addWorker(command, false)) //直接增加worker如果不成功直接reject
        reject(command);
	}
}
```

java.util.concurrent.ThreadPoolExecutor#addWorker



```java
retry:
for (;;) {
    int c = ctl.get();
    int rs = runStateOf(c);

    // Check if queue empty only if necessary. 
    if (rs >= SHUTDOWN &&
        ! (rs == SHUTDOWN &&
           firstTask == null &&
           ! workQueue.isEmpty()))
        return false;// 两种情况1.如果非运行状态  2.不是这种情况（停止状态并且是null对象并且workQueue不等于null）

    for (;;) {
        int wc = workerCountOf(c);
        if (wc >= CAPACITY ||
            wc >= (core ? corePoolSize : maximumPoolSize))
            return false;// 判断是否饱和容量了
        if (compareAndIncrementWorkerCount(c)) //增加一个work数量 然后跳出去
            break retry;
        c = ctl.get();  // Re-read ctl  增加work失败后继续递归
        if (runStateOf(c) != rs)
            continue retry;
        // else CAS failed due to workerCount change; retry inner loop
    }
}

boolean workerStarted = false;
boolean workerAdded = false;
Worker w = null;
try {
    w = new Worker(firstTask);//增加一个worker
    final Thread t = w.thread;
    if (t != null) {//判断是否 为null
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            // Recheck while holding lock.
            // Back out on ThreadFactory failure or if
            // shut down before lock acquired.  锁定后并重新检查下 是否存在线程工厂的失败或者锁定前的关闭
            int rs = runStateOf(ctl.get());

            if (rs < SHUTDOWN ||
                (rs == SHUTDOWN && firstTask == null)) {
                if (t.isAlive()) // precheck that t is startable
                    throw new IllegalThreadStateException();  
                workers.add(w);   //增加work
                int s = workers.size();
                if (s > largestPoolSize)
                    largestPoolSize = s;
                workerAdded = true;
            }
        } finally {
            mainLock.unlock();
        }
        if (workerAdded) { //本次要是新增加work成功就调用start运行
            t.start();
            workerStarted = true;
        }
    }
} finally {
    if (! workerStarted)
        addWorkerFailed(w);
}
return workerStarted;




Thread wt = Thread.currentThread();//1.取到当前线程
Runnable task = w.firstTask;
w.firstTask = null;
w.unlock(); // allow interrupts
boolean completedAbruptly = true;
try {
    while (task != null || (task = getTask()) != null) { //获取任务 看看是否能拿到
        w.lock();
        // If pool is stopping, ensure thread is interrupted;
        // if not, ensure thread is not interrupted.  This
        // requires a recheck in second case to deal with
        // shutdownNow race while clearing interrupt
        if ((runStateAtLeast(ctl.get(), STOP) ||
             (Thread.interrupted() &&
              runStateAtLeast(ctl.get(), STOP))) &&
            !wt.isInterrupted())
            wt.interrupt();// 确保线程是能中断的
        try {
            beforeExecute(wt, task); //开始任务前的钩子
            Throwable thrown = null;
            try {
                task.run();//执行任务
            } catch (RuntimeException x) {
                thrown = x; throw x;
            } catch (Error x) {
                thrown = x; throw x;
            } catch (Throwable x) {
                thrown = x; throw new Error(x);
            } finally {
                afterExecute(task, thrown); //任务后的钩子
            }
        } finally {
            task = null;
            w.completedTasks++;
            w.unlock();
        }
    }
    completedAbruptly = false;
} finally {
    processWorkerExit(w, completedAbruptly);
}

```

java.util.concurrent.ThreadPoolExecutor#runWorker

 

java.util.concurrent.ThreadPoolExecutor#processWorkerExit



```java
if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
    decrementWorkerCount();

final ReentrantLock mainLock = this.mainLock;
mainLock.lock();
try {
    completedTaskCount += w.completedTasks;
    workers.remove(w);  //移除work
} finally {
    mainLock.unlock();
}

tryTerminate();

int c = ctl.get();
if (runStateLessThan(c, STOP)) { //判断是否还有任务
    if (!completedAbruptly) {
        int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
        if (min == 0 && ! workQueue.isEmpty())
            min = 1;
        if (workerCountOf(c) >= min)
            return; // replacement not needed
    }
    addWorker(null, false);
}
```

#### 4.调度线程池原理

## 五、常见的锁类

1.关于ReentrantLock

![去呗](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200516120720.png)

> Synchronized 和ReentrantLock对比：
> Synchronized jvm层级的锁 自动加锁自动释放锁
> Lock：依赖特殊的 cpu指令，代码实现、手动加锁和释放锁、 Condition（生产消费模式
> ReentrantReadWriteLock 读写锁
> 细粒度问题
> ，读是共享的、写是独占的
> java8增加了对读写锁的优化： StampedLock(读是乐观、写是悲观)