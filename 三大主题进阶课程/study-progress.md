---
title: 原理卡进度看板
created: 2026-08-17
updated: 2026-09-05
tags: [学习进度, 原理卡]
---

# 原理卡进度看板

- [x] **C07** `[日期: 2026-08-28]`
- [x] **C08** `[日期: 2026-09-03]`
- [x] **C09** `[日期: 2026-09-05]`
- [x] **C10** [[lessons/concurrency/C10-并发协作三剑客|CountDownLatch vs CyclicBarrier vs Semaphore]] `[日期: 2026-09-05]` `[复述: 能]`
- [ ] **C11** [[lessons/concurrency/C11-ConcurrentHashMap与LongAdder|ConcurrentHashMap 锁桶头与 LongAdder 分段累加]] `[日期: ____-__-__]` `[复述: 能 / 不能]`

## 复述时卡住的点

* C08：口语用「容量为 0 的阻塞队列」，不要和 AQS 同步队列混。
* C09：污染的是同一条 Worker 的下一个任务。
* C10：Latch 不是「所有人一起过门」；Semaphore 不是时间窗口。
