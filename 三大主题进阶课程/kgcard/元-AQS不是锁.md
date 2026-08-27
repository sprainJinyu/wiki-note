---
title: 元-AQS不是锁
created: 2026-08-26
updated: 2026-08-26
tags: [元知识点, AQS]
---

# 元-AQS不是锁

AQS 是一种设计，不是一把锁。  
这套设计让多个线程抢同一份资源时能排队、能睡、能被叫醒，把争夺变成有序。  
`ReentrantLock` 是它的使用者之一。
