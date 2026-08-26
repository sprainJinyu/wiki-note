---
title: K04 - Consumer Group 分组与单分区独占模型
created: 2026-08-26
tags: [原理卡, Kafka, ConsumerGroup, 分区独占, 消费模型, 水平扩展]
lesson_id: K04
duration: 20min
---

# K04 · Consumer Group 分组与单分区独占模型

> **今天只带走这一条**：同一个 Consumer Group 内，**一个 Partition 在同一时刻只能分配给某一个 Consumer 独占消费**。Consumer 实例数超过 Partition 总数时，多出的实例将处于**完全空闲（Idle）**状态。  
> **用时**：15–20 分钟。

---

## 1. 原理

```
Topic: order-events (3 Partitions)
┌───────────────┬───────────────┬───────────────┐
│  Partition 0  │  Partition 1  │  Partition 2  │
└───────┬───────┴───────┬───────┴───────┬───────┘
        │               │               │
        ▼               ▼               ▼
┌───────────────┬───────────────┬───────────────┐  ┌───────────────┐
│  Consumer C1  │  Consumer C2  │  Consumer C3  │  │  Consumer C4  │
└───────────────┴───────────────┴───────────────┘  └───────┬───────┘
  Consumer Group A (共 4 个实例)                            │
                                                   【纯空闲 (Idle)】
```

1. **单 Partition 独占性**：
   * 为了保证**单个 Partition 内部消息消费的严格顺序性**，Kafka 规定同一 Consumer Group 内不允许两个消费者并发读取同一个 Partition。
2. **水平扩展上限（Scalability Limit）**：
   * **Topic 的 Partition 数量决定了单个 Consumer Group 的最大并行度**。
   * 如果 Topic 只有 3 个分区，部署 4 个 Pod，第 4 个 Pod 将永远分不到数据（浪费算力）。想要提升单组消费并发，必须**扩容 Topic 的分区数**。
3. **多组广播模型**：
   * 不同的 Consumer Group 彼此完全独立。同一个 Topic 的同一条消息会被广播分发给所有的 Group。

### 🔍 代码/配置嗅觉
```java
// Spring-Kafka 设置单实例内的并发 Consumer 线程数 (对应子容器数)
@Bean
public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    factory.setConcurrency(3); // 相当于该应用内部启动 3 个 Consumer 实例
    return factory;
}
```

---

## 2. 误区

❌ **「线上消息积压了，我把 Consumer 实例数从 5 个扩容到 20 个，就能 4 倍速消费。」**  
如果该 Topic 一共只有 5 个 Partition，扩容到 20 个实例毫无用处（15 个实例会处于 Idle 空转）。必须先调大 Partition 数量。

❌ **「不同 Consumer Group 消费同一个 Topic 会互相抢数据。」**  
不会。每个 Group 拥有独立的 Offset 追踪指针，互不干扰，相当于发布-订阅广播。

---

## 3. 自检

1. **如果一个 Topic 有 4 个分区，组内只有 2 个 Consumer，分区会如何分配？**  
   *提示：每个 Consumer 会负责消费 2 个分区（如 C1 负责 P0/P1，C2 负责 P2/P3）。*
2. **为什么单个 Partition 内部不能让两个 Consumer 线程并发拉取并消费？**  
   *提示：一旦两个线程并发读取同一个分区，消息处理的先后顺序和 Offset 提交顺序将彻底打乱。*

---

## 4. 资料

* `../../materials/kafka/01-Kafka核心底层原理与架构指南.md`
* `../../materials/kafka/raw/kafka-notes/`
