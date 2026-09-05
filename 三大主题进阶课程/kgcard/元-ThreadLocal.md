---
title: 元-ThreadLocal
created: 2026-09-05
updated: 2026-09-05
tags: [元知识点, ThreadLocal, C09]
---

# 元-ThreadLocal

`ThreadLocal` 实例是查当前线程上那张 [[元-ThreadLocalMap]] 的钥匙。  
常做成 `static`，各条线程共用同一把钥匙；Value 按线程隔离。

一个实例对当前线程只占一格。再 `set` 就覆盖，不追加。

`get` / `set` / `remove` 走：`Thread.currentThread()` → 该线程的 `threadLocals` → 用 `this` 当 Key。

没有全局的「线程 → Map」注册表。
