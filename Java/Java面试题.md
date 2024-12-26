# 泛型的定义方式

? 是通配符, 表示未知类型
List\<?>, 表示List中的元素可以是任何类型
List\<? extends T>, 表示List中的元素类型可以是T类型及其子类的列表
List\<? super T>, 表示List中的元素类型可以是T类型及其父类的列表
List\<? extends String & Comparable\<T>>, 表示List中的元素类型, 必须同时继承String并且实现Cpmparable接口
在类, 接口, 方法的声明中, 可以定义一个或多个**类型变量**
比如下面的例子, Box\<T>定义了类型变量T, 表示Box可以接受任何类型
```java
public class Box<T> {
    private T value;

    public void setValue(T value) {
        this.value = value;
    }

    public T getValue() {
        return value;
    }
}
```

# 垃圾回收

## 判断对象是否可以回收的方法:
1. 引用计数器
2. 可达性分析 - 可以解决循环引用的问题

## 垃圾回收的流程
1. 标记 - 标记存活的对象
2. 删除 - 回收不存活的对象, 释放内存
3. 压缩 (可选) - 将活动内存复制到连续内存

## JVM的内存分成几个区域:
1. **方法区(jdk8可叫元空间)** - 存放类的元数据, 静态变量, 常量 - 回收废弃的类, 无用的常量
2. **堆** - 存放对象和数组 - **垃圾回收的主要区域** - 回收不用的对象
   分成新生代 (存放新生成的对象) 和老年代 (存放生命周期较长的对象) 两个区域
   新生代又分成Eden和Survivor两个区
3. 虚拟机栈和本地方法栈 - 不涉及回收
4. 程序计数器 - 不涉及回收

## 堆的回收算法
1. 新生代是 复制算法 
   新对象放到Eden区, 满了后, 存活的放到Survivor区, 过次GC后还存活的放到老年代
   **新生代满了就触发回收, 回收速度快但是频繁**
2. 老年代是 标记-清除 或 标记-整理
   前者, 标记存活, 清除不用的, 导致内存碎片化
   后者, 标记存活, 清除不用的时候, 复制存活的到连续内存
   **老年代满了触发, 因为对象多, 所以慢**

## 垃圾回收器
1. Parallel
   多线程回收新生代和老生代, 适合多核处理器
2. G1
   面向大的堆内存的回收器, 适合多核的服务器
3. CMS (Concurrent mark-sweep)
   低延迟回收器, 适合需要减少停顿时间的
4. Serial
   单线程垃圾回收, 单核cpu和小应用
   
## 垃圾回收优化
1. 避免Full GC
   Full GC是对整个堆和方法区进行一次垃圾回收
   Full GC会导致应用线程暂停, 如果老年代满了, 会触发Full GC, 容易引起暂停. 
2. 引起Full GC的原因
   **老年代满了 或者 向老年底复制对象是发现空间不足**
   **短时间加载大量的类, 导致方法区或老年代满了**
   创建了超大的对象, 老年代空间不够存放 (大对象会直接存放在老年代)
   **方法区(元空间)满了**
   **显式调用system.gc()**
   CMS (concurrent mark-sweep) 回收器在清理阶段没有足够的空间分配给新对象
3. 调优方法
   - 增大堆大小, 增大元空间大小
   - 提高新生代相对于老生代的空间的比值 (提高新生代)
   - 提高Survivor对于Eden区的比值 (增大Survivor区)
   - 增大对象晋升至老年代的年龄阈值 (默认15)
   - 避免频繁创建短生命周期的对象 (局部变量, 临时变量), 防止eden区满了, 晋升至老年代. 可以采用对象复用 (StringBuffer, HashMap), 避免创建不必要的对象 (可以用对象池ThreedPoolExecutor和StringBuffer代替创建新对象)
   - 避免创建大对象, 可将其拆分成多个小对象
   - 定期分析垃圾回收日志, 优化代码
   - 选择合适的垃圾回收器. G1减少晋升老年代, GMS适用老年代中长期存活的对象较多, ZGC适合低延迟敏感的应用
   
