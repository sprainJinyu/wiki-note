---
title: 元-Condition通道
created: 2026-08-26
updated: 2026-08-26
tags: [元知识点, 锁, Condition]
---

# 元-Condition通道

Condition 相当于一把锁上的通道。  
从同一把锁可以创建多个独立的 Condition，自然形成分类。  

线程调用对应 Condition 的 `await()`，就进入该通道的等待状态并挂起。  
另一个线程在持有同一把锁时调用 `signal()` 或 `signalAll()`，就能精确唤醒对应通道上的线程。  

典型场景是生产者-消费者：一个通道等「有数据」，一个通道等「有空位」，实现精确唤醒，而不是把所有等待者都叫醒。
