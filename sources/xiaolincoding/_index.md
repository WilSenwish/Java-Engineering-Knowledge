# 小林coding（xiaolincoding.com）站点目录

> **来源**：https://xiaolincoding.com/
> **定位**：从图解计算机到后端面试全攻略——"让天下没有难懂的八股文"
> **抓取日期**：2026-10-18
> **知识库角色**：**原理可视化补充**。图解网络/系统/MySQL/Redis 四大系列全部手绘图解，作域1/2/3 的图解主来源
> **规模**：1500 张图 + 100 万字；面试题集合 1600+ 题、57 万字

## 板块总览与知识域映射

| 专题 | 站点路径 | 规模 | 对应知识域 |
|------|----------|------|------------|
| 图解网络 | /network/ | 20W 字 + 500 图 | 域1/2（网络原理） |
| 图解系统 | /os/ | 16W 字 + 400 图 | 域1（OS/计算机组成） |
| 图解MySQL | /mysql/ | 20W 字 + 320 图 | 域2（MySQL 原理） |
| 图解Redis | /redis/ | 18W 字 + 200 图 | 域2（Redis 原理） |
| Java后端面试题 | /interview/ | 1600+ 题 57W 字 | 全领域（面试题集合） |

## 1. 图解网络（/network/）

**定位**：专注计算机网络学习与面试的开源资料，说人话+图解，面向程序员。覆盖面试 90%+ 网络问题，深度应对大厂。

### 网络基础篇
- 🔥 TCP/IP 网络模型有哪几层？ — /network/1_base/tcp_ip_model.html
- 🔥 键入网址到网页显示，期间发生了什么？ — /network/1_base/what_happen_url.html
- Linux 系统是如何收发网络包的？ — /network/1_base/how_os_deal_network_package.html

### HTTP 篇
- 🔥 HTTP 常见面试题 — /network/2_http/http_interview.html
- HTTP/1.1 如何优化？ — /network/2_http/http_optimize.html
- 🔥 HTTPS RSA 握手解析 — /network/2_http/https_rsa.html
- HTTPS ECDHE 握手解析 — /network/2_http/https_ecdhe.html
- HTTPS 如何优化？ — /network/2_http/https_optimize.html
- HTTP/2 牛逼在哪？ — /network/2_http/http2.html
- HTTP/3 强势来袭 — /network/2_http/http3.html
- 🔥 既然有 HTTP 协议，为什么还要有 RPC？ — /network/2_http/http_rpc.html
- 既然有 HTTP 协议，为什么还要有 WebSocket？ — /network/2_http/http_websocket.html

### TCP 篇（26 篇，全站最精华部分）
- 🔥 TCP 三次握手与四次挥手面试题 — /network/3_tcp/tcp_interview.html
- 🔥 TCP 重传、滑动窗口、流量控制、拥塞控制 — /network/3_tcp/tcp_feature.html
- TCP 实战抓包分析 — /network/3_tcp/tcp_tcpdump.html
- TCP 半连接队列和全连接队列 — /network/3_tcp/tcp_queue.html
- 如何优化 TCP? — /network/3_tcp/tcp_optimize.html
- 🔥 如何理解是 TCP 面向字节流协议？ — /network/3_tcp/tcp_stream.html
- 为什么 TCP 每次建立连接时，初始化序列号都要不一样呢？ — /network/3_tcp/isn_deff.html
- SYN 报文什么情况下会被丢弃？ — /network/3_tcp/syn_drop.html
- 已建立连接的 TCP，收到 SYN 会发生什么？ — /network/3_tcp/challenge_ack.html
- 四次挥手中收到乱序的 FIN 包会如何处理？ — /network/3_tcp/out_of_order_fin.html
- 在 TIME_WAIT 状态的 TCP 连接，收到 SYN 后会发生什么？ — /network/3_tcp/time_wait_recv_syn.html
- 🔥 TCP 连接，一端断电和进程崩溃有什么区别？ — /network/3_tcp/tcp_down_and_crash.html
- 🔥 拔掉网线后，原本的 TCP 连接还存在吗？ — /network/3_tcp/tcp_unplug_the_network_cable.html
- tcp_tw_reuse 为什么默认是关闭的？ — /network/3_tcp/tcp_tw_reuse_close.html
- HTTPS 中 TLS 和 TCP 能同时握手吗？ — /network/3_tcp/tcp_tls.html
- TCP Keepalive 和 HTTP Keep-Alive 是一个东西吗？ — /network/3_tcp/tcp_http_keepalive.html
- TCP 有什么缺陷？ — /network/3_tcp/tcp_problem.html
- 如何基于 UDP 协议实现可靠传输？ — /network/3_tcp/quic.html
- TCP 和 UDP 可以使用同一个端口吗？ — /network/3_tcp/port.html
- 服务端没有 listen，客户端发起连接建立，会发生什么？ — /network/3_tcp/tcp_no_listen.html
- 没有 accept，可以建立 TCP 连接吗？ — /network/3_tcp/tcp_no_accpet.html
- 用了 TCP 协议，数据一定不会丢吗？ — /network/3_tcp/tcp_drop.html
- TCP 四次挥手，可以变成三次吗？ — /network/3_tcp/tcp_three_fin.html
- TCP 序列号和确认号是如何变化的？ — /network/3_tcp/tcp_seq_ack.html