# Java的数据结构类型
## 基本数据结构
### 数组 Array
固定大小, 通过下标访问. 连续存储, 访问速度快
相关类 `Arrays` 提供了排序, 搜索, 转字符串格式输出 等使用方法.
### 链表 LinkedList
动态大小, 数据存储在节点中, 节点之间通过指针连接. 
适合频繁插入和删除数据时用.
相关类 java.util.LinkedList 是双向链表的实现. 因为它同时实现了Queue和Deque, 并且适合用来实现 栈 的逻辑, 所以自带了很多操作队列 和 栈 操作函数
队列方法:
offer 向队列末端添加元素
poll 从队列头部取出元素
peek 返回队列头部元素, 但是不删除它.

栈方法:
push 向栈压入一个元素
pop 弹出栈最上面的元素
peek 返回栈顶的元素但是不删除它.

队列和栈都有peek函数, 它返回的是队列的头部元素还是栈顶元素, 是由添加数据的操作决定的. 
用add添加, 说明是队列, peek就返回头部元素.
用push添加, 说明是栈, peek就返回栈顶元素.
### 队列 Queue
先进先出FIFO
适合做任务调度, 广度优先搜索
接口是 java.util.Queue
特殊形式有: 双端队列Deque, 优先队列PriorityQueue
**实现**它的类有 LinkedList, ArrayDeque, PriorityQueue
### 栈 Stack 
先进后出 LIFO. 
相关类 java.util.Stack性能不好, **不推荐使用**. 
推荐用Deque实现栈的逻辑.
### 双端队列 Deque
支持两端插入或删除的特殊队列
用来实现 **栈** 或者 **队列**
接口是 java.util.Deque

ArrayDeque类实现了Deque接口. 内置了函数, 实现了栈的功能. push()是压栈, pop()是出栈
还可以使用 LinkedList来做栈. 
这两个类用哪个做栈, 可以看出, 底层逻辑是入栈时, 向链表的头部插入数据. pop时从头部出栈. 模拟的 先入后出
```java
LinkedList<Integer> stack = new LinkedList<>();
// ArrayDeque<Integer> stack = new ArrayDeque<>();
stack.push(1);  
stack.push(2);  
stack.push(3);  
System.out.println("stack " + stack);  
stack.pop();  
System.out.println("stack " + stack);  
stack.pop();  
System.out.println("stack " + stack);
```

```
stack [3, 2, 1]
stack [2, 1]
stack [1]
```

### 哈希表 Hash Table
基于键值对的存储. 查询, 插入, 删除都适合
**相关**类是HashMap, Hashtable(内部用syncronized保证线程安全, 不常用了)

## 特殊的数据结构
### 树 Tree
分层的数据结构. 节点包含了数据和指针. 
`java.util.TreeMap` 和 `java.util.TreeSet` 实现了Tree的功能. 它们内部都是基于**红黑树**的`NavigableMap`实现. 
TreeMap存储**键值对**. 根据key, 维护了树的结构. 遍历输出时是根据key从小到大输出.
TreeSet存储的是**值**, 值是唯一的. 根据值的顺序维护了树的结构. 遍历输出时是将值从小到大输出.

### 图 Graph
由节点和路径组成. 分成有向和无向
Java没有线程的类实现图. 需要自己用 `Map<List>`来实现. 

### 堆 Heap
Java中的 `java.util.PriorityQueue` 实现了堆的数据结构. 默认是小顶堆. 可以自己编写比较器实现大顶堆. 
 PriorityQueue内部是由数组实现的, 会自动在内部维护堆的结构. PriorityQueue提供了add, remove, peek函数, 用来实现堆的操作. add会在数组末尾添加元素, 然后上浮到正确的位置. remove会交换堆顶和数组的最后一个元素, 然后堆顶元素会下潜到正确的位置. peek可以查看堆顶的元素. 

