---
layout: post
title: "JAVA多线程04-Condition"
date: 2015-07-23 10:24:26 +0800
comments: true
categories: 
---


在第一篇的多线程里面，有一个问题

> 子线程循环10次，主线程循环100次，然后子线程再循环10次，主线程再循环100次，这样子交替循环100次

当时，用的是synchronized, wait, notify来进行完成，这一次，使用Condition来完成这个

``` java 

public class ConditionDemo
{
	
	public static void main(String[] args)
	{
		Business business = new ConditionDemo().new Business();
		
		new Thread(new Runnable()
		{
			
			@Override
			public void run()
			{
				for(int i = 0; i < 50; i++)
				{
					business.sub(i);
				}
			}
		}).start();
		
		for(int i = 0; i < 50; i++)
		{
			business.main(i);
		}
	}
	
	
	class Business
	{
		private Lock lock = new ReentrantLock();
		private Condition condition = lock.newCondition();
		
		boolean isSub = true;
		
		public void sub(int i)
		{
			lock.lock();
			try
			{
				while(!isSub)
				{
					try
					{
						condition.await();
					}
					catch (InterruptedException e)
					{
						e.printStackTrace();
					}
				}
				
				for(int j = 0; j < 10; j++)
				{
					System.out.println("sub " + j + " looper of " + i);
				}
				
				isSub = false;
				condition.signal();
			}
			finally
			{
				lock.unlock();
			}
		}
		
		public void main(int i)
		{
			lock.lock();
			try
			{
				while (isSub)
				{
					try
					{
						condition.await();
					}
					catch (InterruptedException e)
					{
						e.printStackTrace();
					}
				}
				
				for(int j = 0; j < 100; j++)
				{
					System.out.println("main " + j + " looper of " + i);
				}
				
				isSub = true;
				condition.signal();
			}
			finally
			{
				lock.unlock();
			}
		}
	}

}
```

与之前的那个版本相比，Condition实现的优势就是，Condition可以有多个，这样就可以进行复杂的通信控制
可以实现更复杂的同步操作

下面是使用Condition实现的一个堵塞队列

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

这只是一个简单的堵塞队列的实现，在Java里面，已经有这样一个类了，ArrayBlockingQueue，
直接使用就可以了
