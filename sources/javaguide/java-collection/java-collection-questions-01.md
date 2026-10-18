---
来源站点: javaguide.cn（JavaGuide）
原文链接: https://javaguide.cn/java/collection/java-collection-questions-01.html
抓取日期: 2026-10-18
所属知识域: 域1-Java核心与编码工程化
板块: Java 集合
---

# Java集合常见面试题总结(上)

> 覆盖集合概述、List、Set、Queue 四大部分，共 23 个面试问题。所有代码示例与输出结果均已保留。

## 集合概述

### Java 集合概览

Java 集合（容器）主要由两大接口派生而来：

- **`Collection` 接口**：主要用于存放单一元素。其下有三个主要子接口：`List`、`Set`、`Queue`。
- **`Map` 接口**：主要用于存放键值对。

### ⭐️ 说说 List, Set, Queue, Map 四者的区别？

- `List`（对付顺序的好帮手）: 存储的元素有序、可重复。
- `Set`（注重独一无二的性质）: 存储的元素不可重复。
- `Queue`（实现排队功能的叫号机）: 按特定排队规则确定先后顺序，元素有序、可重复。
- `Map`（用 key 来搜索的专家）: 键值对存储，key 无序、不可重复，value 无序、可重复。注意"无序"指 `HashMap` 这类实现——键值对之间没有显式关联顺序；`LinkedHashMap` 和 `TreeMap` 通过双向链表或红黑树维护顺序。

### 集合框架底层数据结构总结

**Collection 接口下：**

- **List**
  - `ArrayList`：`Object[]` 数组
  - `Vector`：`Object[]` 数组
  - `LinkedList`：双向链表（JDK1.6 之前为循环链表，JDK1.7 取消）
- **Set**
  - `HashSet`（无序，唯一）: 基于 `HashMap` 实现
  - `LinkedHashSet`: `HashSet` 子类，内部通过 `LinkedHashMap` 实现
  - `TreeSet`（有序，唯一）: 红黑树（自平衡排序二叉树）
- **Queue**
  - `PriorityQueue`: `Object[]` 数组实现小顶堆
  - `DelayQueue`: `PriorityQueue`
  - `ArrayDeque`: 可扩容动态双向数组

**Map 接口下：**

- `HashMap`：JDK1.8 之前数组+链表（拉链法解决冲突）；JDK1.8 以后当链表长度大于阈值（默认 8）时将链表转化为红黑树（转换前会判断，如果当前数组长度小于 64，则先进行数组扩容而不是转红黑树）
- `LinkedHashMap`：继承 `HashMap`，在拉链式散列结构基础上增加一条双向链表，可保持插入顺序，并实现访问顺序逻辑（LRU 场景）
- `Hashtable`：数组+链表
- `TreeMap`：红黑树

### 如何选用集合？

- 需要根据键获取元素值时选用 `Map`：排序选 `TreeMap`，不排序选 `HashMap`，线程安全选 `ConcurrentHashMap`。
- 只需存放元素值时选 `Collection`：保证唯一选 `Set`（`TreeSet`/`HashSet`），不保证选 `List`（`ArrayList`/`LinkedList`）。

### 为什么要使用集合？

数组是存储一组类型相同数据的基本容器，但存在不足：实际开发中存储的数据类型多样、数量不确定。相比数组，Java 集合的优势：**大小可变、支持泛型、具有内建算法**，提高了数据存储和处理的灵活性。

## List

### ⭐️ ArrayList 和 Array（数组）的区别？

`ArrayList` 内部基于动态数组实现，比 `Array`（静态数组）更灵活：

- `ArrayList` 会根据实际存储元素动态扩容，也可通过 `trimToSize()` 主动缩小底层数组；`Array` 创建后长度固定。
- `ArrayList` 允许使用泛型确保类型安全，`Array` 不可以。
- `ArrayList` 只能存储对象（基本类型需用包装类）；`Array` 可直接存储基本类型和对象。
- `ArrayList` 支持插入、删除、遍历等操作并提供丰富 API；`Array` 只能按下标访问。
- `ArrayList` 创建时不需要指定大小；`Array` 创建时必须指定大小。

### ArrayList 和 Vector 的区别?（了解即可）

- `ArrayList` 是 `List` 的主要实现类，底层 `Object[]`，适用于频繁查找，线程不安全。
- `Vector` 是 `List` 的古老实现类，底层 `Object[]`，线程安全。

### Vector 和 Stack 的区别?（了解即可）

两者都是线程安全的，都使用 `synchronized` 同步。`Stack` 继承自 `Vector`，是后进先出的栈。随着 Java 并发编程发展，`Vector` 和 `Stack` 已被淘汰，推荐使用并发集合类（`ConcurrentHashMap`、`CopyOnWriteArrayList`）。

