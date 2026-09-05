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