### IP 篇
- 🔥 IP 基础知识全家桶 — /network/4_ip/ip_base.html
- 🔥 ping 的工作原理 — /network/4_ip/ping.html
- 断网了，还能 ping 通 127.0.0.1 吗？ — /network/4_ip/ping_lo.html

### 学习心得
- 计算机网络怎么学？ — /network/5_learn/learn_network.html
- 画图经验分享 — /network/5_learn/draw.html

## 2. 图解系统（/os/）

**定位**：操作系统五大结构（进程管理、内存管理、文件系统、设备管理、网络系统）+ 计算机组成（CPU）+ Linux 命令。16W 字 + 400 图。

### 硬件结构
- CPU 是如何执行程序的？ — /os/1_hardware/how_cpu_run.html
- 磁盘比内存慢几万倍？ — /os/1_hardware/storage.html
- 如何写出让 CPU 跑得更快的代码？ — /os/1_hardware/how_to_make_cpu_run_faster.html
- CPU 缓存一致性 — /os/1_hardware/cpu_mesi.html
- CPU 是如何执行任务的？ — /os/1_hardware/how_cpu_deal_task.html
- 什么是软中断？ — /os/1_hardware/soft_interrupt.html
- 为什么 0.1 + 0.2 不等于 0.3 ？ — /os/1_hardware/float.html

### 操作系统结构
- Linux 内核 vs Windows 内核 — /os/2_os_structure/linux_vs_windows.html

### 内存管理
- 🔥 为什么要有虚拟内存？ — /os/3_memory/vmem.html
- malloc 是如何分配内存的？ — /os/3_memory/malloc.html
- 内存满了，会发生什么？ — /os/3_memory/mem_reclaim.html
- 在 4GB 物理内存的机器上，申请 8G 内存会怎么样？ — /os/3_memory/alloc_mem.html
- 如何避免预读失效和缓存污染的问题？ — /os/3_memory/cache_lru.html
- 深入理解 Linux 虚拟内存管理 — /os/3_memory/linux_mem.html
- 深入理解 Linux 物理内存管理 — /os/3_memory/linux_mem2.html

### 进程管理
- 🔥 进程、线程基础知识 — /os/4_process/process_base.html
- 🔥 进程间有哪些通信方式？ — /os/4_process/process_commu.html
- 🔥 多线程冲突了怎么办？ — /os/4_process/multithread_sync.html
- 🔥 怎么避免死锁？ — /os/4_process/deadlock.html
- 🔥 什么是悲观锁、乐观锁？ — /os/4_process/pessim_and_optimi_lock.html
- 一个进程最多可以创建多少个线程？ — /os/4_process/create_thread_max.html
- 线程崩溃了，进程也会崩溃吗？ — /os/4_process/thread_crash.html

### 调度算法
- 进程调度/页面置换/磁盘调度算法 — /os/5_schedule/schedule.html

### 文件系统
- 文件系统全家桶 — /os/6_file_system/file_system.html
- 进程写文件时，进程发生了崩溃，已写入的数据会丢失吗？ — /os/6_file_system/pagecache.html

