# 索引失效有哪些？

> **来源**：小林coding（xiaolincoding.com）
> **原文链接**：https://xiaolincoding.com/mysql/index/index_lose.html
> **抓取日期**：2026-10-18
> **所属知识域**：域2 后端技术组件工程化
> **板块**：图解MySQL · 索引篇

---

## 前提：索引存储结构

- InnoDB 用 **B+Tree**，叶子节点**保存数据本身**（聚簇索引）；MyISAM 叶子节点保存**物理地址**；
- 二级索引叶子节点存**主键值** → 查完整数据需**回表**；只查索引覆盖字段则**覆盖索引**不回表。

## 六种索引失效场景

### 1. 左/左右模糊匹配

```sql
select * from t_user where name like '%林';   -- ✗ 全表扫描
select * from t_user where name like '林%';   -- ✓ 走索引（range）
```

原因：B+ 树按索引值**有序排列**，只能按前缀比较，后缀匹配无法定位起始点。

### 2. 对索引列使用函数

```sql
select * from t_user where length(name)=6;    -- ✗ 全表扫描
```

原因：索引保存**原始值**，非函数计算结果。MySQL 8.0 起支持**函数索引**：`alter table t_user add key idx_name_length ((length(name)))`。

### 3. 对索引列表达式计算

```sql
explain select * from t_user where id + 1 = 10;   -- ✗ 全表扫描
explain select * from t_user where id = 10 - 1;   -- ✓ 走索引
```

原因同函数：索引不存计算后的值。MySQL 未做简单表达式重写，把责任交给程序员。

### 4. 隐式类型转换（最易踩坑）

```sql
-- phone 是 varchar 索引列
select * from t_user where phone = 1300000001;   -- ✗ 全表扫描
-- id 是整型索引列
select * from t_user where id = '1';             -- ✓ 走索引
```

**MySQL 遇到字符串与数字比较，自动把字符串转成数字**（`select "10" > 9` 结果为 1 可验证）。所以：

- 字符串索引列 vs 数字参数 → 相当于 `CAST(phone AS signed) = 1300000001`——**CAST 作用在索引列上 = 对索引用函数** → 失效；
- 整型索引列 vs 字符串参数 → CAST 作用在输入参数上 → 不影响索引。

### 5. 联合索引非最左匹配

- 联合索引 `(a,b,c)` 可用：`where a=1`、`where a=1 and b=2`、`where a=1 and b=2 and c=3`；
- 失效：`where b=2`、`where c=3`、`where b=2 and c=3`；
- 特殊：`where a=1 and c=3`（索引截断）——MySQL 5.5 之前 a 走索引回表后在 Server 层过滤 c；**MySQL 5.6+ 索引下推（ICP）**：c 在存储引擎层过滤，Extra 显示 `Using index condition`，减少回表。

原因：联合索引按第一列排序，第一列相同才按第二列排——不从最左列开始无法利用有序性。

### 6. WHERE 中 OR 跨索引与非索引列

```sql
select * from t_user where id = 1 or age = 18;   -- id 是索引、age 不是 → ✗ 全表扫描
```

OR 语义是满足其一即可，只要一个条件列无索引就只能全表。给 age 建索引后走 **index_merge**（分别扫描两个索引再合并）。

## 总结表

| 失效场景 | 根因 | 规避 |
|----------|------|------|
| `like %xx` / `like %xx%` | 无法定位前缀起始点 | 前缀匹配 `xx%`；或全文索引 |
| 索引列用函数 | 索引存原始值 | 换写法；或函数索引（8.0+） |
| 索引列表达式计算 | 同上 | 把计算移到等值另一边（`id = 10-1`） |
| 隐式类型转换 | 字符串↔数字自动转换 = 对索引 CAST | 参数类型与列类型严格一致 |
| 联合索引非最左 | 排序仅从最左列开始 | 按最左前缀设计查询 |
| OR 跨非索引列 | 单条件索引无意义 | 全建索引走 index_merge 或改 UNION |
