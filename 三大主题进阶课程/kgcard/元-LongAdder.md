---
title: 元-LongAdder
created: 2026-09-05
updated: 2026-09-05
tags: [元知识点, LongAdder, C11]
---

# 元-LongAdder

`LongAdder` 把加法打散到 `base` 和多个 `Cell` 上。没碰车改 `base`；撞车了改自己那一格。  
`sum()` 把 `base` 与各 `Cell` 加起来。

拆的是单点 CAS 热点，不是时间窗口。
