# 简单说一下中间件

Java框架或多或少用到了Java高级特性，如下。

- Jackson：利用反射、注解，实现通用的序列化机制。用注解定制输出字段的名称、格式，利用反射获取注解信息输出序列化。
- Spring MVC、Jersey：利用反射、注解，实现序列化处理Web通信内容。
- AOP：利用反射、注解、动态代理，实现通用代码与业务代码分离。
- SPI：利用类加载器，实现面向接口编程。
- Tomcat、OSGI、Java 9 module：利用类加载器，不同的ClassLoader可以加载相同的类但相互隔离。
- JSP：利用类加载器，实现修改代码不用重启即可生效。起一个后台线程定时重复加载类。
- Spring、Guice：利用反射、注解、动态代理、类加载器，实现对象管理容器。

## Java 9 Module

每个模块使用自定义ClassLoader实现动态更新。

## OSGI

Open Service Gateway Initiative，每个模块使用自定义ClassLoader实现动态更新。

## Lombok

通过JSR 269的api，在编译期间的Annotation Process阶段根据不同的Lombok注解调用不同的Handler修改了抽象语法树，达到增强字节码的效果。

## JDBC

Java Database Connectivity是Java语言中提供的访问关系型数据的接口。javax.sql包添加了数据源扩展（DataSource）、连接池、ResultSet扩展、分布式扩展。

- 建立数据源连接：在JDBC 1.0规范中使用DriverManager，在JDBC 2.0规范中使用DataSource。
- 执行SQL语句：通过Statement接口执行查询和更新操作。
- 检索SQL执行结果：通过ResultSet接口检索执行结果。
- 关闭连接：关闭上述各个接口。

```java
try {
    // 建立数据源连接 1
    Class.forName("org.hsqldb.jdbcDriver");
    Connection connection = DriverManager.getConnection("jdbc:hsqldb:mem:mybatis"，"sa"，"");

    /** 建立数据源连接 2
     *  DataSourceFactory dsf = new UnpooledDataSourceFactory();
     *  Properties properties = new Properties();
     *  InputStream configStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("database.properties");
     *  properties.load(configStream());
     *  dsf.setProperties(properties);
     *  DataSource dataSource = dsf.getDataSource();
     *  Connection connection = dataSource.getConnection();
     */

    // 执行SQL语句
    Statement statement = connection.createStatement();

    // 检索SQL执行结果
    ResultSet resultSet = statement.executeQuery("select * from user");
    ResultSetMetaData metaData = resultSet.getMetaData();
    int columnCount = metaData.getColumnCount();
    while (resultSet.next()) {
        for (int i = 1; i <= columnCount; i++) {
            String columnName = metaData.getColumnName(i);
            String columnValue = resultSet.getString(columnName);
        }
    }

    // 关闭连接
    IOUtils.closeQuietly(statement);
    IOUtils.closeQuietly(connection);
} catch (Exception e) {
    e.printStackTrace();
}
```

## MyBatis

### MyBatis简介

MyBatis是一款在持久层使用的SQL映射框架，可以将SQL语句单独写在XML配置文件中，或者使用带有注解的Mapper映射类来完成数据库记录到Java实体的映射。属于半自动的ORM框架。

- 消除了大量的JDBC冗余代码，包括参数设置、结果集封装等。
- SQL语句可控制，方便查询优化，使用更加灵活。
- 学习成本比较低，对于新用户能够快速学习使用。
- 提供了与主流IoC框架Spring的集成支持。
- 引入缓存机制，提供了与第三方缓存类库的集成支持。

### MyBatis组件

- Configuration：用于描述MyBatis主配置文件信息，MyBatis框架在启动时会加载主配置文件，将配置信息转换为Configuration对象。
- SqlSession：面向用户的API，是MyBatis与数据库交互的接口。
- Executor：SQL执行器，用于和数据库交。SqlSession可以理解为Executor组件的外观，真正执行SQL的是Executor组件。
- MappedStatement：用于描述SQL配置信息，MyBatis框架启动时，XML文件或注解配置的SQL信息会被转换为MappedStatement对象注册到Configuration组件中。
- StatementHandler：封装了对JDBC中Statement对象的操作，包括为Statement参数占位符设置值，通过Statement对象执行SQL语句。
- TypeHandler：类型处理器，用于Java类型与JDBC类型之间的转换。
- ParameterHandler：用于处理SQL中的参数占位符，为参数占位符设置值。
- ResultSetHandler：封装了对ResultSet对象的处理逻辑，将结果集转换为Java实体对象。

