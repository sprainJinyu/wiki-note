---
title: L03 - 指令重排序与 Happens-Before 规则
created: 2026-08-17
updated: 2026-08-21
tags: [原理卡, 并发编程, Happens-Before, 指令重排]
lesson_id: L03
duration: 20min
---

# L03 · 没有 happens-before，就不保证可见和有序

> **今天只带走这一条**：A happens-before B，才保证 A 的结果对 B 可见、且不能把 A 排到 B 后面。没有这条边，JMM 不保证。程序员用下面 8 条规则连边，不必自己推屏障。
> **用时**：15–20 分钟。

---

## 1. 原理

源码顺序不是最终执行顺序。编译器、CPU、内存子系统都可能重排。单线程有 **as-if-serial**：看起来不能把结果改错。多线程没有这层保护——除非两个操作之间有 happens-before。

**happens-before 不是「时钟上谁先发生」**，是 JMM 给程序员的承诺：有这条边，就有可见性和有序性。

| 规则 | 含义 |
| :--- | :--- |
| 程序顺序 | 同一线程里，前面的操作 happens-before 后面的（仍允许重排，但不能让单线程结果变错） |
| 监视器锁 | 对同一把锁的 `unlock` happens-before 随后的 `lock` |
| volatile | 对某 `volatile` 的写 happens-before 随后对它的读 |
| 传递性 | A hb B 且 B hb C，则 A hb C |
| 线程启动 | 调用 `start()` 之前的写，对子线程 `run()` 可见 |
| 线程终止 | 子线程里的操作 happens-before 另一线程检测到它结束（如 `join()` 返回） |
| 中断 | `interrupt()` happens-before 被中断线程检测到中断 |
| 终结 | 构造结束 happens-before `finalize()` |

传递性是日常最常用的一条：用一个 `volatile` 当开关，开关写之前的普通字段写，对读到开关为 true 的线程一并可见。

```text
线程 A：payload = ...;   ready = true;     // ready 是 volatile
线程 B：if (ready) 使用 payload;            // 读到 true 则 payload 可见
```

`payload` 不必是 volatile。边是：A 里 payload 的写 hb ready 的写（程序顺序），ready 的写 hb ready 的读（volatile 规则），ready 的读 hb 使用 payload（程序顺序），再靠传递性连起来。

---

## 2. 误区

**「两个操作没有 happens-before，JVM 也必须按源码顺序执行。」**  
没有。只要不破坏单线程 as-if-serial，就可以重排，另一个线程可能看见中间态或旧值。

**「happens-before 表示物理时间上 A 一定先于 B。」**  
它是可见性/有序性承诺，不是墙上时钟。

---

## 3. 自检

1. **上一例里，为什么 `payload` 可以不是 volatile？**
   *提示：volatile 写之前的普通写，会随这条写一起对随后的 volatile 读可见（传递性）。*
2. **没有 happens-before 的两个操作，能不能颠倒？**
   *提示：可以，只要单线程结果不变。*

---

## 4. 资料

* `../materials/concurrency/redspider-concurrent/article/02/7.md`
* 《Java 并发编程实战》第 16.1 节
