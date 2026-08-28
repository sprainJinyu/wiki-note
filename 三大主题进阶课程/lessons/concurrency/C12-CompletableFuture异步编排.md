---
title: C12 - CompletableFuture 异步编排与超时熔断
created: 2026-08-28
updated: 2026-08-28
tags: [原理卡, 并发编程, CompletableFuture, 异步编排, 性能优化]
lesson_id: C12
duration: 20min
---

# C12 · 异步编排：多下游并行聚合，绝不用默认池

> **今天只带走这一条**：`CompletableFuture` 解决多下游异步并行聚合（场景 A）；`supplyAsync` 必须显式传入业务自定义线程池（严禁共用默认 `ForkJoinPool.commonPool()`）；多任务并行汇聚用 `allOf` / `thenCombine`；末端必须用 `exceptionally` 或 `handle` 进行异常降级与超时熔断，防止异步线程静默挂死。
> **用时**：15–20 分钟。

---

## 1. 原理

### 一、 核心编排模式与最佳实践

```
                          ┌── supplyAsync(查用户信息, ioPool) ──┐
客户端请求 ──> 异步发起 ──┼── supplyAsync(查用户积分, ioPool) ──┼──> allOf / thenCombine ──> 聚合结果返回
                          └── supplyAsync(查风控评分, ioPool) ──┘
                                        │
                                        └── .orTimeout(500, TimeUnit.MILLISECONDS)
                                        └── .exceptionally(ex -> 降级默认值)
```

```java
// 1. 显式指定独立 I/O 线程池
CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> userService.get(uid), ioThreadPool);
CompletableFuture<Score> scoreFuture = CompletableFuture.supplyAsync(() -> scoreService.get(uid), ioThreadPool);

// 2. 双任务结果汇聚
CompletableFuture<UserVO> voFuture = userFuture.thenCombine(scoreFuture, (user, score) -> new UserVO(user, score))
        .orTimeout(300, TimeUnit.MILLISECONDS) // 超时控制 (Java 9+)
        .exceptionally(ex -> UserVO.fallback()); // 异常兜底降级
```

---

## 2. 误区

**「直接调用 `CompletableFuture.supplyAsync(supplier)` 不传 Executor 最简洁省事。」**  
**高危生产陷阱！** 不传线程池时，默认使用的是 JVM 全局唯一的 `ForkJoinPool.commonPool()`。当某个外部接口变慢或网络阻塞时，整个 JVM 的所有异步操作（包括 Stream 并行流）全会被拖死瘫痪。

**「`CompletableFuture.allOf(f1, f2, f3)` 执行完会直接返回聚合后的数据结果。」**  
**认知偏差！** `allOf` 返回的是 `CompletableFuture<Void>`，它的作用仅仅是**等待所有任务完成**。获取各个结果必须在 `allOf.join()` 后分别从 `f1.join()`、`f2.join()` 中提取。

---

## 3. 自检

1. **`CompletableFuture.join()` 和 `Future.get()` 的核心区别是什么？**  
   *提示：`join()` 抛出非受检异常 `CompletionException`，Lambda 表达式编写更流畅；`get()` 抛出受检异常 `InterruptedException` 和 `ExecutionException`，必须显式 try-catch。*
2. **当需要并发调用 10 个 RPC 接口，只要有任意一个返回就立即响应前端，应该使用哪个方法？**  
   *提示：`CompletableFuture.anyOf(f1, f2, ...)`。*

---

## 4. 资料

* 《Java 8 实战》第 11 章（CompletableFuture：组合式异步编程）
* Oracle Java SE API 文档：`java.util.concurrent.CompletableFuture`