## 支持并发的数据结构 (线程安全)
### 并发列表
CopyOnWriteArrayList. 适合读多写少的场景
### 并发映射
ConcurrentHashMap.
### 并发队列
BlockingQueue接口. 适合生产者-消费者模型
**实现类**有: ArrayBlockingQueue, LinkedBlockingQueue, PriorityBlockingQueue

## 集合框架及其实现类
### 列表 List
有序可重复, 可以按索引访问
实现类是:
ArrayList 基于动态数组实现
LinkedList 基于链表实现
### 集合 Set
不允许重复元素
实现类:
HashSet 基于哈希表, 无序
LinkedHashSet 基于哈希表, 插入顺序排序
TreeSet 基于红黑树, 从小到大排序
### 映射 Map
存储键值对, 键唯一
实现类:
HashMap 无序, 性能优越
LinkedHashMap 按插入顺序排列
TreeMap 基于红黑树, 按照Key从小到大排列

### 队列Queue 与 双端队列 Deque
| **实现类**                 | **接口**          | **线程安全** | **无界/有界** | **主要特性**      | 适合的场景          | 补充说明                                                               |
| ----------------------- | --------------- | -------- | --------- | ------------- | -------------- | ------------------------------------------------------------------ |
| `LinkedList`            | `Queue`/`Deque` | 否        | 无界        | 基于链表，支持双端操作   | 普通的队列<br>可以模拟栈 |                                                                    |
| `PriorityQueue`         | `Queue`         | 否        | 无界        | 基于小顶堆，支持优先级排序 | 优先级队列          | poll返回的是队列中优先级最高的元素 (最小值). 通过Comparable接口或者Comparator的逻辑来确定优先级(大小) |
| `ArrayBlockingQueue`    | `Queue`         | 是        | 有界        | 阻塞队列，基于数组实现   | 阻塞             |                                                                    |
| `LinkedBlockingQueue`   | `Queue`         | 是        | 可有界       | 阻塞队列，基于链表实现   | 阻塞             |                                                                    |
| `ConcurrentLinkedQueue` | `Queue`         | 是        | 无界        | 非阻塞队列，基于链表实现  | 高并发场景          |                                                                    |
| `ArrayDeque`            | `Deque`         | 否        | 无界        | 高效的双端队列       | 普通的队列<br>可以模拟栈 |                                                                    |
| `LinkedBlockingDeque`   | `Deque`         | 是        | 可有界       | 阻塞双端队列，基于链表   | 阻塞             |                                                                    |
| `ConcurrentLinkedDeque` | `Deque`         | 是        | 无界        | 非阻塞双端队列，基于链表  | 高并发场景          |                                                                    |
这些类都提供了方便的操作方法, 
比如实现Queue的提供了: put, take, poll, offer
实现Deque的提供了addFirst, addLast, takeFirst, takeLast, pollFirst, pollLast

#### PriorityQueue
创建PriorityQueue时, 传入了自定义的Comparator, 
```java
PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a.value));
```
队列中的元素不一定是有序排列的, 但是可以保证高效的得到优先级最高的元素.
poll返回的是队列中优先级最高的元素 (最小值)
不要直接修改PriorityQueue中的元素, 会导致排序不准. 因为PriorityQueue不会自动维护顺序. 正确的做法是删除元素, 然后再放入修改后的元素.
# 什么是ConcurrentHashMap
ConcurrentHashMap是是线程安全的哈希表. 是java.util.concurrent包的一部分. 他允许多个线程并发读取和更新哈希表. 提高性能.
jdk1.7时, 原理是它将哈希表分成多个段(segment), 锁定时锁定某个分段. 不同线程操作不同的分段. 减少线程间的死锁或者等待(锁竞争), 提高性能.
每个segment内部是一个数组来存储数据, 跟HashMap相似. 数组的每个元素位置叫做一个桶. 
扩容时, 每个segment内部会扩容, 采用的是HashMap的扩容方式. 扩容判断也是每个segment内部判断
jdk1.8时, 不使用segment了, 而是使用CAS机制和Synchronized锁的结合控制并发. 

