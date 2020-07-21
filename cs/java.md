# 简单说一下Java

Java主要分为基础知识和高级特性。

- 基础知识：集合、文件、异常等。
- 高级特性：反射、注解、动态代理、类加载器等。

## 集合

容器类有两个根接口，Collection和Map。

- Collection：单个元素的集合，允许重复，有序。
  - List：有序、可重复。
    - ArrayList：基于数组实现，增删慢，查找快，线程不安全。
    - LinkedList：基于双向链表实现，增删快，查找慢，线程不安全。
    - CopyOnWriteArrayList：写时复制。
    - Vector：过时，基于数组实现，增删慢，查找快，用synchronized实现线程安全。
    - Stack：过时，继承于Vector。
  - Queue：先进先出队列，尾部添加，头部查看或删除。
    - LinkedList：基于链表实现。
    - ArrayDeque：基于循环数组实现。
    - PriorityQueue：优先级队列，基于堆实现。
    - ArrayBlockingQueue：基于数组实现，有界阻塞队列。
    - PriorityBlockingQueue：支持优先级排序，无界阻塞队列。
    - DelayQueue：支持延迟操作，无界阻塞队列。
    - SynchronousQueue：用于线程同步的阻塞队列。
    - LinkedBlockingQueue：基于链表实现，有界阻塞队列。
    - LinkedTransferQueue：基于链表实现，无界阻塞队列。
    - LinkedBlockingDeque：基于链表实现，双向阻塞队列。
  - Set：无序、不可重复。
    - HashSet：基于HashMap实现，无序。
    - LinkedHashSet：基于HashSet、LinkedList实现，双向链表记录顺序。
    - TreeSet：基于二叉树实现，可排序，实现SortedSet接口。
    - CopyOnWriteArraySet：写时复制。
- Map：键值对的集合，不允许key重复，无序。
  - HashMap：基于数组、链表、红黑树实现，可存null，线程不安全。
  - LinkedHashMap：基于HashTable实现，双向链表记录顺序。
  - TreeMap：基于二叉树实现，要求键实现Comparable接口或提供Comparator对象。
  - ConcurrentHashMap：线程安全，java7分段锁实现。
  - HashTable：过时，线程安全（synchronized）。

### ArrayList

- elementData：存储实际元素，有预留空间。
- size：实际元素的个数。
- modCount：并发修改异常，迭代过程中容器不能发生结构性变化。

编译器会把forEach语法转换为Iterator代码。用forEach进行删除时会报错，需要正确调用Iterator.remove，因为Iterator内部维护了cursor、lastRet、expectedModCount。

### Queue

- 抛异常的方法：add、remove、element。
- 不抛异常的方法：offer、poll、peek。

### Map

利用hash值提高比较性能。hash值默认是对象内存地址的计算。若想比较两个对象是否相等，需同时覆盖对象的hashCode方法和equals方法，并且其方法的返回值相同。

HashMap扩容的时候链表会倒置顺序，多线程会导致HashMap的Entry链表形成环形数据结构，一旦形成环形数据结构，Entry的next节点永远不为空，就会产生死循环获取Entry。比如两个线程，一个线程先完成倒置，另一个线程就会死循环。

ConcurrentHashMap，Java7每个段单独处理。Java8后来的线程会协助扩容。

### Collections

Collections以静态方法的方式提供了通用功能。

- 对容器接口对象进行操作。
  - 查找和替换。
  - 排序和调整顺序。
  - 添加和修改。
- 返回一个容器接口对象。
  - 适配器：将其他类型的数据转换为容器接口对象。空容器方法、单一对象方法、转换适配方法。
  - 装饰器：修饰一个给定容器接口对象，增加某种性质。写安全、类型安全、线程安全（通过synchronized包装方法）。

## 文件

### Java文件概述

- 流：处理二进制数据，输入流从提供者读取数据、输出流向目的地写入数据。
- 装饰器设计模式：缓冲装饰、8种基本类型和字符串装饰、压缩和解压缩装饰。
- Reader、Writer：处理文本数据。
- 随机读写文件：RandomAccessFile，适用于大小已知的记录组成的文件。
- File：文件元数据。
- NIO：缓冲区、通道、选择器，支持内存映射文件、文件加锁、自定义文件系统、非阻塞式IO、异步IO。
- 序列化和反序列化：对象状态持久化、网络远程通信中用于传递和返回对象。