## HikariCP

- 精简字节码：代码量少，只有130Kb，使用Javassist生成动态代理类进行字节码精简。
- FastList：只实现必要的接口，去掉越界检查，删除从尾部扫描。
- ConcurrentBag：无锁设计，ThreadLocal缓存，队列窃取，直接切换。

## Tomcat

每个Web应用使用自己的ClassLoader。

## Spring

利用IoC和AOP实现。

- IoC：Inversion of control，控制反转。控制权由对象本身转向容器；用注解修饰依赖类，通过对象管理容器类获取对象时自动创建依赖类。
- AOP：Aspect Oriented Programming，面向切面编程。创建一个AOP注解，约定被切的方法，容器类初始化类时通过反射解析注解进行动态代理。

### Spring模块组成

- 核心容器层：Spring-Beans、Spring-Core、Spring-Context、SpEL。
- 数据访问层：JDBC、ORM、OXM、JMS、事务处理。
- Web应用层：Web、Web-MVC、Web-Socket、Web-Portlet。
- 其他模块：AOP、Aspects、Instrumentation、Messaging、Test。

### Spring容器基础概念

- BeanDefinition：描述Spring Bean的配置信息，Spring配置Bean的方式通常有3种：XML配置文件、Java注解、Java Config方式。
- BeanDefinitionRegistry：BeanDefiniton的容器，所有的Bean配置解析后生成的BeanDefinition对象都会注册到BeanDefinitionRegistry对象中。
- BeanFactory：Spring的Bean工厂，负责Bean的创建及属性注入。所有的单例Bean都会注册到BeanFactory容器中。
- BeanFacotryPostProcessor：Spring提供的扩展机制，用于在所有的Bean配置信息解析完成后修改Bean工厂信息。
- ImportBeanDefinitionRegistrar：作用于Spring解析Bean的配置阶段，当解析@Configuration注解时，通过此接口的实现类向BeanDefinitionRegistry容器添加额外的BeanDefinition对象。
- BeanPostProcessor：Bean的后置处理器，在Bean初始化方法调用前后执行定义的拦截逻辑。
- ClassPathBeanDefinitionScanner：BeanDefinition扫描器，将指定包下的Class信息转换为BeanDefinition对象并注册到BeanDefinitionRegistry容器中。
- FactoryBean：Spring中的工厂Bean，处理Spring中配置较为复杂或者由动态代理生成的Bean实例。通过Bean名称获取FactoryBean实例时，获取的是FacotryBean对象的getObject()方法返回的实例。

### Spring容器启动过程

- 对所有Bean的配置信息进行解析，将Bean的配置信息转换为BeanDefinition对象，并注册到BeanDefinitionRegistry容器中。
- 从BeanDefinitionRegistry容器中获取实现了BeanFactoryPostProcessor接口的Bean定义，然后实例化Bean，调用postProcessBeanFactory()方法，可对Bean工厂的信息进行修改。
- 根据BeanDefinitionRegister容器中的BeanDefinition对象实例化所有的单例Bean，并对Bean的属性进行填充。
- 执行所有实现了BeanPostProcessor接口的Bean的postProcessBeforeInitialization()方法，可对原始的Bean进行包装。
- 执行Bean的初始化方法，初始化方法包括配置Bean的init-method方法，实现InitializingBean接口重写的afterPropertiesSet()方法。
- 执行所有实现了BeanPostProcessor接口的Bean的postProcessAfterInitialization()方法。

### Spring Bean

- 作用域：Singleton（单例）、Prototype（原型）、Request（HTTP请求级别）、Session（HTTP会话级别）、Global Session（HTTP全局会话）。
- 依赖注入：构造器、set方法、静态工厂、实例工厂。
- 自动装配：no、byName、byType、constructor、autodetect。

### Spring MVC