# HashMap扩容
HashMap是线程不安全的.
jdk1.8, hashmap的底层是 **数组 + 链表 + 红黑树**, 以前的版本就是 数组 + 链表
HashMap底层是使用了数组来存放数据. 用哈希算法按照key来获取存放的位置. 会争取散列分布. 如果hash值冲突了, 那么数组会存放链表的首节点地址或者红黑树的根节点 (数组这个位置叫做桶), 新的key, value存放在链表或者红黑树的节点上.

如果单个桶的节点数量超过8并且数组长度超过64, 则转成红黑树. 
如果数组长度小于等于64, 则不会转成红黑树, 而是直接扩容
如果节点数量小于等于6, 则会从红黑树退化成链表, 减少内存开销

如果元素数量超过哈希表负载因子 (元素数量与哈希表的容量的比率), 则需要扩容. 理论上是按照1.5倍扩容, 并向上取最接近的2的**幂次** (2, 4, 8, 16, 32, 64...). 
比如原容量是16, 则新容量是 16 * 1.5 = 24. 向上取最接近的2的幂次是32. 所以最新容量是32
之际操作中, 基本上就是原容量的2倍.

减小负载因子, 可以提高空余的空间, 减少哈希冲突, 查找删除操作也更快 (减少遍历的链表长度或者开放地址的探测次数)
增加负载因子, 可以减少空余的空间, 提高内存利用率, 减少扩容频率 (扩容涉及到元素的重新分布)

# Set和Map

| 特性     | Map                                                       | Set                                                                                   |
| ------ | --------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| 数据结构   | 键值对集合                                                     | 唯一元素集合                                                                                |
| 是否允许重复 | 键不可重复，值可重复                                                | 元素不可重复                                                                                |
| 访问方式   | 根据键访问值                                                    | 只能遍历访问                                                                                |
| 常用实现   | `HashMap`, `TreeMap`,<br>LinkedHashMap, ConcurrentHashMap | `HashSet`, `TreeSet`,<br>LinkedHashSet, ConcurrentSkipListSet<br>(前三者都是基于对应的Map类型实现的) |
| 适用场景   | 映射关系，快速查找键值对                                              | 唯一集合，快速排重                                                                             |
| 常见用例   | 配置文件加载(key=value)<br>实现缓存(value是缓存数据)<br>统计数据(单词计数)       | 唯一值集合(用户ID)<br>过滤重复请求<br>数据关系(求交集, 并集, 差集)                                            |

`Map` 和 `Set` 是 Java 中重要的集合接口，选择哪个要看实际需求是管理键值对还是操作唯一集合。


# List
List的几种实现

| 特性      | ArrayList   | LinkedList            | Vector                   | CopyOnWriteArrayList  |
| ------- | ----------- | --------------------- | ------------------------ | --------------------- |
| 查询性能    | 高（O(1)）     | 低（O(n)）               | 高（O(1)）                  | 高（O(1)）               |
| 插入/删除性能 | 中（O(n)）     | 高（O(1)）               | 中（O(n)）                  | 低（O(n)）               |
| 线程安全    | 否           | 否, 需要手动保证同步           | 是                        | 是                     |
| 内存开销    | 低           | 高                     | 低                        | 中                     |
| 适用场景    | 读多写少<br>当缓存 | 频繁插入或删除<br>任务队列, 链式操作 | 多线程下动态数组<br>兼容遗留代码, 并发环境 | 多线程下读多写少<br>配置列表, 黑名单 |

ArrayList底层是数组. 容量不足时ArrayList会扩容为原来的1.5倍
LinkedList支持队列或双端队列操作
Vector会扩容为原来的两倍
CopyOnWriteArrayList, 内部有两个数组, 一个用来读, 一个用来写入. 修改list内容时, 会将读的数组复制到另一个数组, 然后修改, 然后替换原来的读的数组. 适合多线程下读多写少的场景. 比如配置列表.

### **选择建议**

