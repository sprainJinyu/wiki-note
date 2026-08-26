---
title: L07 - AQS 三要素
created: 2026-08-17
updated: 2026-08-21
tags: [原理卡, 并发编程, AQS]
lesson_id: L07
duration: 20min
---

# L07 · AQS：state、持锁线程、等待队列

> **今天只带走这一条**：`AbstractQueuedSynchronizer` 只干三件事——一个 `state`、记下谁拿着、拿不到的线程进队列里睡。`ReentrantLock`、`Semaphore`、`CountDownLatch` 都是在这三件上面改「state 表示什么」。
> **用时**：15–20 分钟。加锁逐步流程在附录，通勤不必读。

---

## 1. 原理

```
        volatile int state          ← 同步状态（独占锁里：0 空闲，>0 重入次数）
        exclusiveOwnerThread        ← 当前持锁线程（用来判断重入）

  Head（哨兵） ⇄ Node ⇄ Node ⇄ Tail     ← 没拿到的线程，一个节点一个
```

1. **`state`**：资源现在怎样。改它必须用 CAS，保证「看见 0 就改成 1」不会两个人同时成功。
2. **持锁线程**：独占模式下，谁已经拿到。同一线程再 acquire，只给 `state+1`，这就是可重入。
3. **等待队列**：失败的线程包成 Node 挂到队尾，`LockSupport.park` 睡；释放的人 `unpark` 队头后面那个。

队列头部的 **Dummy / 哨兵节点不是「正在持锁的那个线程」**。持锁线程记在 `exclusiveOwnerThread` 上，常常根本不在队列里。哨兵用来避免头尾是 null 时的分叉逻辑：释放时永远「唤醒 head 的后继」。

独占：同一时刻一个人用（`ReentrantLock`）。共享：可以有多个人用，`state` 表示许可数（`Semaphore`、`CountDownLatch`）。

Node 上还有 `waitStatus`。值为 `SIGNAL`（-1）时表示后继已经或即将 `park`，当前节点释放时必须 `unpark` 后继。通勤记住这个责任即可，不必背全部状态码。

---

## 2. 误区

**「Head 节点就是当前持锁线程。」**  
Head 是哨兵。持锁信息在 `exclusiveOwnerThread` 和 `state`。

**「AQS 是一种锁。」**  
AQS 是同步器骨架。锁、闩、信号量是它的用户。

---

## 3. 自检

1. **队列头部放一个空节点，图省事省在哪？**
   *提示：唤醒和入队都对着「head 的后继 / tail」写，少处理 null。*
2. **`waitStatus == SIGNAL` 对当前节点意味着什么？**
   *提示：后继已经（或即将）park，你释放时有责任 unpark 后继。*

---

## 4. 资料

* `../materials/concurrency/redspider-concurrent/article/02/11.md`
* 《Java 并发编程的艺术》第 5 章

---

## 附录：独占 acquire（通勤不必读）

```
acquire(1)
  1. tryAcquire → 成功则结束
  2. 失败：CAS 把自己加到队尾
  3. 若前驱是 head，再 tryAcquire 一次；成功则自己成为新 head
  4. 否则把前驱标成 SIGNAL，LockSupport.park
释放：tryRelease 把 state 减到 0，unpark(head 的后继)
```

今天能复述三要素即可。流程是为了以后读源码时有一张地图，不是通勤目标。