### 设备管理
- 键盘敲入 A 字母时，操作系统期间发生了什么？ — /os/7_device/device.html

### 网络系统
- 🔥 什么是零拷贝？ — /os/8_network_system/zero_copy.html
- 🔥 I/O 多路复用：select/poll/epoll — /os/8_network_system/selete_poll_epoll.html
- 高性能网络模式：Reactor 和 Proactor — /os/8_network_system/reactor.html
- 什么是一致性哈希？ — /os/8_network_system/hash.html

### Linux 命令
- 如何查看网络的性能指标？ — /os/9_linux_cmd/linux_network.html
- 如何从日志分析 PV、UV？ — /os/9_linux_cmd/pv_uv.html

## 3. 图解MySQL（/mysql/）

**定位**：专注 MySQL 学习与面试，面向后端开发和 DBA。20W 字 + 320 图。高频问题：B+树、索引失效、隔离级别实现、RR 幻读、锁、update 锁全表、三大日志。

### 基础篇
- 🔥 执行一条 SQL 查询语句，期间发生了什么？ — /mysql/base/how_select.html
- MySQL 一行记录是怎么存储的？ — /mysql/base/row_format.html

### 索引篇
- 🔥 索引常见面试题 — /mysql/index/index_interview.html
- 从数据页的角度看 B+ 树 — /mysql/index/page.html
- 🔥 为什么 MySQL 采用 B+ 树作为索引？ — /mysql/index/why_index_chose_bpuls_tree.html
- MySQL 单表不要超过 2000W 行，靠谱吗？ — /mysql/index/2000w.html
- 🔥 索引失效有哪些？ — /mysql/index/index_lose.html
- count(*) 和 count(1) 有什么区别？哪个性能最好？ — /mysql/index/count.html
- MySQL 分页有什么性能问题？怎么优化？ — /mysql/index/limit.html

### 事务篇
- 🔥 事务隔离级别是怎么实现的？ — /mysql/transaction/mvcc.html
- MySQL 可重复读隔离级别，完全解决幻读了吗？ — /mysql/transaction/phantom.html

### 锁篇
- 🔥 MySQL 有哪些锁？ — /mysql/lock/mysql_lock.html
- MySQL 是怎么加锁的？ — /mysql/lock/how_to_lock.html
- update 没加索引会锁全表? — /mysql/lock/update_index.html
- MySQL 记录锁+间隙锁可以防止删除操作而导致的幻读吗？ — /mysql/lock/lock_phantom.html
- MySQL 死锁了，怎么办？ — /mysql/lock/deadlock.html
- 字节面试：加了什么锁，导致死锁的？ — /mysql/lock/show_lock.html

### 日志篇
- 🔥 undo log、redo log、binlog 有什么用？ — /mysql/log/how_update.html

### 内存篇
- 揭开 Buffer_Pool 的面纱 — /mysql/buffer_pool/buffer_pool.html

### 架构篇
- MySQL 的架构是怎样的？ — /mysql/architecture/mysql_architecture.html

## 4. 图解Redis（/redis/）

**定位**：专注 Redis 学习与面试，面向后端开发和架构师。18W 字 + 200 图。高频问题：数据类型底层实现、AOF/RDB、过期删除 vs 内存淘汰、主从/哨兵/Cluster、缓存三难题、缓存一致性、大 Key。

### 基础篇
- 什么是 Redis？ — /redis/base/wath_is_redis.html

### 面试篇
- 🔥 Redis 常见面试题 — /redis/base/redis_interview.html

### 数据类型篇
- 🔥 Redis 数据类型和应用场景 — /redis/data_struct/command.html
- 🔥 图解 Redis 数据结构 — /redis/data_struct/data_struct.html

### 持久化篇
- 🔥 AOF 持久化是怎么实现的？ — /redis/storage/aof.html
- 🔥 RDB 快照是怎么实现的？ — /redis/storage/rdb.html
- Redis 大 Key 对持久化有什么影响？ — /redis/storage/bigkey_aof_rdb.html

### 功能篇
- 🔥 Redis 过期删除策略和内存淘汰策略有什么区别？ — /redis/module/strategy.html
- 🔥 多节点争抢资源，Redis 分布式锁是怎么实现的？ — /redis/module/setnx.html

