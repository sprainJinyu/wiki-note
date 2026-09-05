---
title: 元-CountDownLatch单向倒数
created: 2026-08-27
updated: 2026-09-05
tags: [元知识点, AQS, CountDownLatch, C10]
---

# 元-CountDownLatch单向倒数

`CountDownLatch` 是一次性门闪。构造时设定倒数，工作线程干完 `countDown()` 减 1，可以离开。  
等待线程在 `await()` 上看是否到 0。到 0 门开，只放行等待者。

倒数与等待可以是两类角色。不能 `reset`。