### ArrayList 可以添加 null 值吗？

可以存储任何类型对象包括 `null`，但不建议添加 `null`（无意义、难维护、易忘判空导致 NPE）。

### ⭐️ ArrayList 插入和删除元素的时间复杂度？

**插入：**

- 头部插入：所有元素向后移动一位，O(n)。
- 尾部插入：容量未达极限时 O(1)；容量达到极限需先 O(n) 扩容复制，再 O(1) 添加。
- 指定位置插入：目标位置之后所有元素后移，平均移动 n/2 个，O(n)。

**删除：**

- 头部删除：所有元素向前移动，O(n)。
- 尾部删除：O(1)。
- 指定位置删除：平均移动 n/2 个元素，O(n)。

### ⭐️ LinkedList 插入和删除元素的时间复杂度？

- 头部插入/删除：只需修改头结点指针，O(1)。
- 尾部插入/删除：只需修改尾结点指针，O(1)。
- 指定位置插入/删除：需先移动到指定位置（有头尾指针，从较近端出发，平均遍历 n/4），O(n)。

### LinkedList 为什么不能实现 RandomAccess 接口？

`RandomAccess` 是标记接口，表明实现类支持随机访问（通过索引快速访问）。`LinkedList` 底层是链表，内存地址不连续，只能通过指针定位，不支持随机快速访问。

### ⭐️ ArrayList 与 LinkedList 区别？

- **线程安全**：两者都不同步，不保证线程安全。
- **底层数据结构**：`ArrayList` 用 `Object` 数组；`LinkedList` 用双向链表。
- **插入删除受位置影响**：`ArrayList` 尾部插入 O(1)，指定位置 O(n)；`LinkedList` 头尾 O(1)，指定位置 O(n)。
- **随机访问**：`LinkedList` 不支持高效随机访问；`ArrayList`（实现 `RandomAccess`）支持。
- **内存空间**：`ArrayList` 在列表结尾预留一定容量；`LinkedList` 每个元素需额外存放直接后继/前驱指针。

**项目中一般不会使用 `LinkedList`，需要它的场景几乎都可以用 `ArrayList` 代替，性能通常更好。** `LinkedList` 作者约书亚·布洛克（Josh Bloch）自己都说从来不会使用 `LinkedList`。不要下意识认为链表就最适合增删——`LinkedList` 仅头尾增删近似 O(1)，其他情况平均 O(n)。

**RandomAccess 接口应用**：`Collections.binarySearch()` 中判断传入 list 是否 `RandomAccess` 实例，是则调用 `indexedBinarySearch()`，否则调用 `iteratorBinarySearch()`。

### ⭐️ 说一说 ArrayList 的扩容机制

详见源码分析专题（arraylist-source-code.html#arraylist-扩容机制分析）。

### ⭐️ 集合中的 fail-fast 和 fail-safe 是什么？

**fail-fast（快速失败）**：针对可能发生的异常提前表明故障并停止运行，通过尽早发现和停止错误降低故障系统级联风险。`java.util` 包下大部分集合（`ArrayList`、`HashMap`）不支持线程安全，通过维护 `modCount` 记录修改次数，迭代期间比对 `expectedModCount` 和 `modCount` 是否一致来判断是否存在并发操作：

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

**fail-safe（安全失败）**：即使面对意外情况也能恢复并继续运行。经典实现是 `CopyOnWriteArrayList`，通过写时复制（Copy-On-Write）保证修改操作时复制出一份快照，基于快照完成增删后将底层数组引用指向新数组，避免迭代时被并发修改干扰。缺点是遍历时无法获得实时结果。

## Set

### Comparable 和 Comparator 的区别

- `Comparable` 接口出自 `java.lang` 包，有 `compareTo(Object obj)` 方法，是对象自身的自然排序。
- `Comparator` 接口出自 `java.util` 包，有 `compare(Object obj1, Object obj2)` 方法，是外部定制排序。

需要对集合使用自定义排序时，重写 `compareTo()` 或使用 `Comparator`。同一集合需要两种排序方式时，可同时实现 `Comparable` 和使用两个 `Comparator`。

### 无序性和不可重复性的含义是什么

- **无序性不等于随机性**：指存储的数据在底层数组中并非按数组索引顺序添加，而是根据数据的哈希值决定。
- **不可重复性**：添加的元素按 `equals()` 判断返回 false，需要同时重写 `equals()` 和 `hashCode()`。

### 比较 HashSet、LinkedHashSet 和 TreeSet 三者的异同

都是 `Set` 接口实现类，都能保证元素唯一，都不是线程安全的。区别在底层数据结构：

