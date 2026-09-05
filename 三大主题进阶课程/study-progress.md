---
title: 原理卡进度看板
created: 2026-08-17
updated: 2026-09-05
tags: [学习进度, 原理卡]
---

# 原理卡进度看板

> **勾选标准**：能用自己的话复述该卡「今天只带走这一条」。

### 阶段 2：JDK 核心工具储备

- [x] **C07** `[日期: 2026-08-28]` `[复述: 能]`
- [x] **C08** `[日期: 2026-09-03]` `[复述: 能]`
- [x] **C09** [[lessons/concurrency/C09-ThreadLocal内存模型与隐患|ThreadLocalMap 弱引用 Key 强引用 Value]] `[日期: 2026-09-05]` `[复述: 能]`
- [ ] **C10** [[lessons/concurrency/C10-并发协作三剑客|CountDownLatch vs CyclicBarrier vs Semaphore]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **C11** `[日期: ____-__-__]` `[复述: 能 / 不能]`

## 复述时卡住的点

* C07：扩容的是工作线程数，不是把 corePoolSize 改大。
* C08：`SynchronousQueue` ≠ AQS 同步队列；口语用「容量为 0 的阻塞队列」。
* C09：污染的是同一条 Worker 的下一个任务，不是别的线程。
