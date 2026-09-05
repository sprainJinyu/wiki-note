---
title: 元-ConcurrentHashMap桶
created: 2026-09-05
updated: 2026-09-05
tags: [元知识点, ConcurrentHashMap, C11]
---

# 元-ConcurrentHashMap桶

JDK 8 的 `ConcurrentHashMap` 底层是 `Node[] table`。一个下标是一个桶。

空槽：CAS 写入首节点。  
已有头：`synchronized` 锁这个桶的头节点，不锁整张表。

不同下标可以同时写。