### 二进制文件和字节流

- InputStream、OutputStream：基类，抽象类。
- FileInputStream、FileOutputStream：输入源和输出目标是文件的字节流。
- ByteArrayInputStream、ByteArrayOutputStream：输入源和输出目标是字节数组的字节流。
- DataInputStream、DataOutputStream：装饰类，按基本类型和字符串而非只是字节读写流。
- BufferedInputStream、BufferedOutputStream：装饰类，对输入输出流提供缓冲功能。

### 文本文件和字符流

- Reader、Writer：基类，抽象类。
- InputStreamReader、OutputStreamWriter：适配器类，将字节流转换为字符流。
- FileReader、FileWriter：输入源和输出目标是文件的字符流。
- CharArrayReader、CharArrayWriter：输入源和输出目标是char数组的字符流。
- StringReader、StringWriter：输入源和输出目标是String的字符流。
- BufferedReader、BufferedWrite：装饰类，对输入输出流提供缓冲功能，可按行读写。
- PrintWrite：装饰类，将基本类型和对象转换为其字符串形式输出的流。

## 异常

### 分类

- Throwable：分为Error、Exception。
- Error：运行错误，系统出现Error将退出进程，常见的有ThreadDeath。
- Exception：运行异常，异常处理机制可处理，分为RuntimeException、CheckedException。
  - RuntimeException：运行期间抛出的异常，常见的有NullPointerException、ClassCastException。
  - CheckedException：编译阶段强制捕获和处理此类异常，常见的有IOException、ClassNotFoundException。

### 处理方式

- 抛出异常：thronws（方法）、throw（代码）、系统自动抛出。
- 使用try catch捕获并处理异常。

## 反射

动态获取（程序运行时）类和对象的信息，动态调用对象的方法。

### 使用步骤

```java
// 1.获取类的Class对象 class.class obejct.getClass Class.forName(classpath)
Class clazz = Class.forName("com.example.Person");
// 2.获取类的所有方法
Method[] methods = clazz.getDeclaredMethods();
// 3.获取类的所有属性
Field[] fields = clazz.getDeclaredFields();
// 4.获取类的所有构造方法
Constructor[] constructors = clazz.getDeclaredConstructors();

// 创建对象方式1：newInstace
Person p1 = (Person) clszz.newInstance();
// 创建对象方式2：constructor
Constructor c = clazz.getDeclaredConstructor(String.class);
Person p2 = (Person) c.newInstance("张三");

// 调用对象方法：method.invoke
Method m = clazz.getDeclaredMethod("setName", String.class);
m.invoke(p2, "法外狂徒");
```

## 注解

设置程序中元素的关联信息和元数据信息，通过反射获取注解对象中的信息。

### 元注解

负责注解其他注解。

- @Target：注解所修饰的对象范围，TYPE、FILED、METHOD、PARAMETER、CONSTRUCTOR、LOCAL_VARIABLE、ANNOTATION_TYPE、PACKAGE、TYPE_PARAMETER、TYPE_USE。
- @Retention：注解被保留的级别，SOURCE、CLASS、RUNTIME。
- @Documented：可被javadoc工具文档化。
- @Inherited：可被注解继承。

## 动态代理

运行时动态创建一个类，在不修改原有类的基础上动态为通过该类获取的对象添加方法、修改行为。

- JDK动态代理：只代理接口，通过java.lang.reflect包中Proxy类和InvocationHandler接口实现。
- CGLib动态代理：可代理类，通过字节码处理框架ASM来实现，通过转换字节码生成新的类。

### 静态代理

创建一个代理类对实际对象进行方法封装，比如权限校验。

- 适配器模式：接口不同而导致无法一起工作，新类会实现新的接口，并持有旧类和旧接口。
- 装饰器模式：新类会实现旧接口并持有旧对象，新类中的方法会使用旧对象去调用旧方法，同时在该方法中添加新功能，旧对象是外部提供。
- 代理模式：代理类和被代理类实现的是同一个接口，且代理类持有被代理类的对象，意图是隐藏原类。
