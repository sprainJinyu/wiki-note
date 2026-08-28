---
title: C09 - ThreadLocal 内存模型与隐患
created: 2026-08-28
updated: 2026-08-28
tags: [原理卡, 并发编程, ThreadLocal, 内存泄漏]
lesson_id: C09
duration: 20min
---

# C09 · ThreadLocal：Key 是弱引用，用完必 remove()

> **今天只带走这一条**：`ThreadLocalMap` 的 Key 是弱引用（GC 时会被回收变 null），但 Value 是强引用；在线程池复用线程的场景下，**用完必须显式 `remove()`**，否则必造成内存泄漏与跨请求数据污染。
> **用时**：15–20 分钟。

---

## 1. 原理

### 引用链条与内存模型

每一个 `Thread` 对象内部都有一个 `ThreadLocalMap` 变量，其底层数组存储 `Entry`：

```
Thread (线程对象)
  └── threadLocals
        └── ThreadLocalMap
              └── Entry[] table
                    ├── Entry[0] ──> Key (WeakReference 指向 ThreadLocal 对象)
                    │                Value (强引用指向实际业务对象 Object)
                    └── Entry[1] ...
```

```
【强引用】Thread ───────────────> ThreadLocalMap ──────> Entry
                                                           │ (强引用)
【弱引用】ThreadLocal 对象 <······ (Key: 弱引用)           └──> Value 对象
```

1. **为什么 Key 是弱引用？**  
   如果 Key 是强引用，哪怕业务代码将外部的 `tl = null`，只要线程还在运行，`ThreadLocalMap` 里的 Entry 就会强引用着 ThreadLocal 对象，导致其永远无法被垃圾回收。设计为弱引用，在下次 GC 时 Key 会被自动置为 `null`。
2. **为什么还会内存泄漏？**  
   Key 虽然被回收变 `null`，但 `Entry.value` 依然被当前 `Thread` 强引用链咬死。如果线程来自线程池（生命周期极长），这个 `Value` 就永远无法被回收，直到整个 JVM 内存耗尽。

---

## 2. 误区

**「Key 是弱引用，所以 ThreadLocal 会自动帮我们清理内存，不需要管。」**  
**大错！** 弱引用只保证了 ThreadLocal 对象本身能被回收，但对于 `Entry.value` 强引用无能为力。只有在后续调用 `set()` / `get()` / `remove()` 命中 hash 冲突触发“探测式清理”时才会顺便清理 null key，这是被动修补，无法保证不发生 OOM。

**「每个 Web 请求结束后不 `remove()` 也没事，Spring 会在请求结束时回收对象。」**  
**致命陷阱！** Tomcat/Undertow 处理 HTTP 请求使用的是工作线程池。如果不调用 `remove()`，下一个 HTTP 请求被同一个工作线程处理时，会直接读到上一个用户残留的 UserId / 租户信息，造成**灾难性的用户身份串号与安全事故**。

---

## 3. 规避规范（标准代码嗅觉）

```java
public void doBusiness(UserContext ctx) {
    try {
        USER_HOLDER.set(ctx);
        processOrder(); // 业务处理
    } finally {
        USER_HOLDER.remove(); // 必须在 finally 中彻底清除！
    }
}
```

---

## 4. 自检

1. **`ThreadLocal` 变量本身通常为什么要声明为 `private static final`？**  
   *提示：`static` 保证全局只有一份单例实例，避免每次 new 产生多余的 ThreadLocal 对象和 Entry 浪费；`final` 防止引用被恶意修改。*
2. **在 Spring Boot 拦截器（Interceptor）中存入用户信息，应该在哪个生命周期方法中执行 `remove()`？**  
   *提示：在 `afterCompletion()` 中，无论请求成功还是抛出异常，都会被最终触发执行。*

---

## 5. 资料

* `../../materials/concurrency/redspider-concurrent/article/03/21.md`
* 《深入理解 Java 虚拟机》第 3 章（对象的引用类型）
