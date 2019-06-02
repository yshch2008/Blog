---
date: "2019-05-07+08:00"
publishdate: "2019-05-07+08:00"
lastmod: "2019-05-28+08:00"
draft: false
title: "ReentrantLock"
tags: ["修行", "Java", "并发编程"]
series: ["技术"]
categories: ["学习"]
toc: true
---

---

##### 主要手段是通过维护一个队列，然后让等待的下一个线程自旋，其余通过使用unsafe.Park方式挂起，切换和判断使用CAS操作而不是Synchronized实现竟态有序，因此效率比较高。

### 重入时，设置资源的state++，释放时设置为0。

资源 持有一个队列，放置请求该资源的线程。

1. 通常情况下，头节点处于占用状态，而第二个节点自旋请求该资源，条件满足时，将自己设为头节点，清掉前任头节点，并让后一个资源处于自旋状态。

2. 非公平的情况下，线程进入请求队列时会直接先尝试获取资源，如果恰好之前占用的线程刚刚释放掉资源，则直接把自己设为头节点，并让队列中之前在自旋的节点继续自旋请求，再清除掉前任头节点。否则插入到队尾，如果前一个节点是头节点则自旋，不是则挂起。

3. 在公平的情况下，则入队时不请求获取资源而是直接放到队尾，前一个节点是头节点则自旋，否则挂起。

由此可知，非公平锁允许后排队的线程先获得锁，这种情况下，由于该幸运线程不用经过挂起-再唤醒的过程，节省了开销，因此总体上非公平锁效率会高。

```java
public class Protected {
    Protected() {
    }

    private ReentrantLock lock = new ReentrantLock();
    private final Protected _protected;

    public Protected getSingleton(){
        if(_protected==null){
            lock.lock();
            if(_protected==null){
                _protected=new Protected();
            }
            lock.unlock();
        }
        return _protected;
    }

}
```