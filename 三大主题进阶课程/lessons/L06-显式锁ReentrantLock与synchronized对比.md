---
title: L06 - ReentrantLock 与 synchronized
created: 2026-08-17
updated: 2026-08-21
tags: [原理卡, 并发编程, ReentrantLock, synchronized]
lesson_id: L06
duration: 20min
---

# L06 · 两种锁差在中断、超时、多条件、公平

> **今天只带走这一条**：互斥这件事两者都能做。选 `ReentrantLock`，是因为需要响应中断、尝试超时、多路 `Condition`，或明确要公平锁。否则 `synchronized` 更简单，也不会忘了解锁。
> **用时**：15–20 分钟。锁升级细节在附录，通勤不必读。

---

## 1. 原理

|  | `synchronized` | `ReentrantLock` |
| :--- | :--- | :--- |
| 解锁 | 出代码块自动解 | 必须自己 `unlock()`，写在 `finally` |
| 中断 | 等锁时不能被中断打断 | `lockInterruptibly()` |
| 超时 | 不支持 | `tryLock(timeout, unit)` |
| 公平 | 只有非公平 | 可公平、默认非公平 |
| 条件队列 | 一个 `wait/notify` | 可多把 `Condition`，精确唤醒 |
| 实现位置 | JVM 监视器 | Java 代码，底层是 AQS（下一张卡） |

非公平吞吐通常更好：新来的线程可以先试一次锁，碰巧刚释放就免去一次挂起/唤醒。公平锁按排队来，避免饥饿，上下文切换更多。

`tryLock` 的原理就是「拿不到就返回，而不是睡死」。和业务上的重试、分布式锁不是同一层东西：

```java
if (lock.tryLock(500, TimeUnit.MILLISECONDS)) {
    try { /* 临界区 */ }
    finally { lock.unlock(); } // 只有拿到才解
}
```

---

## 2. 误区

**「有了 ReentrantLock，synchronized 可以不用了。」**  
多数互斥用 `synchronized` 更不容易写错。显式锁的成本是忘记 `unlock()`，会一直占着锁。

**「`tryLock` 失败了也要 `unlock()`。」**  
没拿到锁就 `unlock()` 会抛 `IllegalMonitorStateException`。只在成功分支的 `finally` 里解。

---

## 3. 自检

1. **为什么非公平锁往往更快？**
   *提示：允许插队试锁，减少一次线程挂起和唤醒。*
2. **`tryLock` 返回 false 时调用 `unlock()` 会怎样？**
   *提示：`IllegalMonitorStateException`。*

---

## 4. 资料

* `../materials/concurrency/redspider-concurrent/article/03/14.md`
* 《Java 并发编程实战》第 13 章

---

## 附录（通勤不必读）

JDK 6 之后 `synchronized` 曾经按竞争升级：无锁 → 偏向锁 → 轻量级锁（CAS 自旋）→ 重量级锁（进操作系统等待队列）。竞争少时不必一上来就入内核。新 JDK 里偏向锁已默认关闭或移除，知道「竞争少则轻、竞争多则入内核」即可。这不改变上面那张对比表。