---
layout: post
title: "JAVA多线程08-Exchanger"
date: 2015-07-24 15:33:12 +0800
comments: true
categories: 
---


Exchanger用来实现两个线程之间的数据交换，如果一个线程先到，那就会等待与它交换数据的线程也到达，
才能交换，交换完成后才能继续做后面的事情

``` java 

public class ExchangerDemo
{
	
	public static void main(String[] args)
	{
		ExecutorService executorService = Executors.newCachedThreadPool();
		Exchanger<String> exchanger = new Exchanger<String>();
		
		executorService.execute(new Runnable()
		{
			
			@Override
			public void run()
			{
				try
				{
					String str = "aaa";
					System.out.println(Thread.currentThread().getName() + " ready to change the data " + str);
					Thread.sleep((long)(Math.random() * 10000));
					String str1 = exchanger.exchange(str);
					System.out.println(Thread.currentThread().getName() + " get data " + str1);
				}
				catch (InterruptedException e)
				{
					e.printStackTrace();
				}
			}
		});
		
		executorService.execute(new Runnable()
		{
			
			@Override
			public void run()
			{
				try
				{
					String str = "bbb";
					System.out.println(Thread.currentThread().getName() + " ready to change the data " + str);
					Thread.sleep((long)(Math.random() * 10000));
					String str1 = exchanger.exchange(str);
					System.out.println(Thread.currentThread().getName() + " get data " + str1);
				}
				catch (InterruptedException e)
				{
					e.printStackTrace();
				}
			}
		});
		
		executorService.shutdown();
	}

}
```

结果：

``` java 

pool-1-thread-1 ready to change the data aaa
pool-1-thread-2 ready to change the data bbb
pool-1-thread-2 get data aaa
pool-1-thread-1 get data bbb
```
