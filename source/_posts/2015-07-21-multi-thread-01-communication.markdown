---
layout: post
title: "JAVA多线程01-线程间的通信"
date: 2015-07-21 16:22:05 +0800
comments: true
categories: 
---


#题目

> 子线程循环10次，主线程循环100次，然后子线程再循环10次，主线程再循环100次，这样交替循环50次


``` java
public class TraditionalThreadCommunication
{
	
	public static void main(String[] args)
	{
		Business business = new TraditionalThreadCommunication().new Business();
		
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
		boolean isSub = true;
		
		public synchronized void sub(int i)
		{
			while(!isSub)
			{
				try
				{
					this.wait();
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
			this.notify();
		}
		
		public synchronized void main(int i)
		{
			while (isSub)
			{
				try
				{
					this.wait();
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
			this.notify();
		}
	}

}
```

## 要注意的点

* 要把业务封装到一个类里面，这样做同步或其他的就很容易了
* 同步方法里面的while很有必要，不要用if代替，因为会有一定的机会出现伪唤醒的状态，所以while体现了水平
