---
title: 元-await后的while检查
created: 2026-08-26
updated: 2026-08-26
tags: [元知识点, Condition, while]
---

# 元-await后的while检查

调用 `await()` 后被唤醒，执行流是从 `await()` 的下一行继续。  
因为 `await()` 写在 `while` 循环体里，所以会再次命中条件判断。  

如果条件已经不满足（被别人抢先改了状态），就再次进入 `await()`。  
如果改成 `if`，就没有「再检查一次」的机会，可能在条件不满足时继续往下执行而出错。  

真正让线程挂起的永远是 `await()` 本身，`while` 只负责被唤醒后强制再检查。
