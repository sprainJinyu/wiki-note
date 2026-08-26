---
title: K01 - Partition 与 Segment 存储机制
created: 2026-08-26
tags: [原理卡, Kafka, 存储机制, 零拷贝, PageCache]
lesson_id: K01
duration: 20min
---

# K01 · Partition 与 Segment 存储机制

> **今天只带走这一条**：Kafka 性能极高的秘密不是“纯内存”，而是**磁盘顺序追加写（PageCache）** + **网络零拷贝（sendfile）**，数据自始至终不经过 JVM 堆内存。  
> **用时**：15–20 分钟。

---

## 1. 原理

```
Topic: order-events (2 Partitions)
  │
  ├─ Partition 0 目录 (topic-name_0/)
  │   ├── 00000000000000000000.log         ← 实际消息数据 (顺序追加写入)
  │   ├── 00000000000000000000.index       ← Offset 物理位置稀疏索引
  │   └── 00000000000000000000.timeindex   ← 时间戳索引
  │
  └─ Partition 1 目录 (topic-name_1/)
```

1. **磁盘顺序写（Sequential Write）**：
   * 每一个 Partition 对应磁盘上的一个文件夹，内部切分成多个 **Segment**（默认每个最大 1GB）；
   * 消息只能追加到当前 active segment 的 `.log` 文件末尾（Append-only），顺序 I/O 的吞吐量可达数百 MB/s，逼近内存速度。
2. **PageCache 内核页缓存**：
   * 写入操作直接写入操作系统的 PageCache，由 OS 后台脏页回写磁盘，规避 JVM GC 停顿与堆对象膨胀。
3. **网络零拷贝（Zero-Copy / `sendfile`）**：
   * 消费者拉取数据时，数据通过 Linux `sendfile()` 系统调用，直接从 PageCache 经 DMA 拷贝送入网卡缓冲区，**完全不进入 JVM 进程空间**。

### 🔍 代码/配置嗅觉
```properties
# 核心存储配置：每个 Segment 切片大小与日志保留时间
log.segment.bytes=1073741824       # 1GB 一个 Segment
log.cleanup.policy=delete          # 超期物理删除
log.retention.hours=168            # 消息保留 7 天 (与消费进度解耦)
```

---

## 2. 误区

❌ **「Kafka 速度快是因为消息全放在 JVM 内存里。」**  
Kafka 恰恰极力避免把数据放进 JVM 堆内存。它把缓存托管给操作系统的 PageCache，Broker 进程重启缓存也不会丢。

❌ **「Kafka 删除消息像数据库一样执行 Delete。」**  
Kafka 的消息是不可变的。删除是以整个过期的 **Segment 文件** 为单位直接从文件系统 unlink，零额外寻址开销。

---

## 3. 自检

1. **为什么消费者拉取消息时，Broker CPU 占用率通常非常低？**  
   *提示：Zero-Copy（`sendfile`）让数据拷贝在内核态由 DMA 完成，不经过 CPU 和 JVM 堆。*
2. **`.index` 稀疏索引为什么不为每条消息都建立索引记录？**  
   *提示：稀疏索引（默认每隔 4KB 记一条）极大节省了内存，先二分查索引找到最近点，再在 `.log` 中顺序扫描少量字节即可。*

---

## 4. 资料

* `../../materials/kafka/raw/01-Apache-Kafka官方核心设计与架构规范.md`
* `../../materials/kafka/01-Kafka核心底层原理与架构指南.md`
