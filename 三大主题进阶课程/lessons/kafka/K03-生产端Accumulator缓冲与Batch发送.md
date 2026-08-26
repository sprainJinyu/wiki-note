---
title: K03 - 生产端 Accumulator 缓冲与 Batch 发送
created: 2026-08-26
tags: [原理卡, Kafka, Producer, Batching, Accumulator, 性能调优]
lesson_id: K03
duration: 20min
---

# K03 · 生产端 Accumulator 缓冲与 Batch 发送

> **今天只带走这一条**：业务调用 `producer.send()` **不是立刻发网络请求**，而是把消息塞进内存缓冲区（`RecordAccumulator`），由后台独立的 `Sender` 线程批量打包发送。  
> **用时**：15–20 分钟。

---

## 1. 原理

```
[业务主线程]
     │ producer.send(record)
     ▼
[序列化 Serializer] ──→ [分区器 Partitioner]
                              │
                              ▼
               [RecordAccumulator 内存缓冲池 (默认 32MB)]
                 ├─ TopicA-Partition0: [ Batch 1 (16KB) ][ Batch 2 ]
                 └─ TopicA-Partition1: [ Batch 1 (16KB) ]
                              │
                              ▼ (达到 batch.size 或 超时 linger.ms)
[后台 Sender I/O 线程] ───────┴──────→ 网络批量发送至 Broker
```

1. **双线程解耦设计**：
   * **主线程**：负责序列化、计算目标分区，将消息追加到对应 `TopicPartition` 的批次（`ProducerBatch`）中，快速返回；
   * **Sender 线程**：后台轮询缓冲池，只要某个 Batch 攒满了（`batch.size`）或者等待超时了（`linger.ms`），就取出发送。
2. **吞吐调优双核心**：
   * `batch.size`：单个 Batch 最大字节数（默认 16KB）；
   * `linger.ms`：若 Batch 未满，Sender 愿意等多久（默认 0ms）。生产环境调大为 `5~20ms` 可大幅提升批量聚合度，吞吐量翻倍。

### 🔍 代码/配置嗅觉
```properties
# 生产端高吞吐抗压关键参数
batch.size=32768                   # 单批次 32KB
linger.ms=10                       # 最多等 10ms 凑批
buffer.memory=67108864             # 缓冲池 64MB (耗尽时 send 会阻塞 max.block.ms)
compression.type=lz4               # 开启客户端 LZ4 压缩，极大减轻带宽
```

---

## 2. 误区

❌ **「linger.ms 设置为 10ms，意味着每条消息至少有 10ms 延迟。」**  
只要并发足够高，消息迅速填满了 `batch.size`（32KB），Sender 会立即触发发送，根本不需要等满 10ms。

❌ **「send() 返回了 Future 就代表 Broker 已经落盘成功。」**  
`send()` 返回的只是消息成功进入本地 `RecordAccumulator` 的状态。必须等待 `Future.get()` 或在回调 `Callback` 中捕获到 `RecordMetadata`，才代表 Broker 真正 ACK 确认。

---

## 3. 自检

1. **如果业务突发流量过大，`buffer.memory` 缓冲池被占满了，后续的 `send()` 会怎样？**  
   *提示：主线程会被阻塞，阻塞时间超过 `max.block.ms`（默认 60s）后抛出 `TimeoutException`。*
2. **为什么单条消息体积特别大（如 1MB）时，Batching 批处理机制会失效？**  
   *提示：单条数据超过 `batch.size` 时，Producer 会为它单独创建一个不复用的内存块并立即发送，无法凑批。*

---

## 4. 资料

* `../../materials/kafka/01-Kafka核心底层原理与架构指南.md`
* `../../materials/kafka/raw/kafka-notes/`
