# Spring 常见面试题总结

> **来源**：JavaGuide（javaguide.cn）
> **原文链接**：https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2 后端技术组件工程化（框架中间件）
> **板块**：系统设计 · Spring 专题

---

## 一、Spring 基础

- **Spring**：开源轻量级 Java 开发框架，核心功能 **IoC + AOP**；
- **三者关系**：Spring（最底层，核心 Spring-Core/IoC）→ Spring MVC（Web 层模块，天生与 Spring 集成）→ Spring Boot（简化配置、开箱即用，但 MVC 开发仍基于 Spring MVC）。

## 二、IoC

- **IoC 是一种思想**：对象创建与管理权从类内部 `new` 反转**交给 IoC 容器**；
- 收益：降低耦合、资源易管理（容器级单例）、便于替换实现；
- **Bean 声明注解**：`@Component`（通用）/ `@Repository`（Dao）/ `@Service`（Service）/ `@Controller`（Web 层）；

### @Component vs @Bean

| 对比项 | @Component | @Bean |
|--------|-----------|-------|
| 作用目标 | 类 | 方法 |
| 装配方式 | 类路径扫描自动装配 | 方法内定义产生 bean |
| 自定义性 | 较弱 | 强（可写复杂逻辑） |
| 第三方库类 | 无法使用 | **只能通过 @Bean 注册** |

### @Autowired vs @Resource

| 对比项 | @Autowired | @Resource |
|--------|-----------|-----------|
| 来源 | Spring 内置 | Jakarta/JSR-250 |
| 匹配顺序 | **先 byType 再 byName** | **先 byName 再 byType** |
| 多实现指定 | `@Qualifier` | `name` 属性 |
| 使用位置 | 构造器/方法/字段/参数 | 主要字段和方法 |

### 注入方式

三种：构造器 / Setter / Field。**Spring 官方推荐构造器注入**：依赖完整性（避免 NPE）、不可变性（线程安全）、初始化保证、测试便利（无需容器直接传 mock）。Setter 注入适合可选依赖。

## 三、AOP

- **AOP**：把与业务无关但被各模块共同调用的逻辑（事务、日志、权限）封装为切面，减少重复代码、降低耦合；
- **Spring AOP 基于动态代理**：目标实现接口 → **JDK Proxy**；未实现接口 → **CGLIB**（生成子类）；

### 核心术语

| 术语 | 含义 |
|------|------|
| Target / Proxy | 被通知对象 / 应用通知后创建的代理对象 |
| JoinPoint | 目标类所有可拦截方法（连接点） |
| Pointcut | 被切面拦截的连接点（切入点 ⊆ 连接点） |
| Advice | 增强逻辑（通知） |
| Aspect | Pointcut + Advice |
| Weaving | 织入，把通知应用到目标对象生成代理的过程 |

### Spring AOP vs AspectJ

| 特性 | Spring AOP | AspectJ |
|------|-----------|---------|
| 增强时机 | 运行时（动态代理） | 编译期/类加载期（字节码） |
| 切入点 | 仅方法级（不支持 final/static） | 方法/字段/构造器/静态方法 |
| 性能 | 有代理开销 | 运行时无代理开销 |
| 选择 | 大多数场景 | 需增强 final/字段/高性能时 |

### 通知类型与顺序

- `Before` / `After` / `AfterReturning` / `AfterThrowing`（与 AfterReturning 互斥）/ `Around`（可编程控制，甚至不执行目标方法）；
- 多切面顺序：`@Order`（**值越小优先级越高**）或实现 `Ordered#getOrder()`。

## 四、Bean 作用域、线程安全与生命周期

### 作用域

singleton（默认，容器唯一）/ prototype（每次新建）/ request / session / application / websocket（后四者仅 Web）。

### 线程安全

- 大部分 Bean 无状态（Dao/Service）→ 默认线程安全；
- **有状态单例 Bean 解决方案**：① 无可变成员变量（无状态设计）；② ThreadLocal；③ synchronized/ReentrantLock。

