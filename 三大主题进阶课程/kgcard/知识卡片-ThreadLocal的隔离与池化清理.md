---
title: 知识卡片-ThreadLocal的隔离与池化清理
created: 2026-09-05
updated: 2026-09-05
tags: [知识卡片, ThreadLocal, C09]
---

# 知识卡片-ThreadLocal的隔离与池化清理

**组成元知识点**：  
[[元-ThreadLocal]] · [[元-ThreadLocalMap]] · [[元-工作线程Worker]]

**核心表述**（基于用户口述润色）：

`ThreadLocalMap` 挂在 `Thread` 上。Key 是 `ThreadLocal` 实例，Value 是存放的值。按线程隔离，不按任务隔离。

`ThreadLocal` 是薄入口：`get` / `set` / `remove` 取当前线程的那张表，用这个实例当 Key。没有全局管理线程与地图对应关系的第三张表。

Key 弱引用：业务侧把 `ThreadLocal` 置空后，钥匙实例可以被回收。  
Value 强引用：`Thread` → Map → `Entry` → Value，不 `remove()` 就跟着还活着的线程走。

线程池里同一条 Worker 会接下一单任务。不 `remove()`：Value 占着堆（泄漏），下一单可能 `get` 到上一单的值（污染）。  
污染的是同一条线程的下一个任务，不是别的线程。

过期 Key：地图里 Key 已是 `null`、Value 还在。局部 `new` 出来的钥匙被 GC 后会走这条；`static` 钥匙不走。顺路清理发生在这条线程后续的 `get` / `set` / `remove`，不是定时扫。Tomcat 常驻 Worker 上的问题是钥匙还在、格子还在，要靠 `remove()`。

`private static final`：全进程同一把钥匙，`set` / `get` / `remove` 对得上同一格。

`remove()` 放在一定执行到的出口：自己的方法里用 `finally`；拦截器里用 `afterCompletion`（成败都会到，相当于请求链路上的 `finally`）。
