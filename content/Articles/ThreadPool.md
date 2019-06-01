---
date: "2014-04-12T18:23:59+08:00"
publishdate: "2019-5-20+08:00"
lastmod: "2019-5-20+08:00"
draft: false
title: "ThreadPool"
tags: ["修行", "Java", "并发编程"]
series: ["技术"]
categories: ["学习"]
toc: true
---

线程池的原理比较简单，目的主要是减少频繁销毁、创建线程的开销以及在多数情况下可以让申请线程的操作得到更快的响应。

java.util.concurrent包提供了ExecutorService简化创建线程池：

```java
ExecutorService excutor = Executors.newFixedThreadPool(10);//只需要最后填一个数量即可

```
打开源码：
```java
   public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

可以看到其实调用的还是ThreadPoolExecutor，继续打开：
```java 
    // Public constructors and methods

    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters and default thread factory and rejected execution handler.
     * It may be more convenient to use one of the {@link Executors} factory
     * methods instead of this general purpose constructor.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     * @param unit the time unit for the {@code keepAliveTime} argument
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```

主要方法execute ：
```java
    /**
     * Executes the given task sometime in the future.  The task
     * may execute in a new thread or in an existing pooled thread.
     *
     * If the task cannot be submitted for execution, either because this
     * executor has been shutdown or because its capacity has been reached,
     * the task is handled by the current {@code RejectedExecutionHandler}.
     *
     * @param command the task to execute
     * @throws RejectedExecutionException at discretion of
     *         {@code RejectedExecutionHandler}, if the task
     *         cannot be accepted for execution
     * @throws NullPointerException if {@code command} is null
     */
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
```

主要参数：corePoolSize,maximumPoolSize,(keepAliveTime,unit),[RejectedExecutionHandler handler]

申请时主要策略：
1. 假如池中的被占用的线程数量少于corePoolSize，那么有新的线程申请时开启一个线程供消费；
2. 假如池中的被占用的线程数量大于等于corePoolSize，但小于maximumPoolSize，同样开启新的线程供消费；
3. 假如池中的线程都被占用，则根据抢占策略执行 `抢占最老任务(DiscardOldestPolicy)`、`忽略(DiscardPolicy)`、`抛异常（AbortPolicy）`、`排队（CallerRunsPolicy）`,默认的时抛异常
线程的任务执行完毕时主要策略：
1. 如果关闭该线程后就绪或执行状态的线程数小于corePoolSize，则不销毁线程
2. 否则等待一段时间(keepAliveTime,unit)后销毁线程

另外也可以通过`prestartCoreThread`或者`prestartAllCoreThreads`让线程池预热核心线程以供消费。