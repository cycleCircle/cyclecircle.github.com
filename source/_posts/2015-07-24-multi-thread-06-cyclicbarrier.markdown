---
layout: post
title: "JAVA多线程06-CyclicBarrier"
date: 2015-07-24 14:55:29 +0800
comments: true
categories: 
---


CyclicBarrier可以实现的功能是：多个线程一起工作，要同时达到某种效果后才能做下一件事，就好像
一群人出去玩，每个人是一个线程，约定在一个地方集合，只有当全部人到达那个地方再能前往目的地。
CyclicBarrier就可以解决这样的问题

``` java 

public class CyclicBarrierDemo
{
	
	public static void main(String[] args)
	{
		ExecutorService executorService = Executors.newCachedThreadPool();
		final CyclicBarrier cyclicBarrier = new CyclicBarrier(3);
		for(int i = 0; i < 3; i++)
		{
			Runnable runnable = new Runnable()
			{
				
				@Override
				public void run()
				{
					try
					{
						Thread.sleep((long)(Math.random() * 10000));
						System.out.println(Thread.currentThread().getName() + " arrived 1 " + 
									(cyclicBarrier.getNumberWaiting() == 2 ? "all number arrived, keep going" : "waiting"));
						cyclicBarrier.await();
						
						Thread.sleep((long)(Math.random() * 10000));
						System.out.println(Thread.currentThread().getName() + " arrived 2 " + 
									(cyclicBarrier.getNumberWaiting() == 2 ? "all number arrived, keep going" : "waiting"));
						cyclicBarrier.await();
						
						Thread.sleep((long)(Math.random() * 10000));
						System.out.println(Thread.currentThread().getName() + " arrived 3 " + 
									(cyclicBarrier.getNumberWaiting() == 2 ? "all number arrived, keep going" : "waiting"));
						cyclicBarrier.await();
						
					}
					catch (Exception e)
					{
						e.printStackTrace();
					} 
				}
			};
			
			executorService.execute(runnable);
		}
		executorService.shutdown();
	}

}
```
