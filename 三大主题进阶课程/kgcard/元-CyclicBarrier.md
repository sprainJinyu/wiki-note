---
title: 元-CyclicBarrier
created: 2026-09-05
updated: 2026-09-05
tags: [元知识点, CyclicBarrier, C10]
---

# 元-CyclicBarrier

`CyclicBarrier` 是可循环的栅栏。先到的人等后到的人，每个到达者都要 `await()`，人齐一起过。

同一批线程既是当事人也是等待者。可多轮使用。
