# 如何使用 Builder 模式创建线程池

> **来源站点**：拿个offer（nageoffer.com）12306 项目核心技术文档
> **原文链接**：https://nageoffer.com/12306/threadpool-build/
> **抓取日期**：2026-10-20
> **所属知识域**：域7 业务场景与实战案例 / 域1 Java核心与编码工程化
> **板块**：12306 项目文档库·核心技术文档

---

## 1. Builder 模式应用场景

Builder 设计模式也叫做**构建者模式或者建造者模式**。"设计模式重要的是思想，一种设计模式可能不止一种实现方式。"

**适用判据**：如果**类的属性之间有一定的依赖关系或者约束条件**，那么就可以考虑使用 Builder 设计模式。

### 1.1 线程池为什么适合 Builder

默认创建的线程池构造方法最多有**七个参数**：核心线程数、最大线程数、阻塞队列、线程存活时间……

线程池设置如此多构造参数，**是因为对这些参数会有一定规则的校验**，不满足则不允许创建，通过**抛异常终止程序**。校验规则：

1. 核心线程数不可以小于 0；
2. 线程存活时间不可以小于 0；
3. 最大线程数不可以小于等于 0，同时也不可以小于核心线程数；
4. 阻塞队列、线程工厂、拒绝策略参数均不可为空。

两个作用：让核心参数满足线程池运行流程；保障运行时稳定性。

**结论**：构造参数过多 + 属性之间存在依赖关系和约束条件 → 线程池创建非常适合 Builder 模式。

## 2. Hutool Builder 创建线程池（源码级）

Hutool thread 包下线程池相关有两处使用 Builder：创建线程池 + 创建线程工厂。

**`ExecutorBuilder#create` 返回自己本身**——先创建构建器对象，通过 set 方法把数据填充上去，最后进行依赖关系整理和条件约束：

```java
public static ExecutorBuilder create() {
  return new ExecutorBuilder();
}
```

**`build` 方法**：Hutool 定义了 `Builder` 接口，实现此接口即完成 Builder 模式，泛型 T 代表需要返回的构造对象类型（线程池 Builder 泛型就是 `ThreadPoolExecutor`）。`build()` 调用真正管理依赖和约束的重载方法 `build(ExecutorBuilder builder)`，将赋过值的构建对象传入，返回所需线程池对象。

**链式风格**是作者抽象公共业务时的常用模式。

## 3. Builder 模式不同的实现方式

### 3.1 静态内部类实现

```java
@Getter
public class HttpParameters {
    private Builder builder;
    public static Builder newBuilder() { return new Builder(); }
    private HttpParameters(Builder builder) { this.builder = builder; }

    @Getter
    public static class Builder {
        private String url;
        private Object parameter;
        private String httpType;
        public Builder parameter(Object parameter) { this.parameter = parameter; return this;}
        public Builder url(String url) { this.url = url; return this; }
        public Builder httpType(String httpType) { this.httpType = httpType; return this; }
        public HttpParameters build() {
            if (StringUtils.isBlank(url)) { throw new RuntimeException("URL不允许为空 "); }
            // ...
            return new HttpParameters(this);
        }
    }
}
```

获取参数需先获取 Builder 对象；也可以把 Builder 属性在目标类里再定义一份，"方式都很灵活"。

## 4. 核心要点

- Builder 强调的是**管理依赖关系或者约束条件**，而非固定代码写法；
- Hutool 式（`Builder` 接口 + `create()` 自创建）与静态内部类式都是正确实现，按团队习惯选择；
- 适合场景速查：参数 ≥ 4 个且存在校验/依赖 → Builder；参数少且无约束 → 直接构造器或静态工厂。
