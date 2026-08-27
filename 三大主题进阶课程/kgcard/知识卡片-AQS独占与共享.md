---
title: 知识卡片-AQS独占与共享
created: 2026-08-27
updated: 2026-08-27
tags: [知识卡片, AQS]
---

# 知识卡片-AQS独占与共享

**组成元知识点**：  
[[元-AQS不是锁]] · [[元-state在可重入锁中的双重含义]] · [[元-Semaphore许可]] · [[元-CountDownLatch单向倒数]]

**核心表述**（基于用户口述润色）：  

AQS 定的是一套同步规则，不等于锁。  
独占：初始 `state` 为 0，有人占领后大于 0 并可重入累加，别人排队。  
共享有两种常用场景：Semaphore 的数量可以还回；CountDownLatch 单向清零后统一进入下一步。
