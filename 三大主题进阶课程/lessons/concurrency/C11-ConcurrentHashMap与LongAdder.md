---
title: C11 - ConcurrentHashMap 锁粒度与 LongAdder 分段累加
created: 2026-08-28
updated: 2026-08-28
tags: [原理卡, 并发编程, ConcurrentHashMap, LongAdder, CAS]
lesson_id: C11
duration: 20min
---

# C11 · 高并发容器与计数：锁桶头与分散热点

> **今天只带走这一条**：JDK 8 的 `ConcurrentHashMap` 采用 **CAS 赋空槽 + synchronized 锁桶首节点**，扩容时多线程协同转移（ForwardingNode）；高并发写计数用 `LongAdder` 将写热点**分散到 Cell 数组（Striped64 思想）**，彻底避免全员 CAS 争抢同一变量导致 CPU 烧死。
> **用时**：15–20 分钟。

---

## 1. 原理

### 一、 ConcurrentHashMap (JDK 8) 并发控制

```
Node[] table
 ├── [0] ── null  ──────────> 插入时：CAS 尝试直接写入首节点（无锁）
 ├── [1] ── Node ➔ Node ───> 插入/更新：synchronized(Node[1]) 只锁当前链表头！
 ├── [2] ── TreeRoot ──────> 插入/更新：synchronized(TreeRoot) 只锁红黑树根！
 └── [3] ── ForwardingNode ─> hash=-1，表示正在扩容，其他线程进来协助数据搬迁
```

* **读操作完全无锁**：`Node.val` 和 `Node.next` 全由 `volatile` 修饰，保证可见性。
* **扩容支持多线程协同**：由 `sizeCtl` 标记控制，遇到 `ForwardingNode` 的线程主动帮助搬移其他分段。

---

### 二、 `AtomicLong` vs `LongAdder`（高并发计数）

```
AtomicLong:   所有线程 ─── CAS 争抢同一地址 ───> volatile long value (极高并发下大量空转)

LongAdder:    线程 1 ──── CAS ───> Cell[0]
              线程 2 ──── CAS ───> Cell[1]   ────> 最终 sum() = base + ∑Cell[i]
              线程 3 ──── CAS ───> base
```

* 内部维护 `base` 和 `Cell[]`。无竞争时直接改 `base`；一旦 CAS 冲突，通过线程 Hash 映射到不同 `Cell` 独立累加，将单点热点打散。

---

## 2. 误区

**「`ConcurrentHashMap` 每个方法都是线程安全的，所以 `if (!map.containsKey(k)) map.put(k, v);` 也是安全的。」**  
**重大逻辑漏洞！** 多个线程安全的方法组合在一起并不是原子操作。在 `containsKey` 与 `put` 之间会被其他线程插队。正确做法是使用原子复合方法：`map.computeIfAbsent(k, key -> load(key))` 或 `putIfAbsent`。

**「`LongAdder` 在任何场景下都比 `AtomicLong` 好。」**  
**不一定！** `LongAdder` 用空间换时间，在低并发时开销略高于 `AtomicLong`；且 `sum()` 获取的是最终一致性近似值（求和过程中并发写可能未完全计入）。需要强一致严格顺序号（如订单号自增）必须用 `AtomicLong`。

---

## 3. 自检

1. **为什么 JDK 8 放弃了 JDK 7 的 Segment 分段锁设计？**  
   *提示：JDK 7 最多只有 16 个 Segment（并发度上限 16）；JDK 8 将锁粒度细化到 table 数组的每一个桶（Bucket），有多少个桶并发度就是多少，且内存更省。*
2. **微服务做单机 QPS 限流或点赞计数，在高吞吐下推荐选哪个？**  
   *提示：推荐 `LongAdder`，避免高并发下 CAS 剧烈自旋烧高 CPU。*

---

## 4. 资料

* `../../materials/concurrency/redspider-concurrent/article/03/15.md`
* 《Java 并发编程的艺术》第 6 章
