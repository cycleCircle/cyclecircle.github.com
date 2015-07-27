---
layout: post
title: "JAVA多线程07-CountDownLatch"
date: 2015-07-24 15:17:56 +0800
comments: true
categories: 
---



CountDownLatch计数器，可以很容易的实现类似裁判与运动员，一份文件等待多个批复等问题

``` java 

public class CountDownLatchDemo
{
	
	public static void main(String[] args)
	{
		ExecutorService executorService = Executors.newCachedThreadPool();
		final CountDownLatch cdOrder = new CountDownLatch(1);
		final CountDownLatch cdAnswer = new CountDownLatch(3);
		for(int i = 0; i < 3; i++)
		{
			Runnable runnable = new Runnable()
			{
				
				@Override
				public void run()
				{
					try
					{
						System.out.println(Thread.currentThread().getName() + " ready to go");
						cdOrder.await();
						System.out.println(Thread.currentThread().getName() + " running");
						Thread.sleep((long)(Math.random() * 10000));
						System.out.println(Thread.currentThread().getName() + " return the result");
						cdAnswer.countDown();
					}
					catch (InterruptedException e)
					{
						e.printStackTrace();
					}
				}
			};
			
			executorService.execute(runnable);
		}
		
		
		try
		{
			Thread.sleep((long)(Math.random() * 10000));
			System.out.println(Thread.currentThread().getName() + " ready to send order");
			cdOrder.countDown();
			System.out.println(Thread.currentThread().getName() + " sent order");
			cdAnswer.await();
			System.out.println(Thread.currentThread().getName() + " get all results");
		}
		catch (InterruptedException e)
		{
			e.printStackTrace();
		}
		
		executorService.shutdown();
	}

}
```

结果：

``` java 

pool-1-thread-3 ready to go
pool-1-thread-1 ready to go
pool-1-thread-2 ready to go
main ready to send order
main sent order
pool-1-thread-3 running
pool-1-thread-1 running
pool-1-thread-2 running
pool-1-thread-3 return the result
pool-1-thread-2 return the result
pool-1-thread-1 return the result
main get all results
```
