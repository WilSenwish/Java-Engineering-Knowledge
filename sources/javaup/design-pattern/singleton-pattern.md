---
来源站点: javaup.chat（码力全开）
原文链接: https://javaup.chat/java/design-pattern/singleton-pattern/
抓取日期: 2026-10-19
所属知识域: 域1 Java核心与编码工程化
板块: 设计模式-单例
---

# 单例模式深度解析

## 单例模式核心思想

单例模式确保一个类在整个应用程序生命周期中只存在一个实例，并提供一个全局访问点来获取该实例。

实现单例的核心要点：

1. **私有化构造函数**：防止外部通过 new 关键字创建对象实例
2. **提供获取实例的静态方法**：保证只初始化一个单例对象
3. **考虑线程安全问题**：在多线程环境下确保单例的唯一性

## 五种经典实现方式

### 懒汉式单例

懒汉式的核心思想是**延迟加载**——只有在真正需要使用时才创建实例对象。

```java
public class ConfigurationManager {
    private static ConfigurationManager instance;
    
    private ConfigurationManager() {
        // 私有构造，加载配置文件
    }
    
    public static synchronized ConfigurationManager getInstance() {
        if (instance == null) {
            instance = new ConfigurationManager();
        }
        return instance;
    }
}
```

**优势**：避免过早创建对象造成资源浪费。
**劣势**：首次创建时存在延迟；synchronized 会带来性能开销。

### 饿汉式单例

饿汉式在类加载时就立即创建单例对象，确保在任何时刻访问都能获取到实例。

```java
public class ApplicationContext {
    private static final ApplicationContext INSTANCE = new ApplicationContext();
    
    private ApplicationContext() {}
    
    public static ApplicationContext getInstance() {
        return INSTANCE;
    }
}
```

也可用静态代码块实现。**优势**：实现简单，天然线程安全。**劣势**：类加载时就创建，可能造成资源浪费。

### 静态内部类单例

结合懒加载和线程安全的优势：

```java
public class CacheManager {
    private CacheManager() {}
    
    private static class CacheManagerHolder {
        private static final CacheManager INSTANCE = new CacheManager();
    }
    
    public static CacheManager getInstance() {
        return CacheManagerHolder.INSTANCE;
    }
}
```

**工作原理**：外部类加载时内部类不加载；调用 getInstance() 才触发内部类初始化；类初始化过程由 JVM 保证线程安全。

### 双重检查锁单例（DCL）

```java
public class ServiceRegistry {
    // 必须使用volatile防止指令重排
    private volatile static ServiceRegistry instance;
    
    private ServiceRegistry() {}
    
    public static ServiceRegistry getInstance() {
        if (instance == null) {                    // 第一次检查：避免不必要的同步
            synchronized (ServiceRegistry.class) {
                if (instance == null) {            // 第二次检查：防止重复创建
                    instance = new ServiceRegistry();
                }
            }
        }
        return instance;
    }
}
```

**为什么需要 volatile**：对象创建三步（分配内存、初始化对象、引用指向内存地址）可能被 JVM 重排序，导致其他线程获取到未完全初始化的对象。volatile 确保有序性。

### 枚举单例

```java
public enum GlobalConfig {
    INSTANCE;
    
    private String appName;
    private String version;
    
    GlobalConfig() {
        this.appName = "MyApplication";
        this.version = "1.0.0";
    }
}
```

## 枚举单例的优势分析

《Effective Java》明确指出：**单元素的枚举类型已经成为实现单例的最佳方法**。

**代码简洁**：对比 DCL 的繁琐，枚举一行 `INSTANCE;` 搞定。

**天然线程安全**：枚举项通过 `static final` 修饰，在类加载的静态初始化阶段完成实例化，JVM 类加载机制保证同步。

**防止反序列化破坏**：枚举的反序列化机制不同于普通对象，不会通过反射或 Unsafe 创建新实例，从根本上杜绝破坏。

## CAS 无锁实现单例

```java
public class EventDispatcher {
    private static final AtomicReference<EventDispatcher> INSTANCE = new AtomicReference<>();
    
    private EventDispatcher() {}
    
    public static EventDispatcher getInstance() {
        for (;;) {
            EventDispatcher dispatcher = INSTANCE.get();
            if (dispatcher != null) {
                return dispatcher;
            }
            dispatcher = new EventDispatcher();
            if (INSTANCE.compareAndSet(null, dispatcher)) {
                return dispatcher;
            }
            // CAS失败，其他线程已创建，继续循环获取
        }
    }
}
```

**优势**：无锁。**劣势**：高并发下可能自旋并创建多个临时对象。

## 单例模式的破坏与防护

### 反射攻击

反射 `setAccessible(true)` 可突破私有构造创建新实例。

**防护**：构造函数加检查：

```java
private TaskScheduler() {
    if (instance != null) {
        throw new RuntimeException("单例对象禁止重复创建！");
    }
}
```

### 序列化攻击

序列化/反序列化通过 Unsafe 创建新对象，破坏单例。

**防护**：定义 `readResolve`：

```java
private Object readResolve() {
    return instance;   // ObjectInputStream 反序列化时检查该方法，用其返回值
}
```

## 单例实现方式对比与最佳实践

1. **首选枚举单例**：简洁，天然防御反射和序列化；
2. **需要延迟加载**：静态内部类；
3. **需要防止破坏**：DCL + readResolve + 构造函数检查；
4. **性能敏感**：CAS 实现（评估自旋开销）。
