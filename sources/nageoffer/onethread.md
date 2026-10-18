# oneThread 动态线程池

> **来源站点**：拿个offer（nageoffer.com）
> **原文链接**：https://nageoffer.com/onethread/
> **抓取日期**：2026-10-18
> **所属知识域**：域1 Java核心与编码工程化 / 域4 工程交付与云原生 / 域7 业务场景与实战案例
> **板块**：中间件轮子项目（星球专属，公开介绍页）

---

## 1. 基础介绍

oneThread 是基于**配置中心**构建的动态可观测 Java 线程池框架，弥补 JDK 原生线程池**参数配置不灵活的不足**，支持核心参数的**在线动态调整**、**运行时状态监控**与**阈值告警**，提升系统稳定性与可运维性。兼容主流配置中心（`Nacos`、`Apollo`），实现参数热更新与统一管理。

后端核心代码（含注释）1.34w 行，纯 Java 代码 4.99k 行。

**技术栈**：Spring、Spring Boot、JUC、Tomcat、Jetty、Nacos、Apollo、Prometheus、Grafana。

## 2. 为什么会有动态线程池？

> 参考：美团《Java线程池实现原理及其在美团业务中的实践》

**事故一：任务执行拒绝**——展示接口内部用线程池并行计算，流量预估不足，最大核心数设置偏小，大量 `RejectedExecutionException` 触发接口降级。

**事故二：任务执行时间过长**——用线程池做资源隔离，但**队列设置过长**，最大线程数设置失效（队列永远不满就永远不扩容到 max），请求增加时任务大量堆积，执行时间过长导致下游超时失败。

## 3. 技术实现

### 3.1 动态参数刷新能力

基于配置中心实现核心参数动态刷新，**模板方法模式**复用刷新逻辑，支持多种配置中心（Nacos、Apollo）监听事件统一注册处理。

配置示例：

```yaml
onethread:
  nacos:
    data-id: onethread-nacos-cloud-example-ding-ma.yaml
    group: DEFAULT_GROUP
  config-file-type: YAML
  web:
    core-pool-size: 20
    maximum-pool-size: 400
    keep-alive-time: 60
    notify:
      receives: xxxxxx
  notify-platforms:
    platform: DING
    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxx
  executors:
    - thread-pool-id: onethread-producer
      core-pool-size: 12
      maximum-pool-size: 18
      queue-capacity: 2000
      work-queue: ResizableCapacityLinkedBlockingQueue
      rejected-handler: CallerRunsPolicy
      keep-alive-time: 60
      allow-core-thread-time-out: false
      notify:
        receives: xxxxxx
        interval: 10
      alarm:
        enable: true
        queue-threshold: 80
        active-threshold: 80
```

### 3.2 拒绝策略增强机制

基于 JDK `InvocationHandler` 动态代理扩展拒绝策略，拦截 `rejectedExecution` 事件触发告警，实现任务拒绝实时响应：

```java
@AllArgsConstructor
public class RejectedProxyInvocationHandler implements InvocationHandler {

    private final Object target;
    private final AtomicLong rejectCount;

    private static final String REJECT_METHOD = "rejectedExecution";

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (REJECT_METHOD.equals(method.getName()) &&
                args != null && args.length == 2 &&
                args[0] instanceof Runnable &&
                args[1] instanceof ThreadPoolExecutor) {
            rejectCount.incrementAndGet();
        }
        if (method.getName().equals("toString") && method.getParameterCount() == 0) {
            return target.getClass().getSimpleName();
        }
        try {
            return method.invoke(target, args);
        } catch (InvocationTargetException ex) {
            throw ex.getCause();
        }
    }
}
```

## 4. 项目结构

```text
.
├── core          # 动态线程池核心模块，基础类定义
├── dashboard-dev # 前端可视化页面（查看/调试）
├── example       # 示例包（apollo-example / nacos-cloud-example）
├── spring-base   # Spring 扫描动态线程池、启用开关、Banner
└── starter       # 配置中心组件包
    ├── adapter   # 适配层（对接 Tomcat 等 Web 容器线程池）
    │   └── web-spring-boot-starter
    ├── apollo-spring-boot-starter
    ├── common-spring-boot-starter
    ├── dashboard-dev-spring-boot-starter
    └── nacos-cloud-spring-boot-starter
```

## 5. 项目质量

- **合理分层**：core（核心）+ starter（可插拔配置中心）+ spring-base（基础封装）+ example + dashboard-dev（可视化）——「核心逻辑 + 接入层 + 展示层」；
- **设计模式**：模板方法（刷新逻辑复用）、观察者（监听配置变化）、构建者（线程池参数构造）、代理（增强拒绝处理器）；
- **完整链路**：Nacos/Apollo 动态参数 + Prometheus/Grafana 采集可视化 + 钉钉统一告警——配置-执行-监控-告警闭环；
- **可视化控制台**：线程池列表、参数编辑、监控页面（内嵌 Grafana）。

## 6. 能学到什么

- 基于 SpringBoot Starter 构建线程池组件，快速接入；
- 动态刷新线程池参数，兼容多种配置中心；
- 定时监控线程池状态，异常告警；
- 扩展拒绝策略，拦截任务并告警；
- 接入监控系统，指标采集与可视化；
- 优雅关闭线程池，避免任务丢失；
- 模板、构建者、动态代理、观察者等设计模式。

> 本质上来说，从业务开发思维中脱离，了解框架运行本质。

## 7. 学习路径

oneThread **不开源**，仅星球成员可学（GitCode 仓库，申请开通）。适合：做过业务系统开发（拿个offer任意项目/黑马点评/瑞吉外卖）、了解并发编程（线程池、阻塞队列）的同学。
