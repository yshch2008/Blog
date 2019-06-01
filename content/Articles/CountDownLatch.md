---
date: "2014-04-12T18:23:59+08:00"
publishdate: "2019-5-18+08:00"
lastmod: "2019-5-18+08:00"
draft: false
title: "CountDownLatch"
tags: ["修行", "Java", "并发编程"]
series: ["技术"]
categories: ["学习"]
toc: true
---

设置一个计数装置，然后设置某些动作触发减一，计数变成零时表示等待条件满足，让阻拦的线程全部跑起来。
不可以重复用，计数为零之后该锁就没有价值了。