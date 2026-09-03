---
title: 元-FutureTask
created: 2026-09-03
updated: 2026-09-03
tags: [元知识点, 线程池, C08]
---

# 元-FutureTask

`FutureTask` 是 `submit` 时套在原始 `Runnable` / `Callable` 外面的对象。  
它能把执行结果或异常记下来，调用方通过 `get()` 读取。

没有这层包装时，任务就是原始 `Runnable`，没有给调用方的返回值。
