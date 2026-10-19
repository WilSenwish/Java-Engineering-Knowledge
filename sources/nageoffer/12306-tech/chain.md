# 责任链模式重构复杂业务场景

> **来源站点**：拿个offer（nageoffer.com）12306 项目核心技术文档
> **原文链接**：https://nageoffer.com/12306/chain/
> **抓取日期**：2026-10-20
> **所属知识域**：域7 业务场景与实战案例 / 域1 Java核心与编码工程化
> **板块**：12306 项目文档库·核心技术文档

---

## 1. 什么是责任链模式

多个处理器依次处理同一个请求，形成链条，**链条上的每个处理器各自承担各自的处理职责**。SpringMVC 拦截器、MyBatis Interceptor 都是责任链的经典应用。

**两种处理方式**：
1. **全量执行型**：请求会被所有处理器都处理一遍，不存在中途终止——参照 MyBatis 拦截器，"重点在对请求过程中的数据或者行为进行改变"；
2. **可阻断型**：某一处理器不符合自定制规则时**停止流程**，剩余处理器不执行——参照 SpringMVC 拦截器（如 Token 失效即打回）。

```java
// 可阻断型：处理器返回布尔值，false 则中断
public class ChainApplication {
    public static void main(String[] args) {
        HandlerChain handlerChain = new HandlerChain();
        handlerChain.addHandler(Lists.newArrayList(new HandlerA(), new HandlerB()));
        boolean resultFlag = handlerChain.handle();
        if (!resultFlag) {
            System.out.println("责任链中处理器不满足条件");
        }
    }
}
```

## 2. 业务场景：上游数据同步校验

**场景**：系统作为下游服务，基础数据需从上游中台同步。"数据只要是人为录入就极可能存在问题，**遵从对上游系统不信任原则**"，接收时需一系列校验：

1. **必填项校验**：数据无法满足业务必须字段要求，落入库中会产生一系列问题；
2. **非法字符校验**：上游录入规则不可控；
3. **长度校验**：如系统字段限制 50 长度，接入数据 500 长度会出问题。

**Spring 装配**：`VerifyHandlerChain` 实现 `InitializingBean`，在 `afterPropertiesSet` 中获取 IOC 容器中所有 `VerifyHandler` 类型 Bean，添加到处理器链容器；定义 `verify()` 方法展开链式调用，无待验证数据直接返回。

**收益**：新增校验规则只需实现 `VerifyHandler` 接口新建实现类，**符合开闭原则，提高代码扩展性**。

## 3. 为什么不用 if-else/大函数

1. 所有校验写到一起 → "类或者说方法函数奇大无比"。拆分成不同函数/类能解决复杂性，但面临第二个问题；
2. **开闭原则**：添加新功能应该在已有代码基础上**扩展而非修改**。写了三套校验规则运行一段时间，领导让加第四套，就要在原有代码上改动。

"在合适的场景运用适合的设计模式，能够让代码设计复杂性降低，变得更为健壮。"

## 4. 真实业务的复杂度（示例之外）

1. **如何定义处理器的先后调用顺序**：如某处理器执行时间长且过滤数据少，希望放到最后执行；
2. **如何自定义单个数据类型过滤**：如接入学生数据，学号有特定校验规则，该处理器只适合单一类型。

"设计模式强调的应该是一种思想，而不是固定的代码写法，需要结合业务场景灵活变通。"

## 5. MyBatis Interceptor 底层实现（源码级）

MyBatis 3.4.x 的 Interceptor 底层即责任链 + 动态代理相结合，服务于四大"护教法王"：

1. **ParameterHandler**
2. **ResultSetHandler**
3. **StatementHandler**
4. **Executor**

定义了拦截器集合 `interceptors` 与添加拦截器方法。查询语句定义多个拦截器时，先经过各拦截器代码加工，全部执行完毕才走真正查询。分页插件（PageHelper、MyBatis-Plus 分页）即基于 Interceptor 实现。

"想要读框架源码，需要一定的设计模式基础。如果对责任链、动态代理不清楚，那么就不能理解这一块的精髓。"

## 6. 关联

- 牛券项目第 06 小节"基于责任链模式创建优惠券模板"是该模式的业务落地；
- Ragent AI 项目中责任链用于检索后处理与模型故障降级（见 docs/07.1 模块二）。
