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

>标记：`set`/`get` 路径上的过期 Key 探测清理、为何常声明 `private static final`、拦截器 `afterCompletion` 中 `remove`，本次未完整口述。
