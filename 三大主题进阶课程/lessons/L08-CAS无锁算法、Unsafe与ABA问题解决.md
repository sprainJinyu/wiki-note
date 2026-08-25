---
title: L08 - CAS 与 ABA
created: 2026-08-17
updated: 2026-08-21
tags: [原理卡, 并发编程, CAS, ABA]
lesson_id: L08
duration: 20min
---

# L08 · CAS：值还是预期值，才换成新值

> **今天只带走这一条**：CAS 用一次硬件原子操作做「如果还是 E，就改成 N」。ABA 是值又变回 E，但中间被人改过；只比值会误判成功。
> **用时**：15–20 分钟。

---

## 1. 原理

CAS（Compare-And-Swap）三个数：

* **V**：内存里现在的值
* **E**：我以为的旧值
* **N**：想写成的新值

若 `V == E`，则 `V ← N` 并成功；否则失败，什么也不改。比较和写入是一条 CPU 指令完成的（x86 上是 `cmpxchg` 一类），软件里拆不成「先看再写」两步。

失败时的常见做法是自旋：重新读 V，再 CAS。竞争极高时会空转烧 CPU，那是适用边界，不是 CAS 的定义。

**ABA：**

```
线程 1 读到 A
线程 2 把 A 改成 B，再改回 A
线程 1 CAS(A → C) 成功
```

值相同，历史不同。链表栈顶弹出又压入时，指针可能已经指向一块你不再拥有的节点。解决办法是**值旁边再带版本号**，比较时两个都要比。

Java 里：`AtomicInteger` 只管值；需要防 ABA 用 `AtomicStampedReference`（引用 + stamp）。一次 `compareAndSet(期望引用, 新引用, 期望 stamp, 新 stamp)`，不要先把 stamp 取出来做完别的事再 CAS——中间 stamp 可能已经变了。

数据库里同一思想：`UPDATE t SET status='RUNNING', version=version+1 WHERE id=? AND version=?`。影响行数为 0 就是 CAS 失败。

---

## 2. 误区

**「CAS 成功说明中间没人动过。」**  
只说明**当前值**还是你预期的那个。中间可以 A→B→A。

**「先 `getStamp()`，再去做判断，最后 `compareAndSet(..., stamp, stamp+1)` 一定安全。」**  
判断和 CAS 之间别人可以先成功。正确用法是：期望值和期望 stamp 来自同一次读取，失败就整次重来。

---

## 3. 自检

1. **高竞争下纯自旋 CAS 的副作用是什么？**
   *提示：大量失败线程空转，CPU 被打满。*
2. **`WHERE id=? AND status='PENDING'` 这种更新，和 CAS 的哪一点像？**
   *提示：用当前状态当预期值，只有匹配的那一次更新成功。*

---

## 4. 资料

* `../materials/concurrency/redspider-concurrent/article/02/10.md`
* 《Java 并发编程的艺术》第 7 章