- 客户端发起HTTP请求：客户端将请求提交到DispatcherServlet。
- 寻找处理器：DispatcherServlet查询HandlerMapping，找到处理该请求的Controller。
- 调用处理器：DispatcherServlet将请求提交到Controller。
- 调用业务处理逻辑并返回结果：Controller调用业务处理逻辑后，返回ModelAndView。
- 处理视图映射并返回模型：DispatcherServlet查询ViewResoler视图解析器，找到ModelAndView指定的视图。
- HTTP响应：视图负责将结果渲染和展示在客户端浏览器上。

## Spring Boot

### 什么是spring boot

- 约定优于配置。
- 用来简化spring应用开发。
- 简化maven配置，starter自动化配置。
- 嵌入的servlet容器，无需部署war文件。
- main方法启动应用。

### 核心注解

启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

- @SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。
- @EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。
- @ComponentScan：Spring组件扫描。

### 自动配置

spring程序main方法中添加@EnableAutoConfiguration，会自动去maven中读取每个starter中的spring.factories文件，该文件里配置了所有需要被创建spring容器中的bean。

### starters

Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成 Spring 及其他技术，而不需要到处找示例代码和依赖包。如你想使用 Spring JPA 访问数据库，只要加入 spring-boot-starter-data-jpa 启动器依赖就能使用了。

## Spring Cloud

- 服务发现：Eureka
- 协议调用：Feign
- 负载均衡：Zuul
- 集群容错：Hystrix断路器
- 分布式跟踪：Sleuth日志跟踪

## SPI

SPI（Service Provider Interface，服务提供者接口）是面向接口编程，服务规则提供者在JRE的核心API里定义服务访问接口，具体实现由其他开发商提供。

### SPI原理

Java核心API（如rt.jar）是由BootStrapClassLoader加载，用户提供的Jar包是由AppClassLoader加载，如果一个类由类加载器加载，那么其依赖的类也由相同的类加器加载。

用来搜索开发商提供的spi扩展实现类的API类（ServiceLoader）使用ContextClassLoader加载。

ContextClassLoader会获取当前线程上下文加载器，也就是会获取到AppClassLoader。

### 数据库驱动接口示例

比如规范制定者在rt.jar包里定义了数据库的驱动接口java.sql.Driver，那么MySQL开发商会在驱动包的MEATA-INF/services/下建立java.sql.Driver的文件，文件内容是驱动接口的实现类。

## Dubbo

### 增强SPI原理

Dubbo的扩展点加载机制基于SPI而来的，解决了SPI的以下问题：

- SPI会一次性实例化扩展点的所有实现，如果有些扩展实现初始化很耗时，但又没用上，那么加载就会很浪费资源。
- 如果扩展点加载失败，不会向用户通知具体异常。如果RubyScriptEngine因为所依赖的jruby.jar不存在而导致类加载失败，当用户执行Ruby脚本时报空指针异常，而不是RubyScriptEngine不存在。
- 增强了对扩展点IoC和AOP的支持，一个扩展点可直接使用setter()方法注入其他扩展点，也可以对扩展点使用Wrapper类进行功能增强。

### 适配器原理

- Dubbo会给每个SPI扩展接口动态生成一个对应的适配器类，根据参数来选择使用增强SPI实现。
- 比如扩展接口ProxyFactory的适配器类为ProxyFactory$Adaptive，其根据参数proxy来决定使用JdkProxyFactory、JavassistProxyFactory中的一个做代理工厂。
- 比如扩展接口Registry的适配器类为Register$Adaptive，其根据参数Register来决定使用ZooKeeperRegister、RedisRegister、MulticastRegister、DubboRegister中的一个做服务注册中心。

### 服务架构

- Provider：服务提供者，启动时将服务注册到Register，暴露提供的服务。
- Consumer：服务消费者，启动时去Register订阅自己需要的服务地址，通过路由规则和负载均衡后，rpc远程调用服务提供者提供的服务。
- Register：服务注册与发现。
- Monitor：监控中心，统计调用次数、调用时间，Provider、Consumer每分钟上报数据。

### 分层架构

