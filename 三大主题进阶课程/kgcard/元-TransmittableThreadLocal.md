---
title: 元-TransmittableThreadLocal
created: 2026-09-21
updated: 2026-09-21
tags: [元知识点, TTL, C13]
---

# 元-TransmittableThreadLocal

`TransmittableThreadLocal` 把上下文绑在任务提交上，不绑在 `new Thread()` 上。

提交时在请求线程 `capture` 快照；Worker 执行前 `replay` 进当前线程的地图；`run` 结束 `restore` 回执行前的样子。
