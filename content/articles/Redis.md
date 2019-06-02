---
date: "2019-03-28+08:00"
publishdate: "2019-03-28+08:00"
lastmod: "2019-05-28+08:00"
draft: false
title: "Redis"
tags: ["修行", "Java", "分布式"]
series: ["技术"]
categories: ["学习"]
toc: true
---


```java
public Object getObject(long id){
    redisTemplate.setKeySerializer(new StringRedisSerializer());

    Object ob=(Object) redisTemplate.opsForValue().get("objectKey");
    if(null==ob){
        ob=obMapper.selectByPrimaryKey(id);
        redisTemplate.opsForValue().set(k:"objectKey",object);
    }
    return ob;
}
```