- Service和Config：为api接口层，服务提供者用ServiceConfig发布服务，服务消费者用ReferenceConfig进行代理消费。
- 其他各层为SPI层：服务提供者接口是组件化，可替换的。
- Proxy：服务代理层，JavassistProxyFactory、JdkProxyFactory。
- Register：服务注册中心层，ZooKeeperRegister、RedisRegister、MulticastRegister、DubboRegister。
- Cluster：路由层，负载均衡有随机、轮询、最小活跃数、一致性hash，集群容错有失败重试、快速失败、失败安全、失败自动恢复、并行调用。
- Monitor：监控层。
- Protocol：远程调用协议层，RegisterProtocol、DubboProtocol、InjvmProtocol。
- Exchange：信息交换层，封装请求响应模式，同步转异步。
- Transport：网络传输层，抽象为统一接口，NettyTransporter、MinaTransporter。
- Serialize：数据序列化层，DubboSerialization、FastJsonSerialization、Hessian2Serialization、JavaSerialization。

### 服务消费者的请求cmpets

选择服务提供者地址（Cluster） -> 监控（Monitor） -> 协议选择（Protocol） -> 请求响应模式（Exchange） -> 传输方式（Transport） -> 数据序列化（Serialize）

### 服务提供者的处理

Netty完成io线程的连接（Serialize、Transport） -> 线程委派 -> 处理请求响应模式（Exchange）-> 处理协议（Protocol） -> 调用服务实现类（Proxy）

### 线程模型

boss线程池接受客户端的链接请求，并把完成TPC三次握手的连接分发给worker处理线程池，统称为io线程池。

- all：所有消息都委派到业务线程池，包括请求事件、响应事件、连接事件、断开事件、心跳事件。
- direct：所有消息都不委派。
- message：请求、响应消息委派到业务线程池，其他在io线程池。
- execution：请求消息委派到业务线程池，其他在io线程池。
- connection：io线程逐个处理连接、断开，其他委派到业务线程池。

### 集群容错

- 失败重试：当服务消费方调用服务提供者失败后，会自动切换到其他服务提供者服务器进行重试，这通常用于读操作或具有幂等的写操作。
- 快速失败：当服务消费方调用服务提供者失败后，立即报错，只调用一次。用于非幂等性的写操作。
- 失败安全：当服务消费方调用服务出现异常时，直接忽略异常。用于写入审计日志等操作。
- 失败自动恢复：当服务消费方调用服务出现异常后，在后台记录失败的请求，并按照一定的策略后期再进行重试。用于消息通知操作。
- 并行调用：当服务消费方调用一个接口方法后，会并行调用多个服务提供者的服务，只要其中一个成功即返回。用于实时性要求较高的读操作，需要浪费更多服务资源。

## RocketMQ

### 架构

- producer：消息生产者，发送消息到消息服务器。
- broker：消息服务器，消息持久化存储，根据订阅推送push到消息消费者。
- consumer：消息消费者，订阅消息，主动向消息服务器拉取pull。
- nameserver：路由发现，producer发送消息前从nameserver根据负载算法选一台broker。

### nameserver

路由管理、服务注册、服务发现。

Broker每隔30s向NameService上报心跳包。NameService每隔10s扫描心跳包信息，若超过120s未上报则移除Broker信息。

### producer

- 消息长度验证：最大4m。
- 查找主题路由信息：获取具体的Broker。
- 选择消息队列
  - 不启用故障延迟：消息发送规避上次发送失败的Broker。
  - 启用故障延迟：验证消息队列是否可用。
- 消息发送：同步、异步、单向。

### broker

- CommitLog：消息存储文件，所有消息主题的消息存储在CommitLog文件中。
- ConsumerQueue：消息消费队列，每个消息主题包含多个消费队列，消息达到CommitLog文件后异步转发到消费队列，供消息消费者消费。
- IndexFile：消息索引文件，主要存储消息Key与Offset的对应关系。
- abort：关闭异常文件，如果存在abort文件说明Broker非正常关闭。
- checkpoint：文件检测点，存储CommitLog文件最后一次刷盘时间戳、ConsumerQueue最后一次刷盘时间戳、IndexFile最后一次刷盘时间戳。

文件过期时间默认3天，不管消息是否被消费都将被删除。

刷盘方式：同步、异步。异步是先将消息追加到内存映射文件，后定时将内存中的数据刷写到磁盘。

### consuemr

消息消费方式：广播、集群。

