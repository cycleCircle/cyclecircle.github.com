---
layout: post
title: "JAVA多线程02-ThreadLocal"
date: 2015-07-22 10:57:52 +0800
comments: true
categories: 
---


Java线程相关的变量，每个线程都有自己的对应的变量，这样可以用ThreadLocal

还可以通过Map来实现，key为Thread.currentThread，然后value就是线程相关的数据

``` java 

public class ThreadScopeData
{
	
	public static void main(String[] args)
	{
		for(int i = 0; i < 2; i++)
		{
			new Thread(new Runnable()
			{
				
				@Override
				public void run()
				{
					int id = new Random().nextInt();
					MyThreadScopeData.getThreadInstance().setId(id);
					System.out.println(Thread.currentThread().getName() + " set data " + id);
					
					new BusinessA().get();
					new BusinessB().get();
					
				}
			}).start();
		}
	}
	
	
	static class BusinessA
	{
		public void get()
		{
			System.out.println("A from " + Thread.currentThread() + " get data " + MyThreadScopeData.getThreadInstance().getId());
		}
	}
	
	static class BusinessB
	{
		public void get()
		{
			System.out.println("B from " + Thread.currentThread() + " get data " + MyThreadScopeData.getThreadInstance().getId());
		}
	}
	
	
	static class MyThreadScopeData
	{
		private int id;
		
		private MyThreadScopeData()
		{}
		
		private static ThreadLocal<MyThreadScopeData> threadLocal = new ThreadLocal<MyThreadScopeData>();
		
		public static MyThreadScopeData getThreadInstance()
		{
			MyThreadScopeData instance = threadLocal.get();
			if(instance == null)
			{
				instance = new MyThreadScopeData();
				threadLocal.set(instance);
			}
			return instance;
		}

		public int getId()
		{
			return id;
		}

		public void setId(int id)
		{
			this.id = id;
		}
		
	}

}
```

ThreadLocal会将对应的线程的数据自动回收（当线程死亡时）所以不用担心内存泄漏，当然，还可以自动在
线程结束的时候调用remove来把ThreadLocal里面的数据删除
