---
title: 原理卡进度看板
created: 2026-08-17
updated: 2026-09-03
tags: [学习进度, 原理卡]
---

# 原理卡进度看板

> **勾选标准**：能用自己的话复述该卡「今天只带走这一条」。不记代码、不记 Wiki。
> **学习原则**：**专题单通（推荐先连续通关并发 〈 再通关网络/Kafka 〈 最后 DDD）**。
> 待写卡（无正文）勿勾。用法见 [[00-课程总览与大纲]]。

* **总规模**：33 张原理卡 + 1 个可选验证（W01）
* **已就绪正文**：并发 15 篇（C01~C15 全就绪）+ Kafka 5 篇（K01~K05）+ 网络 2 篇（N01~N02）+ 可选 W01

---

## 🧵 并发专题卡组（15 张 + 可选 W01 · 全部正文就绪）

### 阶段 1：底层基石（已 100% 通关）

- [x] **C01** [[lessons/concurrency/C01-JMM内存模型与主内存工作内存|没有同步，就不保证可见]] `[日期: 2026-08-25]` `[复述: 能]`
- [x] **C02** [[lessons/concurrency/C02-volatile底层原理与CPU缓存一致性|volatile 可见、有序、非原子]] `[日期: 2026-08-25]` `[复述: 能]`
- [x] **C03** [[lessons/concurrency/C03-指令重排序与Happens-Before规则|用 happens-before 思考可见性]] `[日期: 2026-08-25]` `[复述: 能]`
- [x] **C04** [[lessons/concurrency/C04-显式锁ReentrantLock与synchronized对比|两种锁差在中断、超时、条件、公平]] `[日期: 2026-08-26]` `[复述: 能]`
- [x] **C05** [[lessons/concurrency/C05-AQS核心原理：CLH队列与Node状态流转|AQS 三要素]] `[日期: 2026-08-26]` `[复述: 能]`
- [x] **C06** [[lessons/concurrency/C06-CAS无锁算法、Unsafe与ABA问题解决|CAS 与 ABA]] `[日期: 2026-08-27]` `[复述: 能]`
- [ ] **W01**（可选轻实验）[[lessons/concurrency/W01-JMM可见性实验与验证|JMM 可见性验证]] `[做了 / 跳过]`

### 阶段 2：JDK 核心工具储备（路线 1 筑基 · 已就绪）

- [x] **C07** [[lessons/concurrency/C07-线程池核心机制与排队本质|进池顺序：core → 队列 → max]] `[日期: 2026-08-28]` `[复述: 能]`
- [x] **C08** [[lessons/concurrency/C08-阻塞队列与拒绝策略|阻塞队列选型与 execute/submit 异常机制]] `[日期: 2026-09-03]` `[复述: 能]`
- [ ] **C09** [[lessons/concurrency/C09-ThreadLocal内存模型与隐患|ThreadLocalMap 弱引用 Key 强引用 Value]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **C10** [[lessons/concurrency/C10-并发协作三剑客|CountDownLatch vs CyclicBarrier vs Semaphore]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **C11** [[lessons/concurrency/C11-ConcurrentHashMap与LongAdder|ConcurrentHashMap 锁桶头与 LongAdder 分段累加]] `[日期: ____-__-__]` `[复述: 能 / 不能]`

### 阶段 3：现代实战编排与掘查（路线 2 贯通 · 聚焦场景 ACD · 已就绪）

- [ ] **C12** [[lessons/concurrency/C12-CompletableFuture异步编排|CompletableFuture 异步编排、多下游汇聚与异常降级]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **C13** [[lessons/concurrency/C13-微服务上下文跨线程传递|微服务上下文跨线程传递：阿里 TTL 抓取与回放]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **C14** [[lessons/concurrency/C14-弹性容错与重试机制设计|弹性容错与重试机制：指数退避 Jitter 与线程池隔离]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **C15** [[lessons/concurrency/C15-线上并发故障与jstack定位实战|线上并发故障掘查：top -Hp 与 jstack 快速分析]] `[日期: ____-__-__]` `[复述: 能 / 不能]`

---

## 📬 Kafka 专题卡组（10 张 · 2周）

### 第 1 周：Kafka 核心底层机制（已就绪 5 张）

- [ ] **K01** [[lessons/kafka/K01-Partition与Segment存储机制|顺序写 PageCache + sendfile 零拷贝]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **K02** [[lessons/kafka/K02-副本一致性与高水位HW机制|HW 高水位边界与 acks=-1 零丢底线]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **K03** [[lessons/kafka/K03-生产端Accumulator缓冲与Batch发送|Accumulator 缓冲池与 Batching 批处理]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **K04** [[lessons/kafka/K04-ConsumerGroup分组与单分区独占模型|组内单分区独占与多余实例空闲]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **K05** [[lessons/kafka/K05-Consumer-Rebalance重平衡与协作式分配|CooperativeSticky 协作式分配消除 STW]] `[日期: ____-__-__]` `[复述: 能 / 不能]`

### 第 2 周：Spring Boot 装配与多环境动态隔离（待写 5 张，无正文勿勾）

- **K06** `KafkaTemplate` 轻量包装与单例线程安全
- **K07** `@KafkaListener` BPP 阶段扫描与 SpEL 动态解析
- **K08** 监听容器 `SmartLifecycle` 与后台轮询工作线程
- **K09** 蓝绿环境生产端 `ProducerInterceptor` 动态改写 Topic
- **K10** 消费端 Topic 与 GroupId 必须同步隔离的底层根因

---

## 🌐 网络专题卡组（4 张）

已就绪：

- [ ] **N01** [[lessons/network/N01-TCP三次握手四次挥手与状态机|握手与挥手]] `[日期: ____-__-__]` `[复述: 能 / 不能]`
- [ ] **N02** [[lessons/network/N02-TCP异常判定：RST报文与Connection-Reset掘查|RST vs 读超时]] `[日期: ____-__-__]` `[复述: 能 / 不能]`

待写（按需产出，无正文勿勾）：

- **N03** 超时重传 vs 快速重传；应用层 timeout ≠ RTO
- **N04** 零窗口是背压，不是断线

---

## 🏰 DDD 专题卡组（4 张）

待写（按需产出，无正文勿勾）：

- **D01** 界限上下文是语言边界，不是包名
- **D02** 实体靠身份，值对象靠结构
- **D03** 一次事务只改一个聚合
- **D04** 仓储按聚合存取，不是按表

---

## 📝 复述时卡住的点

*读完复述不出来，把那句话记在这里，下一张卡开始前先看：*

* C07：扩容的是工作线程数，不是把 corePoolSize 改大；救急线程是非核心线程。
* C08：`SynchronousQueue` ≠ AQS 同步队列。同步队列等的是任务队列那把锁；`SynchronousQueue` 容量为 0，直接交接。
