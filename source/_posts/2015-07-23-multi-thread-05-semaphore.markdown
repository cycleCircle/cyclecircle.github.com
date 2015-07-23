---
layout: post
title: "JAVA多线程-信号灯Semaphore"
date: 2015-07-23 11:18:47 +0800
comments: true
categories: 
---


Semaphore可以维护当前访问自身的线程个数，并提供了同步机制，使用Semaphore可以控制同时访问资源的
线程个数，例如，实现一个文件允许的并发访问数

``` java 
	public static void main(String[] args)
	{
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		final Semaphore semaphore = new Semaphore(3);
		for(int i = 0; i < 10; i++)
		{
			Runnable runnable = new Runnable()
			{
				
				@Override
				public void run()
				{
					try
					{
						semaphore.acquire();
					}
					catch (InterruptedException e)
					{
						e.printStackTrace();
					}
					
					System.out.println(Thread.currentThread().getName() + " come in, current has " + (3 - semaphore.availablePermits()));
					
					try
					{
						Thread.sleep((long)(Math.random() * 1000));
					}
					catch (InterruptedException e)
					{
						e.printStackTrace();
					}
					
					System.out.println(Thread.currentThread().getName() + " ready to go");
					
					semaphore.release();
					
					//下面代码有时候执行不准确，因为其没有和上面的代码合成原子单元
					System.out.println(Thread.currentThread().getName() + " gone, current has " + (3 - semaphore.availablePermits()));
				}
			};
			executorService.execute(runnable);
		}
		
	}
```

Semaphore可能通过构造函数传递一个true来实现FIFO的获取信号量的顺序


单个信号量的Semaphore对象可以实现互斥锁的功能，并且可以是由一个线程获得了“锁”，再由另一个线程释放
“锁”，这可应用于死锁的恢复的一些场合
