---
title: 元-Semaphore许可
created: 2026-08-27
updated: 2026-08-27
tags: [元知识点, AQS, Semaphore]
---

# 元-Semaphore许可

信号量的 `state` 是还能发出去的许可数。  
变成 0 意味着数量发完了，后来的人排队。  
有人释放才能把数量返还回来，许可可以反复用。
