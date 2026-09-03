---
title: 元-RejectedExecutionHandler
created: 2026-09-03
updated: 2026-09-03
tags: [元知识点, 线程池, C08]
---

# 元-RejectedExecutionHandler

`RejectedExecutionHandler` 是线程池在第 4 段的处理接口：`workQueue.offer` 失败，且工作线程数已到 `maximumPoolSize`（或建救急线程也失败）时被调用。

它挂在线程池上，不是挂在队列上。
