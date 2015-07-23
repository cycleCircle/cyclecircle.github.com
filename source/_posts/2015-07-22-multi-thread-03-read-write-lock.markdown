---
layout: post
title: "JAVA多线程03-ReadWriteLock"
date: 2015-07-22 17:34:21 +0800
comments: true
categories: 
---


读写锁可以实现读写互斥，写写互斥，读读不互斥，这样可以极大的提高性能

``` java 

	public class CacheSimple
	{
		ReadWriteLock rwl = new ReentrantReadWriteLock();
		
		private Map<String, Object> cache = new HashMap<String, Object>();
		
		public Object get(String key)
		{
			rwl.readLock().lock();
			try
			{
				Object value = null;
				value = cache.get(key);
				if(value == null)
				{
					rwl.readLock().unlock();
					rwl.writeLock().lock();
					
					try
					{
						if(value == null)
						{
							//get data from data source
							//value = xxx.getRealData(key);
						}
						rwl.readLock().lock();
					}
					finally
					{
						rwl.writeLock().unlock();
					}
				}
				return value;
			}
			finally
			{
				rwl.readLock().unlock();
			}
		}
	}
```

加锁和解锁的位置很值得研究
