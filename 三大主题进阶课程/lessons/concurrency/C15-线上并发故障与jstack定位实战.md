---
title: C15 - 线上并发故障与 jstack 定位实战
created: 2026-08-28
updated: 2026-08-28
tags: [原理卡, 并发编程, 线上排查, jstack, 死锁, CPU飙高]
lesson_id: C15
duration: 20min
---

# C15 · 线上排查：3步揪出高CPU，看懂jstack三状态

> **今天只带走这一条**：线上排查并发三步走：`top -Hp <pid>` 找耗 CPU 最高的线程十进制 ID ➔ `printf "%x
" <tid>` 转十六进制 ➔ `jstack <pid> | grep -A 30 <nid>` 快速定位代码行。**RUNNABLE 紧咬代码行是死循环，WAITING 扎堆是线程池或连接池打满，BLOCKED 是死锁互锁。**
> **用时**：15–20 分钟。

---

## 1. 原理

### 一、 60 秒极速定位高 CPU 代码行

```bash
# 第 1 步：找到 Java 进程中 CPU 消耗最高的线程 ID (例如 28314)
top -Hp 28000

# 第 2 步：将十进制线程 ID 转为十六进制 (例如 28314 ➔ 0x6e9a)
printf "%x
" 28314

# 第 3 步：打印堆栈并匹配十六进制 nid
jstack 28000 | grep -A 30 "nid=0x6e9a"
```

---

### 二、 jstack 三大核心线程状态识别直觉

```
1. RUNNABLE          ──> 线程正在运行或争抢 CPU。
                         若堆栈多次抓取均停在同一行业务代码/正则/死循环 ➔ 定位 CPU 100% 根因！

2. WAITING (parking) ──> 线程处于休眠等待。
                         若几百个线程全堆在 getTask() / take() ➔ 属于空闲正常；
                         若全堆在数据库连接池/下游 HTTP client 的 await() ➔ 连接池耗尽或下游阻塞！

3. BLOCKED (on monitor) ──> 线程被 synchronized 阻塞。
                         若出现互相等待对方持有的锁 ➔ jstack 底部会直接输出：
                         "Found 1 deadlock." 并附带完整锁环与代码行！
```

---

## 2. 误区

**「看到 jstack 中有 200 个线程处于 WAITING 状态就觉得系统出故障了。」**  
**误判！** 线程池中没有任务时的空闲 Worker 线程，默认就是通过 `LockSupport.park` 处于 WAITING 状态挂起，不消耗任何 CPU。只有当线程处于 `RUNNABLE` 但不交出 CPU，或者 `BLOCKED` 扎堆才代表异常。

**「线上排查死锁必须停机重启，再慢慢看业务日志。」**  
**落后做法！** `jstack -l <pid>` 命令执行极其轻量，在几毫秒内即可输出快照，JVM 内部会自动检测监视器锁与 `ReentrantLock` 的循环等待环，并在报告尾部清晰打印死锁详情。

---

## 3. 自检

1. **执行 `jstack` 抓到的堆栈中，`nid` 代表什么含义？为什么要用 `printf "%x"` 转换？**  
   *提示：`nid` 即 Native Thread ID（操作系统层面的轻量级进程 ID），`jstack` 中用十六进制表示，而 Linux `top` 命令输出的是十进制，必须转换后才能 grep 匹配。*
2. **当微服务接口 RT 剧增，大量线程卡在 `WAITING (parking)` 且堆栈显示 `com.alibaba.druid.pool.DruidDataSource.getConnection` 时，可能的原因是什么？**  
   *提示：数据库连接池打满，所有工作线程都在排队等待获取可用连接，需要排查慢 SQL 或连接泄漏。*

---

## 4. 资料

* 《深入理解 Java 虚拟机》第 4 章（虚拟机性能监控与故障处理工具）
* 常用命令速查：`jstack`、`jstat -gcutil`、`jcmd`
