---
date: "2014-04-12T18:23:59+08:00"
publishdate: "2019-5-19+08:00"
lastmod: "2019-5-19+08:00"
draft: false
title: "CyclicBarrier"
tags: ["修行", "Java", "并发编程"]
series: ["技术"]
categories: ["学习"]
toc: true
---


一般翻译为栅栏，但其实更像水坝蓄水和泄洪：阻拦线程，直到拦住的线程到达一定数量，然后让这些线程重新开始运行。

可以重复用，放掉一批后可以设置阻拦另一批。