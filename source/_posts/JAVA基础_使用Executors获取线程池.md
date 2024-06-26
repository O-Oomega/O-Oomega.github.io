---
title: JAVA基础_使用Executors获取线程池
date: 2024-6-26 15:00:00
categories:
- JAVA基础
tags:
- JAVA基础
---

### 使用Executors获取线程池

在Java编程中，线程池是管理和复用多个线程的机制。通过合理使用线程池，可以提高系统的并发性能，减少资源的浪费。Java提供了`Executors`类来方便地创建和管理线程池。本文将介绍`Executors`类常用的方法、线程池核心线程数量的计算方法以及使用线程池时的注意事项。

#### Executors常用方法

1. **newFixedThreadPool(int nThreads)**

   创建一个固定数量线程的线程池。即使某个线程由于执行异常而结束，线程池会补充一个新线程来替代它。这种线程池适用于任务数量已知且相对固定的场景。
   
   ```java
   ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5);
   ```

2. **newSingleThreadExecutor()**

   创建一个只有一个线程的线程池对象。如果该线程出现异常而结束，线程池会补充一个新线程。这种线程池适用于需要保证顺序执行任务的场景。
   
   ```java
   ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
   ```

3. **newCachedThreadPool()**

   创建一个可缓存线程的线程池。线程数量会随着任务的增加而增加，如果线程任务执行完毕且空闲了60秒，则会被回收掉。这种线程池适用于短时间内大量任务的场景。
   
   ```java
   ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
   ```

4. **newScheduledThreadPool(int corePoolSize)**

   创建一个定时任务的线程池。可以在给定的延迟后运行任务或者定期执行任务。这种线程池适用于需要定时或者周期性执行任务的场景。
   
   ```java
   ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
   ```

#### 线程池核心线程数量的计算

合理设置线程池的核心线程数量是保证系统性能的重要因素。一般来说，线程池的核心线程数量需要根据任务类型进行调整。

1. **计算密集型任务**

   计算密集型任务主要消耗CPU资源，因此核心线程数量通常设为`CPU的核数 + 1`。多出来的一个线程用于处理突发的上下文切换。

   ```java
   int cpuCoreCount = Runtime.getRuntime().availableProcessors();
   int corePoolSize = cpuCoreCount + 1;
   ```

2. **IO密集型任务**

   IO密集型任务主要消耗IO资源（如文件读写、网络请求），因此核心线程数量通常设为`CPU核数 * 2`，以便充分利用CPU等待IO操作完成的时间。

   ```java
   int corePoolSize = cpuCoreCount * 2;
   ```

#### Executors创建线程池的注意事项

在使用`Executors`创建线程池时，需要注意以下几点，以避免潜在的问题：

1. **FixedThreadPool和SingleThreadPool的任务队列长度为`Integer.MAX_VALUE`**

   这意味着任务队列可以非常长，可能会导致内存溢出。因此，在任务数量不确定的情况下，需谨慎使用这些线程池。

   ```java
   ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5);
   ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
   ```

2. **CachedThreadPool允许创建的线程数量为`Integer.MAX_VALUE`**

   由于线程数量可以无限增长，在大量任务的情况下可能会导致内存溢出。因此，在任务负载较高且执行时间较长的场景中，需要特别注意。

   ```java
   ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
   ```