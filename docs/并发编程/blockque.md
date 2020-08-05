# 阻塞队列

## 1.阻塞队列的基本概念。

​    阻塞队列是一种队列，一种可以在多环境下进行使用的，并支持阻塞等待的队列。阻塞队列和一般的队列的区别是。

- 多线程环境支持，多个线程进行安全访问队列
- 支持生产和消费等待，多个线程之间相互配合，当队列为空的时候，消费线程会阻塞等待队列不为空；当队列满的时候，生产线程阻塞直到队列不满。





Condition

假设缓存队列中已经存满，那么阻塞的肯定是写线程，唤醒的肯定是读线程

相反，阻塞的肯定是读线程，唤醒的肯定是写线程，

假设只有一个Condition会有什么效果呢，缓存队列中已经存满，这个Lock不知道唤醒的是读线程还是写线程了，如果唤醒的是读线程，皆大欢喜，

> Condition它更强大的地方在于：能够更加精细的控制多线程的休眠与唤醒。对于同一个锁，我们可以创建多个Condition，在不同的情况下使用不同的Condition。

```java
package com.turingschool.demo.blockingqueue.diy;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;


/**
 * Condition的强大之处，假设缓存队列中已经存满，那么阻塞的肯定是写线程，唤醒的肯定是读线程，相反，阻塞的肯定是读线程，唤醒的肯定是写线程，
	 * 那么假设只有一个Condition会有什么效果呢，缓存队列中已经存满，这个Lock不知道唤醒的是读线程还是写线程了，如果唤醒的是读线程，皆大欢喜，
 * 如果唤醒的是写线程，那么线程刚被唤醒，又被阻塞了，这时又去唤醒，这样就浪费了很多时间。
 *
 *
 * Condition它更强大的地方在于：能够更加精细的控制多线程的休眠与唤醒。对于同一个锁，我们可以创建多个Condition，在不同的情况下使用不同的Condition。
 *
 *
 * 例如，假如多线程读/写同一个缓冲区：当向缓冲区中写入数据之后，唤醒"读线程"；当从缓冲区读出数据之后，唤醒"写线程"；并且当缓冲区满的时候，
 * "写线程"需要等待；当缓冲区为空时，"读线程"需要等待。
 * 如果采用Object类中的wait(),notify(),notifyAll()实现该缓冲区，当向缓冲区写入数据之后需要唤醒"读线程"时，不可能通过notify()或notifyAll()
 * 明确的指定唤醒"读线程"，而只能通过notifyAll唤醒所有线程(但是notifyAll无法区分唤醒的线程是读线程，还是写线程)。 但是，通过Condition，
 * 就能明确的指定唤醒读线程。
 * @param <E>
 */
public class ArrayBlockingQueue<E> {

	final Object[] items;

	int takeIndex;

	int putIndex;

	int count;

	final ReentrantLock lock;

	private final Condition notEmpty;

	private final Condition notFull;

	/**
	 * Returns item at index i.
	 */
	@SuppressWarnings("unchecked")
	final E itemAt(int i) {
		return (E) items[i];
	}

	/**
	 *
	 * @param capacity  线程容量
	 * @param fair  是否为公平锁
	 */
	public ArrayBlockingQueue(int capacity, boolean fair) {
		if (capacity <= 0)
			throw new IllegalArgumentException();
		this.items = new Object[capacity];
		lock = new ReentrantLock(fair);
		notEmpty = lock.newCondition();  //读线程条件
		notFull = lock.newCondition();   //  写线程条件
	}

	public ArrayBlockingQueue(int capacity) {
		this(capacity, false);
	}

	public boolean put(E e) throws InterruptedException {
		checkNotNull(e);
		lock.lock();
		try {
			while (count == items.length){
				notFull.await();
			}
			items[putIndex] = e;//赋值
			////如果写索引写到队列的最后一个位置了，那么置为0
			if (++putIndex == items.length) {
				putIndex = 0;
			}
			count++;
			notEmpty.signalAll();

			return true;
		} finally {
			lock.unlock();
		}
	}

	public E take() throws InterruptedException {
		lock.lock();
		try {
			while (count == 0)
				notEmpty.await();
			E x = (E) items[takeIndex];
			items[takeIndex] = null;
			if (++takeIndex == items.length)
				takeIndex = 0;
			count--;

			notFull.signalAll();
			return x;

		} finally {
			lock.unlock();
		}
	}

	private static void checkNotNull(Object v) {
		if (v == null)
			throw new NullPointerException();
	}
}

```

ArrayBlockingQueue和LinkedBlockingQueue之前的区别

ArrayBlockingQueue维护的一个ReentrantLock,二LinkedBlockingQueue维护的是两个Reentrantlock

```java
 /** Lock held by take, poll, etc */
    private final ReentrantLock takeLock = new ReentrantLock();

    /** Wait queue for waiting takes */
    private final Condition notEmpty = takeLock.newCondition();

    /** Lock held by put, offer, etc */
    private final ReentrantLock putLock = new ReentrantLock();

    /** Wait queue for waiting puts */
    private final Condition notFull = putLock.newCondition();
```

![image-20200716211246623](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200716211246.png)