- 集群消息消费队列负载：同一个消费者可以分配多个消费队列，同一个消费队列同一时间内只会分配给一个消费者。
- 消息拉取：默认一批拉取32条，提交给消费线程池后继续拉取下一批。PUSH、PULL，Push为消费者端长轮询所实现。
- 消息消费完成：将消息消费进度偏移量存储在消息消费进度存储文件中，集群的进度文件在Broker端，广播的进度文件在消费者端。

### 为什么要nameserver不要zk

在RocketMQ中，不需要选举，Master/Slave的角色也是固定的。当一个Master挂了之后，可以写到其他Master上，但不会从一个Slave切换成Master。这种简化，使得RocketMQ可以不依赖ZK就很好的管理Topic/queue和物理机器的映射关系了，也实现了高可用。

### 集群

- 单节点
  - 优点：本地开发测试，配置简单，同步刷盘消息一条都不会丢。
  - 缺点：不可靠，如果宕机，会导致服务不可用。
- 主从（异步、同步双写）
  - 优点：同步双写消息不丢失（把消息写入master和slave），异步复制存在少量丢失（把数据写入master，master把数据复制到slave），主节点宕机，从节点可以对外提供消息的消费，但是不支持写入。
  - 缺点：主备有短暂消息延迟，毫秒级，目前不支持自动切换，需要脚本或者其他程序进行检测然后进行停止broker，重启让从节点成为主节点。
- 双主
  - 优点：配置简单，可以靠配置RAID磁盘阵列保证消息可靠，异步刷盘丢失少量消息。
  - 缺点: master机器宕机期间，未被消费的消息在机器恢复之前不可消费，实时性会受到影响。
- 双主双从，多主多从模式（异步复制）
  - 优点：磁盘损坏，消息丢失的非常少，消息实时性不会受影响，Master宕机后，消费者仍然可以从Slave消费。
  - 缺点：主备有短暂消息延迟，毫秒级，如果Master宕机，磁盘损坏情况，会丢失少量消息。
- 双主双从，多主多从模式（同步双写）
  - 优点：同步双写方式，主备都写成功，才向应用返回成功，服务可用性与数据可用性都非常高。
  - 缺点：性能比异步复制模式略低，主宕机后，备机不能自动切换为主机。

## Netty

- 传输模型：用什么样的通道将数据发送给对方，是BIO、NIO还是AIO，决定了框架的性能。
- 通讯协议：采用什么样的通讯协议，是HTTP还是内部私有协议，协议的选择不同，性能模型也不同。
- 线程模型：涉及如何读取数据包，读取之后的编解码在哪个线程中进行，编解码后的消息如何派发等方面。

### Netty零拷贝

- ByteBuffer采用DirectBuffer，使用堆外直接内存进行Socket读写，不需要进行字节缓冲区的二次拷贝。
- 提供了组合Buffer对象，可以聚合多个ByteBuffer，组合的时候不需要通过内存拷贝。
- 文件传输采用了transferTo方法，直接将文件缓冲区的数据发送到目标Channel。

### Reactor线程模型

- Reactor单线程模型：Acceptor负责接收客户端的TCP连接请求信息，链路建立成功之后，通过Dispatch将对应的ByteBuffer派发到指定的Handler上进行消息解码，用户Handler通过NIO线程将消息发送给客户端。
- Reactor多线程模型：NIO线程池处理IO操作。
- 主从Reactor多线程模型：NIO线程池接收客户端连接。

### Netty组件

- Bootstrap：启动配置
- EventLoop：事件循环
- Pipeline：管道
- Future、Promise：异步处理
- ByteBuf：内存分配
- 编解码

```java
EventLoopGroup group = new NioEventLoopGroup();
Bootstrap bootstrap = new Bootstrap();
bootstrap.group(group)
  .channel(NioSocketChannel.class)
  .option(ChannelOption.SO_KEEPALIVE, true)
  .handler(ch -> {
    ch;
  });
ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
channelFuture.channel().closeFuture().sync();
```

## ElasticJob

注册中心为zk

作业服务a -> 发起主节点选举 -> 设置分片标记 -> 注册作业全局信息 -> 注册服务器a信息 -> 等待作业运行

## Redis

### 优势

- 内存存储数据库
- 单进程线程的服务，单线程处理网络请求，IO多路复用模型
- 良好的数据类型

### 五种数据类型

