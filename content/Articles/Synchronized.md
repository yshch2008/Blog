---
date: "2014-04-12T18:23:59+08:00"
publishdate: "2019-5-7+08:00"
lastmod: "2019-5-7+08:00"
draft: false
title: "Synchronized"
tags: ["修行", "Java", "并发编程"]
series: ["技术"]
categories: ["学习"]
toc: true
---

Synchronized 使用minitor实现，由jvm维护，因此用完后自动释放。
可重入：获取一个对象的锁后，在没有释放前，该线程针对该对象的获取锁请求都将被允许。
不可中断：一个线程请求该锁时失败，无法调用该线程的interrup()方法使其放弃获取，该线程将一直等待直到获取锁。
唤醒机制：notify/notify all
等待机制：wait