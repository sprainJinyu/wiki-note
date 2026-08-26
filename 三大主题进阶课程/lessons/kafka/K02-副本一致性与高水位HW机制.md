---
title: K02 - 副本一致性与高水位 HW 机制
created: 2026-08-26
tags: [原理卡, Kafka, ISR, 高水位, 数据一致性]
lesson_id: K02
duration: 20min
---

# K02 · 副本一致性与高水位 HW 机制

> **今天只带走这一条**：消费者**永远只能读到所有 ISR 副本中最小的 LEO（即 HW 高水位）**。`acks=-1/all` 配合 `min.insync.replicas=2` 是企业级消息零丢失的绝对底线。  
> **用时**：15–20 分钟。

---

## 1. 原理

```
              Leader 写入进度 (LEO: 6)
                     │
                     ▼
Leader:    [ 0 ][ 1 ][ 2 ][ 3 ][ 4 ][ 5 ][   ]
Follower1: [ 0 ][ 1 ][ 2 ][ 3 ][ 4 ][   ]       (LEO: 5)
Follower2: [ 0 ][ 1 ][ 2 ][ 3 ][   ]            (LEO: 4)
                     ▲
                     │
              High Watermark (HW: 4) ───【消费者最大可见边界】
```

1. **LEO (Log End Offset)**：各个副本当前**下一条待写入的位置**（最新数据偏移量 + 1）。
2. **HW (High Watermark - 高水位)**：所有 **ISR（同步副本）** 中最小的 LEO。
   * **可见性约束**：只有被 ISR 中所有副本都确认同步的数据（`Offset < HW`），才允许被消费者看到。
   * **目的**：防止 Leader 宕机时，新选出的 Leader 丢失已对外暴露的数据。
3. **ISR 动态维护**：Follower 只要在 `replica.lag.time.max.ms` 内向 Leader 发起 Fetch 请求即保存在 ISR 中；落后过久会被动态剔除。

### 🔍 代码/配置嗅觉
```properties
# 生产环境“零丢数据”黄金组合配置
acks=-1                            # 生产者等待全部 ISR 副本确认
min.insync.replicas=2              # ISR 中至少 2 个副本写入成功才算发成功
unclean.leader.election.enable=false # 禁止非 ISR 副本争当 Leader
```

---

## 2. 误区

❌ **「acks = all 表示消息必须写满集群里所有的副本。」**  
`acks=all` 只要求写满 **当前 ISR 集合里的活副本**。如果 ISR 里只剩 Leader 一个人，`acks=all` 会退化为 `acks=1`。必须搭配 `min.insync.replicas=2` 才能真正防丢。

❌ **「Leader 一收到消息，消费者就能立刻消费到。」**  
必须等该消息被 ISR 中所有 Follower 副本同步、Leader 推进 **HW 高水位** 之后，消费者才能 `poll()` 到。

---

## 3. 自检

1. **为什么禁止 `unclean.leader.election`（非同步副本选举）对金融级业务至关重要？**  
   *提示：如果允许落后的副本上位当 Leader，落后的那部分数据就会被永久覆盖丢弃。*
2. **如果 `min.insync.replicas=2`，但当前 ISR 里只剩 1 个副本，生产者发送 `acks=-1` 会发生什么？**  
   *提示：Broker 会直接拒绝写入，抛出 `NotEnoughReplicasException`，保证不产生数据隐患。*

---

## 4. 资料

* `../../materials/kafka/raw/01-Apache-Kafka官方核心设计与架构规范.md`
* `../../materials/kafka/01-Kafka核心底层原理与架构指南.md`