### 高可用篇
- 🔥 主从复制是怎么实现的？ — /redis/cluster/master_slave_replication.html
- 🔥 为什么要有哨兵？ — /redis/cluster/sentinel.html
- 为什么要有 cluster 集群？ — /redis/cluster/cluster.html
- 如何保证 Redis 分布式锁的高可用和高性能？ — /redis/cluster/redlock.html

### 缓存篇
- 🔥 什么是缓存雪崩、击穿、穿透？ — /redis/cluster/cache_problem.html
- 🔥 数据库和缓存如何保证一致性？ — /redis/architecture/mysql_redis_consistency.html

## 5. Java后端面试题（/interview/）— 24 套 1600+ 题

**定位**：全部来自真实面试题库（腾讯/字节/阿里/美团/京东/快手/百度/滴滴/米哈游 + 中小厂/银行/国企），最近一年出现的题目，平均 2w+ 字/套。

| 面试题 | 题量 | 字数 | 链接 |
|--------|------|------|------|
| Java基础 | 79 | 3.4w | /interview/java.html |
| Java集合 | 69 | 2.7w | /interview/collections.html |
| Java并发编程 | 73 | 3.6w | /interview/juc.html |
| Java虚拟机 | 60 | 2.7w | /interview/jvm.html |
| Spring | 84 | 3.4w | /interview/spring.html |
| MySQL | 115 | 4.2w | /interview/mysql.html |
| Redis | 69 | 3.6w | /interview/redis.html |
| 消息队列 | 53 | 2.4w | /interview/mq.html |
| 分布式 | 34 | 2.3w | /interview/cap.html |
| 系统设计 | 19 | 2.5w | /interview/systemdesign.html |
| 操作系统 | 75 | 3.2w | /interview/os.html |
| 计算机网络 | 96 | 3.6w | /interview/network.html |
| 数据结构与算法 | 53 | 2.5w | /interview/data.html |
| C++ | 112 | 3.4w | /interview/cpp.html |
| Golang | 113 | 2.7w | /interview/golang.html |
| Python | 118 | 3.1w | /interview/python.html |
| Linux 命令 | 21 | 5k | /interview/linux.html |
| Git 命令 | 24 | 5k | /interview/git.html |
| Docker | 22 | 5k | /interview/docker.html |
| 测试开发系列 | 5 套 | — | /interview/test_dev.html 等 |

**后端技术进阶篇**：后端技术进阶之路 — /interview/win.html（系统设计、消息队列、后端场景、微服务、分布式）

## 面试项目（与 xiaolinnote 共享）

DevFlow AI 项目（多 Agent 协作，25 万字）、MewHelp 智能客服（AI 应用开发，25 万字）、MewCode Agent（Agent 开发，40 万字）、智能 OnCall Agent（10 万字）

## 去重与互补策略

| 重叠主题 | xiaolincoding 定位 | 与其他站点关系 |
|----------|-------------------|----------------|
| TCP/HTTP/网络 | **图解主来源**（500 图，TCP 26 篇全互联网最细） | JavaGuide cs-basics 作索引；本系列作正文 |
| OS/计算机组成 | **图解主来源**（400 图） | JavaGuide cs-basics/OS 作补充 |
| MySQL/Redis | 图解原理（数据页看 B+树、BufferPool、加锁规则） | JavaGuide 面试题作主干；本系列作原理可视化正文 |
| Java 面试题 | 1600+ 真实面试题 | 与 JavaGuide 面试题互补，按主题各取所长 |

## 抓取优先级

1. **P0**：TCP 三次握手四次挥手、TCP 重传/滑动窗口/流量/拥塞、键入网址全过程、HTTP 面试题、HTTPS RSA/ECDHE 握手（网络核心 6 篇）
2. **P1**：为什么 B+树、索引失效、事务隔离级别实现(MVCC)、MySQL 有哪些锁/怎么加锁、三大日志(how_update)、BufferPool、Redis 数据结构、AOF/RDB、缓存三难题、缓存一致性（MySQL/Redis 核心 10 篇）
3. **P2**：零拷贝、IO 多路复用、Reactor、虚拟内存、进程线程、死锁（OS 与网络系统 6 篇）
4. **P3**：面试题集合按主题与 JavaGuide 互补收录
