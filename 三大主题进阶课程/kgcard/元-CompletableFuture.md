---
title: 元-CompletableFuture
created: 2026-09-14
updated: 2026-09-14
tags: [元知识点, CompletableFuture, C12]
---

# 元-CompletableFuture

`CompletableFuture` 是可再接线的异步结果盒子。  
请求线程负责 `supplyAsync` 分发、`allOf` / `thenCombine` 汇聚、`join` 拿值。  
传进去的 `Executor` 里的 Worker 负责跑任务。

不传池则跑在 `ForkJoinPool.commonPool()`。