- `HashSet`：哈希表（基于 `HashMap`），用于不需要保证插入和取出顺序的场景。
- `LinkedHashSet`：链表和哈希表，插入取出顺序满足 FIFO。
- `TreeSet`：红黑树，元素有序，支持自然排序和定制排序。

## Queue

### Queue 与 Deque 的区别

`Queue` 是单端队列，一端插入、另一端删除，一般遵循 FIFO。根据操作失败后处理方式分为两类方法：

**Queue 接口：**

| 操作 | 抛出异常 | 返回特殊值 |
| --- | --- | --- |
| 插入队尾 | add(E e) | offer(E e) |
| 删除队首 | remove() | poll() |
| 查询队首 | element() | peek() |

`Deque` 是双端队列，两端均可插入删除，增加了队首队尾操作，还可通过 `push()`/`pop()` 模拟栈：

| 操作 | 抛出异常 | 返回特殊值 |
| --- | --- | --- |
| 插入队首 | addFirst(E e) | offerFirst(E e) |
| 插入队尾 | addLast(E e) | offerLast(E e) |
| 删除队首 | removeFirst() | pollFirst() |
| 删除队尾 | removeLast() | pollLast() |
| 查询队首 | getFirst() | peekFirst() |
| 查询队尾 | getLast() | peekLast() |

### ArrayDeque 与 LinkedList 的区别

都实现了 `Deque` 接口：

- `ArrayDeque` 基于可变长数组和双指针；`LinkedList` 通过链表实现。
- `ArrayDeque` 不支持存储 `null`；`LinkedList` 支持。
- `ArrayDeque` 于 JDK1.6 引入；`LinkedList` 早在 JDK1.2 存在。
- `ArrayDeque` 插入可能有扩容（均摊 O(1)）；`LinkedList` 不需扩容但每次插入需申请新堆空间，均摊性能更慢。

**从性能角度，选用 `ArrayDeque` 实现队列比 `LinkedList` 更好，也可用于实现栈。**

### 说一说 PriorityQueue

- 利用二叉堆实现，底层使用可变长数组存储。
- 通过堆元素上浮下沉，O(logn) 完成插入和删除堆顶。
- 非线程安全，不支持 `null`。未提供 `Comparator` 时元素需实现 `Comparable`。
- 默认小顶堆，可传 `Comparator` 自定义优先级。

典型面试题：堆排序、求第 K 大的数、带权图的遍历。

### 什么是 BlockingQueue？

阻塞队列接口，继承自 `Queue`。为插入和移除操作提供抛出异常、返回特殊值、持续阻塞、超时等待四种处理方式。`take()` 在队列为空时阻塞，`put()` 在队列已满时阻塞。常用于生产者-消费者模型。

### BlockingQueue 的实现类有哪些？

1. `ArrayBlockingQueue`：数组实现的有界阻塞队列，创建时指定容量，支持公平/非公平锁。
2. `LinkedBlockingQueue`：单向链表实现的可选有界阻塞队列，默认 `Integer.MAX_VALUE`（无界），仅支持非公平锁。
3. `PriorityBlockingQueue`：支持优先级排序的无界阻塞队列。
4. `SynchronousQueue`：不存储元素的同步队列，每个插入操作必须等待对应删除操作。
5. `DelayQueue`：延迟队列，元素到达指定延迟时间才能出队。

### ⭐️ ArrayBlockingQueue 和 LinkedBlockingQueue 有什么区别？

都是线程安全的阻塞队列：

- **底层实现**：`ArrayBlockingQueue` 基于数组；`LinkedBlockingQueue` 基于链表。
- **是否有界**：`ArrayBlockingQueue` 有界，必须指定容量；`LinkedBlockingQueue` 可不指定（默认无界），也可指定为有界。
- **锁是否分离**：`ArrayBlockingQueue` 生产和消费用同一个锁；`LinkedBlockingQueue` 锁分离（`putLock`/`takeLock`），防止生产者和消费者之间的锁争夺。
- **内存占用**：`ArrayBlockingQueue` 提前分配数组内存（往往比实际所需更大）；`LinkedBlockingQueue` 随元素增加逐渐占用内存。

## 数据结构延伸阅读

- 线性数据结构详解：理解数组、链表、栈、队列和 `ArrayList`、`LinkedList`、`ArrayDeque` 的关系。
- 哈希表面试题总结：理解哈希冲突、扩容和 `HashMap` 底层思想。
- 红黑树详解：理解 `TreeMap`、`TreeSet` 及 `HashMap` 树化。
- 堆详解：理解 `PriorityQueue` 底层结构和 Top K 题型。