- **频繁查询：** 使用 `ArrayList` 或 `CopyOnWriteArrayList`（多线程）。
- **频繁插入/删除：** 使用 `LinkedList`。
- **多线程：** 使用 `CopyOnWriteArrayList`（读多写少）或 `Vector`（兼容旧代码）。
- **不可变集合：** 使用 `Collections.unmodifiableList` 或 `List.of`。

# 如何避免死锁

造成死锁必须同时达成的4个条件
1. 一个资源每次只能被一个线程使用
2. 线程被阻塞等待的时候, 不释放资源
3. 线程使用资源的时候, 只要没使用完, 就不能被强行剥夺资源
4. 多个线程形成头尾相连的循环等待

前三个是作为锁要符合的条件. 最后一个是关键. 为了避免死锁, 就要打破第四个条件, 不要出现循环等待. 要注意:
注意加锁顺序, 让每个线程按照同样的顺序加锁, 锁定后要设置超时时间, 要注意死锁检查.
按照顺序加锁的例子, 每个线程都是按照先lock1再lock2的顺序加锁的
```java
public class AvoidDeadlock {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void method1() {
        synchronized (lock1) {
            System.out.println("Thread 1: Holding lock 1...");
            try { Thread.sleep(10); } catch (InterruptedException e) {}
            synchronized (lock2) {
                System.out.println("Thread 1: Holding lock 1 & 2...");
            }
        }
    }

    public void method2() {
        synchronized (lock1) {
            System.out.println("Thread 2: Holding lock 1...");
            try { Thread.sleep(10); } catch (InterruptedException e) {}
            synchronized (lock2) {
                System.out.println("Thread 2: Holding lock 1 & 2...");
            }
        }
    }

    public static void main(String[] args) {
        AvoidDeadlock ad = new AvoidDeadlock();
        new Thread(() -> ad.method1()).start();
        new Thread(() -> ad.method2()).start();
    }
}
```
# Java的线程池
Java的线程池ThreadPoolExecutor来管理线程. 
线程有corePoolSize, maximumPoolSize, 队列(blockingQueue) 这几个关键点. 
线程分为核心线程和非核心线程. 
1. 线程池刚启动时, 线程数量是0, 随着来任务而创建新线程去处理.
2. 如果核心线程的数量小于corePoolSize的值, 那么来一个新任务, 就直接新建一个核心线程去处理.
3. 如果核心线程数量开始等于corePoolSize的值了, 那么新来的任务会被放入队列, 等待核心线程处理.
4. 如果这时任务队列满了, 那么就开始创建非核心线程处理队列的任务. 如果这些非核心线程处理完任务后, 没有新的任务给它, 那么超过超时时间后, 就会被销毁
5. 如果队列满了, 线程数量也达到了maximumPoolSize的值, 那么新任务就会根据拒绝策略处理, 比如抛出异常 , 丢弃任务, 或者运行自定义的处理逻辑.

核心线程正常的情况下, 不会被销毁, 除非:
1. 线程池设置了allowCoreThreadTimeOut(true), 那么在超时后核心线程也会被销毁
2. 核心线程运行异常
3. 线程池调用shutdown()或者shutdownNow()函数,关闭所有线程

**任务队列分成几类**
阻塞队列:
有界阻塞队列. ArrayBlockingQueue, LinkedBlockingQueue, PriorityBlockingQueue. 区别是, 
Array的是有界的队列, 是用数组实现的. 有长度限制. 防止内存溢出的问题. 
Linked是没有边界的(也可以设置边界数量), 可以大量缓存任务. 但是要注意内存溢出的问题. 
Priority是有优先级的. 任务要实现Comparable接口或者提供Comparator. 
无阻塞的队列:
SynchronousQueue, 是一个无界的直接交付的队列, 没有缓冲. 每一个插入必须等待一个删除操作, 每一个删除也需要等待一个插入操作. 

如果任务数量是有限的而且可以预见的, 就用有界队列
如果任务派发和处理的速度匹配, 而且追求低延迟高并发, 就用SynchronousQueue队列
如果任务有优先级区分, 就用Priority的

