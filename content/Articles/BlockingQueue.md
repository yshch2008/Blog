---
date: "2019-03-28+08:00"
publishdate: "2019-03-28+08:00"
lastmod: "2019-03-28+08:00"
draft: false
title: "BlockingQueue"
tags: ["修行", "Java", "并发编程"]
series: ["技术"]
categories: ["学习"]
toc: true
---


### BlockingQueue

---

| Operation/ If fail | Throws Exception | Special Value | Blocks  | Times out                 |      |
| ------------------ | ---------------- | ------------- | ------- | ------------------------- | ---- |
| **Insert**         | add(o)           | offer(o)      | put(o)  | offer(o,timeOut,timeUnit) |      |
| **Remove**         | remove(o)        | poll(o)       | take(o) | poll(timeOut,timeUnit)    |      |
| **Examine**        | element()        | peek()        |         |                           |      |

#### ArrayBlockQueue

数组式（初始化即设置容量，且无法修改）


```java
public class DemoApplication {
	static int id = 0;

	public static void main(String[] args) {

		BlockingQueue queue = new ArrayBlockingQueue(1);

		ExecutorService excutor = Executors.newFixedThreadPool(10);
		Runnable productTask = () -> {
			try {
				for (;;) {
					queue.put((Object) ("product" + (id)));
					id++;
					Thread.sleep(200);
				}

			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
		};

		Runnable consumeTask = () -> {
			try {
				for (;;) {
					System.out.println(queue.take());
					Thread.sleep(1000);
				}
			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
		};
		excutor.execute(productTask);
		excutor.execute(consumeTask);
		excutor.execute(consumeTask);
		excutor.execute(consumeTask);
	}

}
```

#### LinkedBlockQueue

链表式（可视为无容量限制），即生产者不会被阻塞。
