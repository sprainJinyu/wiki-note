---
title: 元-ThreadLocalMap
created: 2026-09-05
updated: 2026-09-05
tags: [元知识点, ThreadLocal, C09]
---

# 元-ThreadLocalMap

`ThreadLocalMap` 是 `Thread` 的实例字段 `threadLocals` 所指的那张表。  
一条线程一根引用、一张表。

表内 `Entry`：Key 是 [[元-ThreadLocal]] 实例，Value 是 `set` 进去的对象。  
多个 `ThreadLocal` 实例就是同一张表上的多把 Key。

Key 是弱引用，Value 是强引用。

## 巧思 / 细则

过期 Key：钥匙实例已被 GC，格子上 Key 变 `null`，Value 还在。  
等这条线程下一次 `get` / `set` / `remove` 顺路拆格子，不是后台扫。  
`static` 钥匙一直被类持有，不走过期清理；池化 Worker 要靠 `remove()`。