# ThreadLocal对象
在Spring中，`ThreadLocal`对象是一种用于确保线程安全的技术。它允许每个线程保存其独立的变量副本，从而在多线程环境中实现线程隔离，避免并发问题。

### `ThreadLocal` 的作用
`ThreadLocal` 提供了一种简洁的方式来实现线程私有数据。每个线程可以通过 `ThreadLocal` 对象维护自己的变量副本，而无需担心其他线程的干扰。这在处理非线程安全的资源（如数据库连接、会话等）时特别有用。

### `ThreadLocal`的用法
以下是一个简单的示例，展示了如何使用`ThreadLocal`在每个线程中保存独立的数据：

```java
public class ThreadLocalExample {
    private static ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);

    public static void main(String[] args) {
        Runnable task = () -> {
            int value = threadLocal.get();
            threadLocal.set(value + 1);
            System.out.println(Thread.currentThread().getName() + ": " + threadLocal.get());
        };

        Thread thread1 = new Thread(task);
        Thread thread2 = new Thread(task);

        thread1.start();
        thread2.start();
    }
}
```

在这个示例中，每个线程都有自己的 `ThreadLocal` 变量副本，因此线程之间不会互相干扰。

### 在Spring中的应用
在Spring中，`ThreadLocal`通常用于管理线程安全的资源，例如数据库事务上下文、会话信息等。Spring为此提供了一些封装和扩展，以便更方便地使用`ThreadLocal`。

#### 示例：使用 `ThreadLocal` 管理数据库事务
在Spring中，可以使用 `ThreadLocal` 管理数据库事务上下文，以确保每个线程都有独立的事务上下文。例如：

```java
public class TransactionManager {
    private static ThreadLocal<Transaction> transactionContext = new ThreadLocal<>();

    public static void beginTransaction() {
        transactionContext.set(new Transaction());
    }

    public static void commitTransaction() {
        Transaction transaction = transactionContext.get();
        if (transaction != null) {
            transaction.commit();
            transactionContext.remove();
        }
    }

    public static void rollbackTransaction() {
        Transaction transaction = transactionContext.get();
        if (transaction != null) {
            transaction.rollback();
            transactionContext.remove();
        }
    }
}
```

### `ThreadLocal` 和 Spring 的结合
Spring提供了一些内置的机制和组件，简化了`ThreadLocal`的使用。例如，Spring的 `RequestContextHolder` 通过 `ThreadLocal` 存储当前HTTP请求的上下文信息，从而实现线程安全的请求处理。

#### 示例：获取当前请求
```java
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;

public class RequestUtils {
    public static HttpServletRequest getCurrentRequest() {
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        return attributes != null ? attributes.getRequest() : null;
    }
}
```

### 小结
`ThreadLocal` 在Spring中的应用非常广泛，能够有效地解决多线程环境下的数据隔离问题。通过结合Spring的组件和框架，可以更加方便地管理线程私有数据，确保线程安全。

# 线程的生命周期

线程的生命周期从 **新建状态** 开始，经过 **可运行状态**、**阻塞状态**、**等待状态**、**定时等待状态**，最终进入 **终止状态**。

### **线程状态转移**

- **新建状态**: 对象已经被创建, 但是还没执行`thread.start()`. 
- **从新建状态到可运行状态**：调用 `thread.start()` 方法，线程进入 **可运行状态**，等待线程调度器分配 CPU 时间。
- **从可运行状态到阻塞状态**：当线程尝试获取一个已经被其他线程持有的锁时，它会进入 **阻塞状态**。
- **从可运行状态到等待状态**：线程调用 `wait()` 或 `join()` 时，进入 **等待状态**。
- **从等待状态到可运行状态**：线程在被唤醒后，进入 **可运行状态**。唤醒可以通过调用 `notify()`、`notifyAll()`，或者当 `join()`, `sleep()` 的等待时间结束后发生。
- **从定时等待状态到可运行状态**：线程在超时后会回到 **可运行状态**，继续执行。
- **从终止状态到不可返回**：线程结束后进入 **终止状态**，它不能再回到任何其他状态。