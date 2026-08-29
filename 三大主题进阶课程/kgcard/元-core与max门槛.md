---
title: 元-core与max门槛
created: 2026-08-29
updated: 2026-08-29
tags: [元知识点, 线程池, C07]
---

# 元-core与max门槛

`corePoolSize` 和 `maximumPoolSize` 是人数门槛，是配置，提交任务时不会把 core 这个数改大。

当前活着的是 `workerCount`（工作线程数）。  
没满 core 就按人数继续开 Worker；满了 core 才入队；队满且人数还小于 max 再开到 max。

## 巧思 / 细则

默认懒创建：池子 `new` 出来时一条 Worker 都还没有。  
没满 core 时只看个数、不看忙闲：旁边已有人在 `take` 上空等，新任务仍会再开一条。
