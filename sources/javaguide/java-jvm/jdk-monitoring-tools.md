# JDK 监控和故障处理工具总结

> **来源站点**：JavaGuide（javaguide.cn）
> **原文链接**：https://javaguide.cn/java/jvm/jdk-monitoring-and-troubleshooting-tools.html
> **抓取日期**：2026-10-18
> **所属知识域**：域5 生产运维与可观测性
> **板块**：JVM · 监控诊断工具

---

## JDK 命令行工具（bin 目录）

| 工具 | 全称 | 用途 |
|------|------|------|
| `jps` | JVM Process Status | 类似 `ps`，查看所有 Java 进程的启动类、JVM 参数 |
| `jstat` | JVM Statistics Monitoring Tool | 收集 HotSpot 运行数据（类/内存/GC/JIT），无 GUI 服务器首选 |
| `jinfo` | Configuration Info for Java | 实时查看和调整虚拟机各项参数 |
| `jmap` | Memory Map for Java | 生成堆转储快照（dump） |
| `jhat` | JVM Heap Dump Browser | 分析 heapdump（建 HTTP 服务浏览器查看），**JDK9 已移除** |
| `jstack` | Stack Trace for Java | 生成当前时刻线程快照 |

### jps

- `jps`：主类名称 + LVMID；
- `jps -q`：只输出 LVMID；
- `jps -l`：主类全名 / Jar 路径；
- `jps -v`：JVM 启动参数；
- `jps -m`：传给 main() 的参数。

### jstat

格式：`jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]`

例：`jstat -gc -h3 31736 1000 10` = 分析 GC 情况，每 1000ms 一次共 10 次，每 3 行打印指标头。

常用 option：`-class`（类加载）、`-compiler`（JIT）、`-gc`（堆与 GC）、`-gccapacity`（各区容量）、`-gcnew`/`-gcnewcapacity`（新生代）、`-gcold`/`-gcoldcapacity`（老年代，现代 JDK 含元空间）、`-gcutil`（GC 统计）。`-gcpermcapacity`（永久代）JDK8 后不存在，改用 `-gcmetacapacity`。加 `-t` 输出 Timestamp 列。

### jinfo

- `jinfo vmid`：全部参数和系统属性；
- `jinfo -flag name vmid`：查具体参数值（如 `MaxHeapSize`）；
- `jinfo -flag [+|-]name vmid`：动态开关可管理参数（不是所有参数都支持运行时改）。

JDK 9+ GC 日志用统一日志 `-Xlog`，运行时用 `jcmd VM.log` 调整。

### jmap

- `jmap -dump:format=b,file=heap.hprof <vmid>`：堆转储；
- 也可查终结器队列、Java 堆、类加载器；
- 不靠 jmap 也能拿 dump：`-XX:+HeapDumpOnOutOfMemoryError` OOM 后自动生成；
- Linux/macOS `kill -3 <pid>` 发 SIGQUIT 得的是**线程转储**（打印到标准错误流），不是堆转储；
- 部分功能受 OS 和目标进程附加权限限制。

### jhat

- `jhat heap.hprof`：起 HTTP 服务（默认 7000 端口）浏览器分析；
- **JDK9 移除（JEP 241）**，官方推荐替代：MAT、VisualVM。

### jstack

- 生成当前时刻线程快照（每条线程正在执行的方法堆栈集合）；
- 用途：定位线程长时间停顿——死锁、死循环、等外部资源；
- 直接输出 `Found one Java-level deadlock` 及等待/持有关系（monitor 地址 + 对象 + 线程栈）。

## JDK 可视化分析工具

### JConsole（基于 JMX）

- 监视本地/远程 Java 进程内存（堆/非堆，细化到 eden/survivor）、线程（可视化 jstack + **检测死锁按钮**）；
- 远程连接参数：

```properties
-Djava.rmi.server.hostname=外网IP
-Dcom.sun.management.jmxremote.port=60001
-Dcom.sun.management.jmxremote.authenticate=false   # 关闭认证
-Dcom.sun.management.jmxremote.ssl=false
```

⚠️ 关认证 + 关 SSL 只适合受控开发/隔离测试环境——生产必须启用认证与安全传输 + 网络访问控制。

- 「执行 GC」按钮语义类似 `System.gc()`，JVM 不保证一定执行；
- GC 术语：Minor GC（新生代，频繁快速）/ Major GC 或 Old GC（老年代，各资料用法不一）/ Full GC（整堆收集，停顿长）——读日志结合收集器和事件判断。

### VisualVM（All-in-One）

- 基于 NetBeans，插件扩展；
- 能力：进程配置环境（jps/jinfo）、CPU/GC/堆/方法区/线程监视（jstat/jstack）、dump 及堆分析（jmap/jhat）、方法级性能分析、离线快照；
- Oracle JDK 9 起不再随 JDK 捆绑，需单独下载；生产启用性能分析要评估采样/插桩开销。

### MAT（Memory Analyzer Tool）

- 离线堆转储分析，OOM/GC 问题首选；
- 看占用最大对象、Dominator Tree、引用链、类加载器。