### 生命周期（四阶段）

1. **实例化**：反射创建；
2. **属性赋值**：处理 @Autowired/@Value/@Resource；
3. **初始化**：`*Aware` 接口 → `BeanPostProcessor#postProcessBeforeInitialization` → `InitializingBean#afterPropertiesSet`/`init-method` → `BeanPostProcessor#postProcessAfterInitialization`（**AOP 代理在此生成**）；
4. **销毁**：`DisposableBean#destroy`/`destroy-method`/`@PreDestroy`。

## 五、循环依赖与三级缓存

- **循环依赖**：Bean 相互持有对方引用（A→B→A）；
- **三级缓存**：

| 级别 | 名称 | 内容 |
|------|------|------|
| 一级 | singletonObjects | 最终形态 Bean（完整生命周期），单例池 |
| 二级 | earlySingletonObjects | 半成品 Bean（未属性填充），防 AOP 重复生成代理 |
| 三级 | singletonFactories | ObjectFactory，getObject()（getEarlyBeanReference）生成原始或代理对象 |

- **解决流程**：A 实例化后**提前暴露**到三级缓存；B 依赖 A 时经 三级→二级 拿到 A 的前期暴露对象注入，打破循环；
- **为什么不能只用两级**：无 AOP 时可以；有 AOP 时三级缓存保证**多次早期引用返回同一代理对象**，避免同一 Bean 多个代理；
- **局限性**：只解决**部分单例**的 Setter/字段注入循环依赖；**构造器注入、原型 Bean 无法解决**；
- `@Lazy`：注入点加 @Lazy 注入延迟解析代理，真正调用时才初始化目标 Bean，打破循环；
- Spring Boot **2.6 起默认禁止**循环依赖，需 `spring.main.allow-circular-references=true` 开启。

## 六、Spring 事务

- **编程式事务**：TransactionTemplate/TransactionManager 手动管理（分布式场景推荐）；
- **声明式事务**：`@Transactional`，本质 AOP；

### 传播行为（默认 REQUIRED）

| 传播行为 | 含义 |
|----------|------|
| REQUIRED | 有事务则加入，无则新建（最常用） |
| REQUIRES_NEW | 新建事务，有则挂起；相互独立 |
| NESTED | 有事务则嵌套事务（savepoint），无则等价 REQUIRED |
| MANDATORY | 有则加入，无则抛异常 |
| SUPPORTS | 有则加入，无则非事务 |
| NOT_SUPPORTED | 非事务运行，有则挂起 |
| NEVER | 非事务运行，有则抛异常 |

### 回滚规则

- **默认只对 RuntimeException 和 Error 回滚，Checked Exception 不回滚**；
- `rollbackFor` 指定需回滚异常；`noRollbackFor` 指定不回滚。

## 七、Spring MVC

### 核心组件

DispatcherServlet（中央处理器）/ HandlerMapping（URL→Handler）/ HandlerAdapter（适配执行）/ ViewResolver（逻辑视图→实际视图）。

### 执行流程（7 步）

请求 → DispatcherServlet → HandlerMapping 匹配 → HandlerAdapter 执行 Handler → 返回 ModelAndView → ViewResolver 解析视图 → 渲染 Model → 返回响应。

**前后端分离变化**：返回 JSON（@RestController 或 @Controller+@ResponseBody），**ViewResolver 不再使用**。

### 统一异常处理

`@ControllerAdvice` + `@ExceptionHandler`。原理：**不是给 Controller 创建 AOP 代理**；Controller 抛异常后由 Spring MVC 的 `HandlerExceptionResolver` 处理链查找匹配方法（`ExceptionHandlerMethodResolver#getMappedMethod()` 按匹配深度排序取最匹配的）。

## 八、Spring Boot 自动配置（待补充）

@SpringBootApplication / @EnableAutoConfiguration / @Conditional / spring-boot-autoconfigure 加载机制（spring.factories → AutoConfiguration.imports）——原文未展开，列入后续抓取计划。
