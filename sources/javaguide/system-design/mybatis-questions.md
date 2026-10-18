# MyBatis 常见面试题总结

> **来源**：JavaGuide（javaguide.cn）
> **原文链接**：https://javaguide.cn/system-design/framework/mybatis/mybatis-interview.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2 后端技术组件工程化（框架中间件）
> **板块**：系统设计 · MyBatis 专题

---

## 一、定位与选型

- **半自动 ORM**：封装 JDBC 重复工作（连接/参数/执行/遍历/关闭），但 **SQL 与映射由开发者控制**；
- 适合 SQL 复杂、需精确优化、重度使用数据库特性的项目；代价是 SQL 与映射代码多、方言迁移/关联加载/批量操作需自行处理。

### MyBatis vs JPA/Hibernate

| 对比项 | MyBatis | JPA/Hibernate |
|--------|---------|---------------|
| SQL | 开发者编写 | 框架按映射生成 |
| 控制粒度 | 精确控制 SQL/索引/DB 特性 | 关注对象模型 |
| 开发成本 | 映射和 SQL 多 | 常规 CRUD 少 |
| 迁移 | 手写 SQL 可能依赖方言 | 标准查询可移植性好 |
| 关联查询 | 显式 Join/嵌套 | 对象关联+抓取策略 |
| 风险 | SQL 分散、映射错误、拼接注入 | N+1、抓取过大、SQL 难控 |

## 二、执行流程

**启动期**：读全局配置建 `Configuration` → 解析 Mapper XML/注解注册 `MappedStatement`/`ResultMap`/`SqlSource` → 创建 `SqlSessionFactory` → Spring 注册 Mapper 代理。

**运行期**：Mapper 代理按接口+方法名找 `MappedStatement` → `SqlSource` 生成 `BoundSql` → `Executor` 查一级缓存，未命中走库 → `StatementHandler` 建 JDBC Statement、`ParameterHandler` 设参 → JDBC 执行 → `ResultSetHandler` 映射为对象 → 写缓存返回。

**插件可拦截四大组件**：Executor、StatementHandler、ParameterHandler、ResultSetHandler（分页/审计/SQL 改写插在这条链路）。

## 三、参数处理与动态 SQL

### #{} vs ${}

| 特性 | #{} | ${} |
|------|-----|-----|
| 生成内容 | JDBC 占位符 `?` | 直接替换进 SQL 文本 |
| 绑定 | ParameterHandler + TypeHandler | 无绑定 |
| 用途 | 值参数 | 列名/表名/排序方向 |
| 注入风险 | 防注入 | **有注入风险** |

- `${}` 出现在配置文件（`${driver}`）是**属性替换**，与 SQL 动态文本替换场景不同；
- `ORDER BY ${orderBy}` 若来自请求 = 注入漏洞，应在 Java 层把外部值映射为**服务端预定义白名单**。

### 动态 SQL 标签与原理

- 标签：`<if>` / `<choose><when><otherwise>` / `<trim><where><set>` / `<foreach>`（IN、批量插入）/ `<bind>`；
- 原理：XML 节点构造为 `SqlNode` 组合结构，执行时各节点结合参数上下文与 **OGNL 表达式**决定是否输出 SQL，最终由 `DynamicSqlSource` 生成 `BoundSql`；`<where>` 自动处理开头多余 AND/OR。

## 四、结果映射

- `resultType`：简单查询自动匹配（可配合 `mapUnderscoreToCamelCase=true` 把 user_name → userName，SQL 列别名 `AS` 亦可）；
- `resultMap`：显式列↔属性映射，支持构造器映射、typeHandler、鉴别器（discriminator）、嵌套 association/collection；**Join 重名列必须用别名并在 resultMap 明确配置**；
- `selectOne()` 查到多条 → 抛 `TooManyResultsException`，**不会自动取第一条**——业务唯一性应由唯一约束保证，不能用 ORDER BY + LIMIT 1 掩盖脏数据；
- **空结果/映射不全排查顺序**：SQL 是否真返回数据 → namespace/Statement ID 对应 → 列别名与属性/驼峰映射 → resultMap 的 column/property/javaType/typeHandler → Join 同名列覆盖/嵌套 <id> 完整性 → Setter/构造器 → 一级缓存返回旧结果。看日志要带最终参数，只有占位符不能说明实际条件。

## 五、缓存

### 一级缓存（SqlSession 级，默认 SESSION）

- 同一 SqlSession 中 Statement+SQL+参数+分页+环境相同 → 第二次直接命中；
- 失效：insert/update/delete、`clearCache()`、提交/回滚/关闭 SqlSession、查询配 `flushCache="true"`；
- **Spring 中感觉没生效**：`SqlSessionTemplate` 复用绑定到当前事务的 SqlSession；**无事务时每次 Mapper 调用后关闭 Session**，下次是新 Session。同一 Service 方法连续两次 Mapper 调用，无事务时不保证共享一级缓存；
- **脏读表现**：Session 内首次查询后数据库被别的事务更新，本 Session 再查仍返回旧对象。解法：clearCache()、缩短 Session 生命周期、`localCacheScope=STATEMENT`。

### 二级缓存（namespace 级）

- 全局 `cacheEnabled` 默认开，但具体命名空间要配 `<cache/>` 或 `<cache-ref>`；
- 查询结果**事务提交后**才对其他 Session 可见；该命名空间增删改默认清空；缓存对象通常需序列化；
- **生产慎用原因**：① 按 namespace 组织，跨 Mapper 改同表互不感知；② 数据被其他应用/脚本/直连 SQL 修改无法感知；③ 本地缓存多实例不同步；④ Join 依赖多表，任一表变化都过期；⑤ 大对象低命中率白占内存；
- **结论**：跨实例共享+明确失效策略 → 业务层用 Redis/Caffeine，DB 为权威源；MyBatis 二级缓存适合数据变化少、读多写少、依赖清晰的查询。

## 六、插件原理

- 实现 `Interceptor`，`@Intercepts` + `@Signature` 声明目标接口/方法/参数类型；
- MyBatis 创建组件时调 `pluginAll()`，符合签名者被 **JDK 动态代理**包裹，方法调用进入 `intercept()`；
- 注意：插件进入所有匹配 SQL 主链路，代码须轻量；**避免修改共享的 MappedStatement**；多插件（分页+审计+数据权限）同时改写 SQL 要做组合测试。

## 七、延迟加载

- 对嵌套查询配置的 association/collection 生效；`lazyLoadingEnabled` 开启后访问属性时代理执行登记查询回写结果；`fetchType=lazy/eager` 可覆盖全局设置；
- MyBatis 3.5.x 默认 **Javassist** 生成代理，CGLIB 自 3.5.10 起弃用；
- 风险：**N+1**；对象离开 SqlSession/事务作用域后访问属性无法加载；
- 实践：接口返回 DTO 时更适合 Service 内查齐数据。

## 八、关键结论速记

| 主题 | 结论 |
|------|------|
| 半自动 ORM | SQL 控制权在开发者 |
| #{} vs ${} | 前者占位符防注入；后者文本替换仅可用白名单片段 |
| 一级缓存 | SqlSession 级；增删改/clearCache/提交/flushCache 失效；Spring 无事务时不共享 |
| 二级缓存 | namespace 级；需 `<cache/>`；提交后可见；生产慎用 |
| 插件 | JDK 动态代理 + @Intercepts，拦截四大组件 |
| 延迟加载 | 代理对象 + Javassist，注意 N+1 与 Session 作用域 |
| selectOne | 多条抛 TooManyResultsException |
