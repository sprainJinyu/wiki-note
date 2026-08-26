---
title: K05 - Consumer Rebalance 重平衡与协作式分配
created: 2026-08-26
tags: [原理卡, Kafka, Rebalance, 重平衡, CooperativeStickyAssignor, 消费端调优]
lesson_id: K05
duration: 20min
---

# K05 · Consumer Rebalance 重平衡与协作式分配

> **今天只带走这一条**：Rebalance 是 Group 内重新分配 Partition 的机制。生产环境应首选**协作式分配器（`CooperativeStickyAssignor`）**，它只渐进迁移受影响的分区，彻底消除了旧版“全员停机（STW）”造成的消费抖动。  
> **用时**：15–20 分钟。

---

## 1. 原理

```
              【传统 Eager Rebalance (全局停顿)】
所有 Consumer ──(放弃所有 Partition)──→ 全员暂停消费 ──(重新分配)──→ 恢复消费
                                     ▲
                                     │ 【痛点：全组 STW 严重抖动】

          【现代 Cooperative Rebalance (协作式粘性)】
仅受影响的 Consumer ──(局部撤销与迁移)──→ 未变动的 Partition 持续正常消费！
```

1. **Rebalance 触发条件**：
   * 组成员变动（新实例启动、旧实例下线或崩溃）；
   * 心跳超时（`session.timeout.ms`）；
   * **单批消息处理耗时过长**，超过了 `max.poll.interval.ms`（Broker 误以为 Consumer 死了）。
2. **两阶段协调协议**：
   * **JoinGroup**：所有 Consumer 向服务端 **GroupCoordinator** 发起请求，选举 Leader Consumer；
   * **SyncGroup**：Leader Consumer 计算好分配方案交由 Coordinator 广播分发给所有成员。
3. **协作式再均衡（Cooperative Sticky）**：
   * 不要求所有消费者先释放手中所有的分区，而是**保持未变动分区的连续消费**，只对发生迁移的少量分区进行两轮渐进式交接。

### 🔍 代码/配置嗅觉
```properties
# 避免线上“假死 Rebalance 恶性循环”的关键配置
partition.assignment.strategy=org.apache.kafka.clients.consumer.CooperativeStickyAssignor
max.poll.interval.ms=300000        # 单次 poll 后的业务处理最大耗时 (5分钟)
max.poll.records=500               # 单次 poll 拉取最大条数 (处理慢时调小)
session.timeout.ms=45000           # 心跳丢失超时时间
```

---

## 2. 误区

❌ **「线上 Rebalance 一定是因为消费者机器宕机了。」**  
最常见的 Rebalance 往往是因为**业务处理耗时太长（超过 `max.poll.interval.ms`）**，Consumer 来不及调用下一次 `poll()`，被 Broker 误判为死掉而踢出组。

❌ **「Rebalance 期间所有消费者都不能干活。」**  
在启用了 `CooperativeStickyAssignor` 后，只有被重新指派的分区会短暂暂停，其余绝大部分分区全程不中断消费。

---

## 3. 自检

1. **如果下游调用外部第三方接口偶尔超时（长达 30 秒），导致频繁触发 Rebalance，该如何调优？**  
   *提示：调小 `max.poll.records`（减少单次拉取量）或调大 `max.poll.interval.ms`。*
2. **为什么 GroupCoordinator 选在 Broker 端，而具体的分区分配算法却在 Consumer 客户端计算？**  
   *提示：解耦 Broker 与分配算法，让客户端可以自由定制分配策略（如自定义机架感知、权重分配），无需重启 Broker。*

---

## 4. 资料

* `../../materials/kafka/01-Kafka核心底层原理与架构指南.md`
* `../../materials/kafka/raw/kafka-notes/`
