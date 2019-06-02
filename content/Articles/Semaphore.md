---
date: "2019-05-17+08:00"
publishdate: "2019-05-17+08:00"
lastmod: "2019-05-28+08:00"
draft: false
title: "Semaphore"
tags: ["修行", "Java", "并发编程"]
series: ["技术"]
categories: ["学习"]
toc: true
---

### Semaphore

---

##### 类似于ReentrantLock在初始state可设置、重入不改state、release不直接重置state而是只释放该线程占用的数量。也类似Golang中的chennel

同样支持公平/非公平模式。

##### 场景：保护资源同时只被N个线程接触；在线程间通信调度（一个release，一个acquire）。