- string：intset、sds
- list：quicklist
- hash：ziplist、dict
- set：intset、dict
- zset：查找用dict，范围用ziplist

### 多种数据结构

- sds：简单动态字符串，二进制安全，五种结构分别存储长度5、8、16、32、64
- skiplist：跳跃表，有序集合，实现比红黑树简单
- ziplist：压缩链表，存储空间连续；元素少且长度少；元素多时修改元素需重新调整空间
- adlist：双向链表，不满足ziplist的元素少且长度少的链表
- quicklist：快速链表，组合ziplist和adlist、综合考虑时间效率和空间效率
- dict：字典，散列表；两个hash表，用来进行扩表
- intset：整数集合，有序存储整型

### 主从复制

主节点持久化后返回调用方，再进行从节点复制，会导致分布式锁出问题，解决方案自己实现一个raft协议。

复制的流程：

- slave node启动，仅仅保存master node的信息，包括master node的host和ip，但是复制流程没开始。
- slave node内部有个定时任务，每秒检查是否有新的master node要连接和复制，如果发现，就跟master node建立socket网络连接。
- slave node发送ping命令到master node。
- 口令认证，如果master node设置了requirepass，那么slave node必须发送masterauth的口令过去进行认证。
- master node第一次执行全量复制（rdb文件），将所有数据发送给slave node。
- master node后续持续将写命令，同步复制（aof文件）给slave node。

复制的核心：

- master和slave都维护一个offset master会不断累加自身的offset，slave也会自身不断累加offset，slave每秒都会上报自己的offset给master，同时master也会记录每个slave的offset。
- backlog master node有一个backlog，默认是1MB大小；master node给slave node复制数据时，也会将数据在backlog中同步写一份。

rdb：保存某一个时间点之前的数据。
aof：服务器端执行的每一条命令。

### 哨兵和集群

- 哨兵：高可用，master故障时自动选择一个slave切换为master
- 集群：数据冗余自动分片到不同节点

### Pipelining

- 发送一批命令通过行位符分隔
- 同一个消息里发送一批命令
- 看起来是原子性
- 但管道与管道里的命令并非有序

### Transactions

- 事务与事务直接是有序的
- 报错不会回滚
- 原子性
- 但命令结果不能作为参数

### redis lua

命令结果可以作为参数

```sql
EVAL script numkeys key [key ...] arg [arg ...]
> EVAL “script” 2 wxList wxCount expired limit

redis.call("incr", wxCount)

> SCRIPT LOAD “script”
afwfwaaefwawafw

EVALSHA sha1 numkeys key [key ...] arg [arg ...]
> EVALSHA  “afwfwaaefwawafw” 2 wxList wxCount expired limit

```

限流使用滑动窗口实现

#### 基于寄存器的Lua虚拟机

- 存储操作数的结构基于CPU的寄存器
- 没有PUSH或POP操作，但是指令需要包含操作数的地址（寄存器）
- 也就是说，指令的操作数是在指令中显式寻址的，这与基于堆栈的模型不同，我们有一个指向操作数的堆栈指针

## ZooKeeper

### paxos协议

论文

### zab协议

ZooKeeper依据paxos协议所实现

### raft协议

etcd依据paxos协议所实现

Leader、Follower、Condidate

选举过程：所有成员参与选举，变为候选者进行拉票，最高者成为领导。若候选者票数一致则随机休眠再次选举。领导向成员发心跳，成员未收到会再次选举。

一致性策略：客户端数据达到领导，领导收到数据标为未提交，向成员复制数据，获得一半成员的响应将数据标为已提交，返回给客户端。

领导者异常：

- 领导接收数据前挂了：内部重选举，客户端幂等重试
- 领导接受到数据标为未提交时挂了：内部重选举，挂掉的领导成为成员，丢弃掉未提交的数据，客户端幂等重试
- 领导接受到数据标为未提交且向成员复制数据后挂了，拥有最新数据的成员成为领导，并向其他成员复制，数据最新，客户端幂等提交
- 领导标记为已提交后挂了：同上，客户端幂等提交
- 脑裂问题：3个节点在a组，两个节点在b组，此时ab组分离，a组的领导发现响应不超过一半成员，写入失败，b组产生领导，客户端向b领导写入成功
