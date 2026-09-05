---
title: 元-Semaphore许可
created: 2026-08-27
updated: 2026-09-05
tags: [元知识点, AQS, Semaphore, C10]
---

# 元-Semaphore许可

`Semaphore` 的 `state` 是还能发出去的许可数。  
`acquire()` 拿走一张，没有就等；`release()` 还回一张。池子有限，可反复用。

管的是此刻同时持有的人数，不是单位时间的请求数。

## 巧思 / 细则

没有持有者。A `acquire()` 之后可以由 B `release()`。和 `ReentrantLock` 不是一把锁。
