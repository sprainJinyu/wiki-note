---
title: L05 - RST 与 Connection Reset
created: 2026-08-17
updated: 2026-08-21
tags: [原理卡, 计算机网络, RST, 超时]
lesson_id: L05
duration: 20min
---

# L05 · RST 是拆连接；Read Timeout 是本端等不及

> **今天只带走这一条**：RST 表示「这条连接立刻作废，不走四次挥手」；Read Timeout 表示「对端可能还在，是我这边等响应超时了」。两者的排查方向相反。
> **用时**：15–20 分钟。

---

## 1. 原理

正常关闭走 FIN。RST 是硬拆：收到就丢掉这条连接的状态，不再握手、不再挥手。

常见来源（抓包里过滤器：`tcp.flags.reset == 1`）：

| 场景 | 往往看到的 Java 异常 | 先查什么 |
| :--- | :--- | :--- |
| 端口没人听 / 进程没起来 | `ConnectException: Connection refused` | 对端进程、端口、防火墙 |
| 对端对半截数据直接 close | `Connection reset` / `reset by peer` | 对端是否读完就关、是否崩了 |
| 中间设备闲置超时踢连接 | `Connection reset`（空闲后再发才出现） | 网关 idle timeout 是否短于连接池空闲时间 |

和超时分开记：

* **Read Timeout**：请求已经发出去，对端也可能处理完了，只是响应没在时限内回到本端。查对端慢、本端线程、GC，**不要默认「没处理」**。
* **Connect Timeout**：连还没建起来。查网络、对端没听、防火墙。
* **RST / Connection Reset**：链路上有人发了 RST。查端口、对端崩溃、闲置被踢。

---

## 2. 误区

**「报了 Read Timeout，重试一次就行。」**  
超时只说明本端没等到响应。对端可能已经写完库、扣完款。非幂等的写不能无条件重试。

**「Connection Reset 和 Read Timeout 都是网络不稳，处理一样。」**  
一个是连接被拆，一个是等响应超时。过滤器、日志、下一步完全不同。

可重试的通常是「连接没建起来、或明确被踢后换新连接」这类传输失败；业务校验失败（参数非法）重试没有意义。这是分类原则，不是某段分类器代码。

---

## 3. 自检

1. **Wireshark 里只看 RST，过滤表达式是什么？**
   *提示：`tcp.flags.reset == 1`。*
2. **调用一个非幂等写接口出现 `SocketTimeoutException: Read timed out`，直接再发一次会怎样？**
   *提示：对端可能已经成功，再发一次就是重复执行。*

---

## 4. 资料

* `../materials/network/wireshark-tcp-troubleshooting-guide.md`
* 林沛满《Wireshark 网络分析就这么简单》中 RST 相关章节
