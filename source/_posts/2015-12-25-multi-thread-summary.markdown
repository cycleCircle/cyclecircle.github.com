---
layout: post
title: "多线程总结"
date: 2015-12-25 15:00:16 +0800
comments: true
categories: 
---

### 线程安全

局部变量存储在线程自己的栈中。也就是说，局部变量永远也不会被多个线程共享。所以，基础类型的局部变量是线程安全的。例如

``` java
public void someMethod(){
  
  long threadSafeInt = 0;

  threadSafeInt++;
}
```
局部的对象引用

对象的局部引用和基础类型的局部变量不太一样。尽管引用本身没有被共享，但引用所指的对象并没有存储在线程的栈内。所有的对象都存在共享堆中。如果在某个方法中创建的对象不会被其它方法获得，也不会被非局部变量引用到，该方法，那么它就是线程安全的。实际上，哪怕将这个对象作为参数传给其它方法，只要别的线程获取不到这个对象，那它仍是线程安全的。下面是一个线程安全的局部引用样例：

``` java
public void someMethod(){
  
  LocalObject localObject = new LocalObject();

  localObject.callMethod();
  method2(localObject);
}

public void method2(LocalObject localObject){
  localObject.setValue("value");
}
```
样例中LocalObject对象没有被方法返回，也没有被传递给someMethod()方法外的对象。每个执行someMethod()的线程都会创建自己的LocalObject对象，并赋值给localObject引用。因此，这里的LocalObject是线程安全的。事实上，整个someMethod()都是线程安全的。即使将LocalObject作为参数传给同一个类的其它方法或其它类的方法时，它仍然是线程安全的。当然，如果LocalObject通过某些方法被传给了别的线程，那它就不再是线程安全的了。

对象成员

对象成员存储在堆上。如果两个线程同时更新同一个对象的同一个成员，那这个代码就不是线程安全的。例如：

``` java
public class NotThreadSafe{
    StringBuilder builder = new StringBuilder();
    
    public add(String text){
        this.builder.append(text);
    }	
}
```
如果两个线程同时调用同一个NotThreadSafe实例上的add()方法，就会有问题。例如：

``` java
NotThreadSafe sharedInstance = new NotThreadSafe();

new Thread(new MyRunnable(sharedInstance)).start();
new Thread(new MyRunnable(sharedInstance)).start();

public class MyRunnable implements Runnable{
  NotThreadSafe instance = null;
  
  public MyRunnable(NotThreadSafe instance){
    this.instance = instance;
  }

  public void run(){
    this.instance.add("some text");
  }
}
```
注意两个MyRunnable共享了同一个NotThreadSafe对象。因此，当它们调用add()方法时会造成竞态条件。

当然，如果这两个线程在不同的NotThreadSafe实例上调用call()方法，就不会有问题。下面是稍微修改后的例子：

``` java
new Thread(new MyRunnable(new NotThreadSafe())).start();
new Thread(new MyRunnable(new NotThreadSafe())).start();
```
现在两个线程都有自己单独的NotThreadSafe对象，调用add()方法时就会互不干扰，再也不会有问题了。所以非线程安全的对象仍可以通过某种方式来消除不安全的问题。



上面那些例子可以帮助你判断代码中对某些资源的访问是否是线程安全的。


如果一个资源的创建，使用，销毁都在同一个线程内完成，

且永远不会脱离该线程的控制，则该资源的使用就是线程安全的。

资源可以是对象，数组，文件，数据库连接，套接字等等。Java中你无需主动销毁对象，所以“销毁”指不再有引用指向对象。

即使对象本身线程安全，但如果该对象中包含其他资源（文件，数据库连接），整个应用也许就不再是线程安全的了。比如2个线程都创建了各自的数据库连接，每个连接自身是线程安全的，但它们所连接到的同一个数据库也许不是线程安全的。比如，2个线程执行如下代码：

检查记录X是否存在，如果不存在，插入X

如果两个线程同时执行，而且碰巧检查的是同一个记录，那么两个线程最终可能都插入了记录：

线程1检查记录X是否存在。检查结果：不存在

线程2检查记录X是否存在。检查结果：不存在

线程1插入记录X

线程2插入记录X

同样的问题也会发生在文件或其他共享资源上。因此，区分某个线程控制的对象是资源本身，还是仅仅到某个资源的引用很重要。


### 线程间的通信

线程间的通信，我们一般会使用 synchronized, wait, notify来完成，这样子来完成线程之间的切换，

在Java5之后，我们也可以使用Condition来完成

Condition的优势就是，Condition可以有多个，这样就可以进行复杂的通信控制 可以实现更复杂的同步操作

``` java 

public class ConditionBlockQueue
{
	
	private final Lock lock = new ReentrantLock();
	private final Condition notFull = lock.newCondition();
	private final Condition notEmpty = lock.newCondition();
	
	private final Object[] queue = new Object[100];
	
	private int putStr, takeStr, count;
	
	public void put(Object obj) throws InterruptedException
	{
		lock.lock();
		try
		{
			while(count == queue.length)
			{
				notFull.await();
			}
			
			queue[putStr] = obj;
			if(++putStr == queue.length)
			{
				putStr = 0;
			}
			count++;
			notEmpty.signal();
		}
		finally
		{
			lock.unlock();
		}
	}
	
	public Object take() throws InterruptedException
	{
		lock.lock();
		try
		{
			while(count == 0)
			{
				notEmpty.await();
			}
			
			Object obj = queue[takeStr];
			if(++takeStr == queue.length)
			{
				takeStr = 0;
			}
			count--;
			notFull.signal();
			return obj;
		}
		finally
		{
			lock.unlock();
		}
	}

}
```

如果我们的每个线程都有自己的对应的变量，这样可以用ThreadLocal

它就像我们自己通过Map来实现，key为Thread.currentThread，然后value就是线程相关的数据

但是ThreadLocal会将对应的线程的数据自动回收（当线程死亡时）所以不用担心内存泄漏，

当然，我们自己可以自动在线程结束的时候调用remove来把ThreadLocal里面的数据删除
