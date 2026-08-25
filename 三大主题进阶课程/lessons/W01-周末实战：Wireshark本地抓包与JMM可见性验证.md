---
title: W01 - 可选：可见性实验
created: 2026-08-17
updated: 2026-08-21
tags: [可选验证, JMM]
lesson_id: W01
duration: 30min
---

# W01 · 可选验证：无同步时可能看不见

> 不是作业，不是生产改造。有一整段空闲、想把 L01 钉住再做。通勤不要做。
> **只验证一条**：普通 `boolean` 在没有同步时，别的线程可能一直看不见这次写。
> **失败也算有收获**：实验经常「看起来没问题」。不复现 ≠ 没有可见性问题。

---

## 做什么

任意本机 Java 项目里丢一个测试类即可，**不要**放进 `member-user-center`。

```java
public class VisibilityDemo {
    // 实验 A：保持注释，看 worker 是否在主线程写入后仍不退出
    // 实验 B：加上 volatile，看是否很快退出
    private static /* volatile */ boolean stop = false;

    public static void main(String[] args) throws Exception {
        Thread worker = new Thread(() -> {
            while (!stop) {
                // 空转。不要在这里打印或加锁，否则可能「顺便」看见新值
            }
            System.out.println("worker 看见 stop=true，退出");
        });
        worker.start();
        Thread.sleep(1000);
        stop = true;
        worker.join(3000);
        if (worker.isAlive()) {
            System.out.println("3 秒后仍在转：这次复现了可见性延迟");
            // 不要指望 interrupt() 能打断空转。只能停 JVM，或给 stop 加 volatile 再跑实验 B。
            System.exit(0);
        } else {
            System.out.println("这次看见了。不代表没有 volatile 也安全，换机器/换 JIT 可能相反。");
        }
    }
}
```

JIT 若没把 `stop` 的读提出循环，实验 A 也会正常退出。所以：

* 复现了：L01 那句话有了一个例子。
* 没复现：L01 那句话仍然成立，只是这次硬件/JIT 刚好让你看见了。

不要把这个 Demo 当单元测试去「断言必挂」。

---

## 有余力再做（仍可选）

验证 L04/L05：开 Wireshark，过滤 `tcp.flags.reset == 1`，访问一个没人听的端口，看 RST。和 JMM 实验无关，做不完就算了。

---

## 不做

* 不写 Wiki 交差
* 不改 `retrytask`
* 不把「抓包 + JMM + 两张卡片」绑在同一个下午
