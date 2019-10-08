---
layout: post
title: 框架相关笔记
category: base
tags: [base]
no-post-nav: true
---

## SpringBoot
### 核心功能及优点:
- 以java -jar的方式启动，独立运行，使【部署】变简单
- 内嵌servlet容器（引入spring-boot-starter-web），使【部署】变简单
- 提供start简化maven配置，使【配置】变简单
- 自动配置Spring Bean，使【配置】变简单
- 准生产的应用监控（acturator），使【监控】变简单

### spring boot常用的条件注解
- @ConditionalOnBean，仅在当前上下文中存在某个bean时，才会实例化这个Bean。
- @ConditionalOnClass，某个class位于类路径上，才会实例化这个Bean。
- @ConditionalOnExpression，当表达式为true的时候，才会实例化这个Bean。
- @ConditionalOnMissingBean，仅在当前上下文中不存在某个bean时，才会实例化这个Bean。
- @ConditionalOnMissingClass，某个class在类路径上不存在的时候，才会实例化这个Bean。
- @ConditionalOnNotWebApplication，不是web应用时才会实例化这个Bean。
- @AutoConfigureAfter，在某个bean完成自动配置后实例化这个bean。
- @AutoConfigureBefore，在某个bean完成自动配置前实例化这个bean。
- @ConfigurationProperties(“spring.redis”) 自动注入属性文件，将配置文件中的属性绑
定到实体上。

### Spring Boot、Spring MVC 和 Spring 有什么区别？
Spring指的是一个生态，它包含很多项目，比如Spring IOC、Spring AOP、Spring Secutity等，SpringMVC就是Spring众多框架中的一个，而Spring Boot是构建在Spring Framework之上的Boot启动器，旨在更容易的配置一个Spring项目。
### 如何统一引入 Spring Boot 版本？
- 继承 spring-boot-starter-parent 项目。配置代码如下：
```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.1.RELEASE</version>
</parent>
```
- 导入 spring-boot-dependencies 项目依赖。配置代码如下：
```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
### 配置文件格式
- .properties格式，spring项目中最常见的配置文件格式。
- .yaml格式，可读性更强，分层结构更明显，不支持@ProperSource注解读取，可通过@Value注解替代。

Spring Boot默认的配置文件为根目录下的application配置文件。
创建application-{profile}.yml文件 存放不同环境特有的配置，在application主配置文件通过spring.profiles.active=xxx指定加载不同的环境配置。

### Spring Boot 有哪些配置方式
- 传统的XML配置文件
- 注解的方式，类似@Service、@Component等
- Java Config的方式，使用@Configuration声明配置类，通过@Bean注解的公共方法声明bean。

### Spring Boot的核心注解
@SpringBootApplication是一个组合注解，包含的主要注解包括：
- @SpringBootConfiguration，功能和@Configuration类似，标注当前类为配置类。
- @ComponentScan，自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。
- @EnableAutoConfiguration，通过导入@Import({AutoConfigurationImportSelector.class})，在内部通过 __SpringFactoriesLoader__ 类从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中以 _org.springframework.boot.autoconfigure.EnableAutoConfiguration_ 为key的所有配置类组合起来，再通过条件注解筛选进行自动加载配置。

### Spring Boot2.0的新特性及主要改动
- 起步JDK8和支持JDK9
- 默认数据库连接池由Tomcat改成HikariCP
- 默认redis客户端有Jedis改成Lettuce
- 建立在Spring Framework 5之上，支持响应式编程(reactive)
- HTTP/2的支持

### 启动流程
![Spring Boot 启动流程图](http://image.wyc1856.club/2019-08-28-13-56-31.png)
> spring boot 服务启动会先初始化一个SpringApplication实例，此阶段只要做以下几件事：
>> 1.把应用启动类添加到PrimarySources集合中   
>> 2.获得应用程序的类型（是否为web项目），后续用来判断是否需要启动内置web容器。  
>> 3.通过SpringFactoriesLoader.loadFactoryNames方法查询classpath:*/META-INF/spring.factories中指定的initializers(初始化器)和listeners(监听器)类名集合，然后通过反射机制进行实例化并设置到springApplication实例内   
>> 4.获得应用的主启动类： new一个RuntimeException并获得其堆栈信息，循环堆栈集合查找方法名为main的类，将其设置为应用主类。

> 创建完实例后调用run方法，此方法主要的作用:
>> 1.初始化SpringApplicationRunListeners实例 (包含一个SpringApplicationLister实例列表和一个Log实例)。   
>> 2.广播服务开始启动事件   
>> 3.通过应用类型获得环境信息，广播环境准备完成事件，配置属性信息，创建应用上下文实例   
>> 4.通过prepareContext方法执行上下文预处理逻辑，包括相关bean的初始化和所有初始化器的执行、广播上下文准备事件、加载上下文信息、广播上下文加载完成事件   
>> 5.执行refreshcontext方法，包含自动装配，注入bean等spring ioc相关操作。自动装配的流程为:通过调用AutoConfigurationImportSelector重写ImportSelector(选择器接口)的selectImports方法，最终还是通过SpringFactoriesLoader.loadFactoryNames方法查询classpath:*/META-INF/spring.factories文件中key为EnableAutoConfiguration的记录，统计出一个自动配置类的类名列表并返回。后续再通过条件注解判断是否需要进行装配，所以引入一些starter即可实现组件的自动配置    
>> 6.广播服务启动完成事件   
>> 7.ApplicationRunner和CommandLineRunner实例执行   
>> 8.广播服务运行中事件，最后返回应用上下文信息

SpringApplicationRunListeners的部分方法会循环调用SpringApplicationRunListener中对应的方法，然后通过实现类EventPublishingRunListener调用SimpleApplicationEventMulticaster进行事件广播，
通过事件类型找到对应的事件监听器实现监听逻辑(还可以启动异步task Executor来实现异步调用);

## Spring
### Spring IoC
Spring 框架的核心就是Spring IoC容器，容器创建Bean，将它们组装在一起，配置它们并管理它们的完整生命周期。
- Spring 容器使用 __依赖注入__ 的方式来管理Beans
- Spring可通过XML配置文件、注解配置、Java Config三种方式来定义和配置源数据 Bean Definition，用于Spring 容器进行对象的实例化、配置和组装。
- 常见的Spring 容器有：BeanFactory(初级容器)、ApplicationContext(高级容器)
- 简单来说，SpringIoC的实现原理，就是 __工厂模式__ 和 __反射机制__。
- 可以设置Bean属性"lazy-init=true"实现延迟加载。

### Spring AOP
#### 理论概念:
- join point:连接点，在 Spring AOP 中指的都是方法节点
- point cut:切点，用来修饰 join point，指定匹配的条件
- advice:匹配 point cut，对目标方法（proxy）提供增强
- aspect:切面，由 point cut 和 advice 组成，它既包括了横切逻辑的定义，也包括了连接点的定义
- weaving:织入，将 aspect 和其他对象连接起来，创建 advised object 的过程
- target:织入 advice 后所产生的代理类，也被称为 advised object

总结：advice 是在 join point 上执行的, 而 point cut 规定了哪些 join point 可以执行哪些 advice。
#### AOP织入的实现方式
- 静态代理：指通过AOP框架提供的命令进行编译，有编译时编织（特殊编译器实现）、类加载时编织（特殊的类加载器实现）。
- 动态代理：运行时在内存中“临时”生成AOP动态代理，有JDK动态代理（需实现至少一个接口，只能为接口方法提供增强）、CGLIB动态代理（使用继承方式实现，类不能被final修饰）
#### Aspect使用流程
1. __声明 point cut__ ，一个 point cut 由两部分组成：
- 一个方法签名
- 一个point cut 表达式，用来指定那些方法执行是我们感兴趣的。一个 point cut 表达式由标志符和操作参数组成。常用的标志符有：execution、within、bean、@annotation。
2. __声明 advice__ ，常用的advice类型有：
- @Before，在 join point 方法之前执行
- @AfterReturning，在 join point 方法正常返回退出后执行
- @AfterThrowing，在 join point 方法抛出异常退出后执行
- @After，在 join point 方法退出后执行，无论是正常返回还是抛出异常
- @Around，在 join point 方法之前和之后执行

### Spring Transaction
#### 事务的特性:
事务有四大特性，简称ACID
- 原子性（atomic）：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
- 一致性（consistence）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的数据必须完全符合所有的预设约束、触发器、级联回滚等。
- 隔离性（isolation）：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化。
- 持久性（durability）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。
#### 事务的隔离级别
不同数据库对事务隔离级别的支持和实现略有不同，在 TransactionDefinition 接口中定义如下:
```java
// TransactionDefinition.java

/**
 * 【Spring 独有】使用后端数据库默认的隔离级别
 *
 * MySQL 默认采用的 REPEATABLE_READ隔离级别
 * Oracle 默认采用的 READ_COMMITTED隔离级别
 */
int ISOLATION_DEFAULT = -1;

/**
 * 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
 */
int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;

/**
 * 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
 */
int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;
/**
 * 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
 */
int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;
/**
 * 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。
 *
 * 但是这将严重影响程序的性能。通常情况下也不会用到该级别。
 */
int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;
```
#### 事务的传播级别
事务的传播级别，并不是数据库事务规范中的名词，而是 spring 自身所定义的。通过事务的传播级别，Spring 才知道如何处理事务，是创建一个新事务，还是继续使用当前事务，或者抛出异常。在 TransactionDefinition 接口中，定义了 __三类七种__ 传播级别，代码如下：
```java
// TransactionDefinition.java

// ========== 支持当前事务的情况 ========== 

/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则创建一个新的事务。
 */
int PROPAGATION_REQUIRED = 0;
/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则以非事务的方式继续运行。
 */
int PROPAGATION_SUPPORTS = 1;
/**
 * 如果当前存在事务，则使用该事务。
 * 如果当前没有事务，则抛出异常。
 */
int PROPAGATION_MANDATORY = 2;

// ========== 不支持当前事务的情况 ========== 

/**
 * 创建一个新的事务。
 * 如果当前存在事务，则把当前事务挂起。
 */
int PROPAGATION_REQUIRES_NEW = 3;
/**
 * 以非事务方式运行。
 * 如果当前存在事务，则把当前事务挂起。
 */
int PROPAGATION_NOT_SUPPORTED = 4;
/**
 * 以非事务方式运行。
 * 如果当前存在事务，则抛出异常。
 */
int PROPAGATION_NEVER = 5;

// ========== 其他情况 ========== 

/**
 * 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行。
 * 如果当前没有事务，则等价于 {@link TransactionDefinition#PROPAGATION_REQUIRED}
 */
int PROPAGATION_NESTED = 6;
```

## Mybatis相关
### 一级缓存
```sh
<setting name="localCacheScope" value="SESSION"/>
```
- MyBatis一级缓存的生命周期默认和SqlSession一致。
- MyBatis SqlSession向用户提供的操作数据库的方法，其实都是委派给Executor实现的。Mybatis缓存的查询和写入就是在Executor中完成的。其中一级缓存主要是通过BaseExecutor完成的，其内部的成员变量localCache就是用来存放一级缓存的。localCache内部维护一个没有容量限定的HashMap。同一会话的update(包括insert和delete)操作会清空缓存。
- MyBatis的一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement。

### 二级缓存
```sh
<setting name="cacheEnabled" value="true"/>
```
- MyBatis的二级缓存相对于一级缓存来说，实现了SqlSession之间缓存数据的共享，同时粒度更加的细，能够到namespace级别，通过Cache接口实现类不同的组合，对Cache的可控性也更强。
- MyBatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
- 在分布式环境下，由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将MyBatis的Cache接口实现，有一定的开发成本，直接使用Redis、Memcached等分布式缓存可能成本更低，安全性也更高。

### Mybatis 是否支持延迟加载？如果支持，它的实现原理是什么？
Mybatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载。其中，association 指的就是一对一，collection 指的就是一对多查询。在\<resultMap>标签中添加\<association>和\<collection>标签。
在 Mybatis 配置文件中，可以配置\<setting name="lazyLoadingEnabled" value="true" /> 来启用延迟加载的功能。默认情况下，延迟加载的功能是关闭的。
它的原理是，使用 CGLIB 或 Javassist( 默认 ) 创建目标对象的代理对象。当调用代理对象的延迟加载属性的 getter 方法时，进入拦截器方法。比如调用 a.getB().getName() 方法，进入拦截器的 invoke(...) 方法，
发现 a.getB() 需要延迟加载时，那么就会单独发送事先保存好的查询关联 B 对象的 SQL ，把 B 查询上来，然后调用a.setB(b) 方法，于是 a 对象 b 属性就有值了，接着完成a.getB().getName() 方法的调用。这就是延迟加载的基本原理。

### Mapper接口的工作原理是什么
Mybatis通过JDK动态代理为Mapper接口生成实现类代理对象，调用代理方法，根据映射找到对应的mappedStatement。然后从sqlSessionFactory中获取sqlSession，交给Executor执行sql。

### 执行器包括哪些？
BaseExcutor(实现一级缓存，子类包括：SimpleExcutor、ReuseExcutor、BatchExcutor)，CachingExcutor(实现二级缓存)；

### Mybatis插件的运行原理，如何编写一个插件？


## Dubbo相关
### Dubbo异常处理
dubbo的异常处理类是com.alibaba.dubbo.rpc.filter.ExceptionFilter 类,源码这里就不贴了.归纳下对异常的处理分为下面几类:
1. 如果provider实现了GenericService接口,直接抛出
2. 如果是checked异常，直接抛出
3. 在方法签名上有声明，直接抛出
4. 异常类和接口类在同一jar包里，直接抛出
5. 是JDK自带的异常，直接抛出
6. 是Dubbo本身的异常，直接抛出
7. 否则，包装成RuntimeException抛给客户端

### 如何使用自定义异常 
1. 自定义异常声明为checked异常，直接继承Exception类
2. api方法声明抛出该自定义异常
3. 把自定义异常放在api包中
4. 重写ExceptionFilter，允许直接返回指定的自定义异常

### Dubbo调用流程
![dubbo调用流程图](http://image.wyc1856.club/2019-09-02-17-17-59.png)
> provider流程
>> 第0步，start启动服务  
>> 第1步，注册提供者信息到注册中心

> consumer流程
>> 第2步，subscribe订阅使用到的服务提供者列表，并缓存到本地   
>> 第3步【异步】，当服务提供者发生变化时，注册中心notify服务消费者，获取最新的服务列表，更新本地缓存

> invoke调用
>> 第4步，consumer直接发起对provider的调用，无需经过注册中心，而对多个provider的负载均衡，consumer通过cluster组件实现

> count监控
>> 第5步【异步】，consumer和provider都异步通知监控中心

### Dubbo对调用结果进行缓存
Dubbo通过CacheFilter过滤器，提供对结果缓存的功能，既可以适用于Consumer也可以适用与Provider。目前提供了三种实现:
- lru：基于最近最少使用原则，保持最热数据被缓存。通过继承LinkedHashMap实现。
- threadlocal：当前线程缓存。
- jcache：可以桥接各种缓存实现。

### 本地暴露和远程暴露的区别
远程暴露：consumer调用provider是跨进程的，需要进行网络通信。
本地暴露：使用 _injvm://_ 协议，是一个伪协议，它不开启端口，不发起远程调用，但执行Dubbo的filter链。

### Dubbo有哪些负债均衡策略
- Random LoadBalance：随机，按权重设置随机概率。
- RoundRobin LoadBalance：轮询，按公约后的权重设置轮询比率。
- LeastActive LoadBalance：最少活跃调用数，相同活跃数的随机。使慢的提供者收到更少的请求，因为越慢的提供者的调用前后计数差（时间差）会越大。
- ConsistentHash LoadBalance：一致性Hash，相同参数的请求总是发到同一提供者。

### Dubbo其他知识点
- 支持的通信协议有9中，常见的有dubbo和rest协议。
- 支持点对点直连，以服务接口为单位忽略注册中心的提供者列表。另外，直连provider时，如果要debug调试provider，禁用该provider注册到注册中心。否则，会被其他consumer调用到。
- Dubbo使用Javassist和JDK两种方式实现动态代理。
- Dubbo可使用 Apache SkyWalking 组件实现链路追踪。
- Dubbo可使用 Sentinel 组件实现服务降级。

## 消息中间件
### 使用场景
- 应用解耦：从系统之间的主动调用变成消息的订阅发布，从而解耦。
- 异步处理：系统间串行逐个同步调用非常耗时且不稳定（当任一系统调用报错或超时都将导致整个逻辑失败），使用消息队列做到异步、并行处理。当然，前提是返回的结果不依赖于处理的结果。
- 流量削峰：大量请求（例如秒杀）直接打到数据库将会导致数据库挂掉，通过将请求先转发到消息队列中，然后系统按照数据库所能处理的并发量从消息队列中逐步拉取消息进行消费。
- 消息通讯：消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。有基于消息队列的RPC框架，例如rabbitmq-jsonrpc（基于rabbitmq低延迟特性）。
- 日志处理：比较熟悉的 ELK + Kafka 日志方案。业务系统向Kafka中推送日志消息。Logstash从Kafka中消费日志信息，并存入Elasticsearch中。Elasticsearch负责日志的搜索和统计工作。Kibana是基于Elasticsearch的数据可视化组件。

其中，应用解耦和异步处理比较核心。

### 问题
- 系统可用性降低：在原来的系统架构中加入了MQ，万一MQ挂了，整套系统就挂了。所以，消息队列一定要做好高可用。
- 系统复杂度提高：需要考虑和关注的问题变多了，消息如何保证不丢失（消息确认和持久化）？消息如何保证不被重复消费（消息添加唯一标识，添加排重记录。业务层做幂等判断）？需要消息顺序的业务场景如何处理等。
- 一致性问题：在使用MQ时，一定要保证数据的最终一致性。

### 消息的投递方式
> push
>> 优点：保证及时性   
>> 缺点：受限于消费者的消费能力，可能造成消息的堆积。

> pull
>> 优点：主动权掌握在消费方，可根据自己的消费速度进行消息的拉取。   
>> 缺点：消费方不知道什么时候获取新的消息，会有消息延迟和忙等。

### 消息的持久化方式
- 分布式KV存储
- 关系型数据库DB
- 文件系统

从存储效率上来说，文件系统>分布式KV存储>关系型数据库DB。另外，从消息中间件本身定义考虑，应该尽量减少对于外部第三方中间件的依赖。所以个人觉得采用消息刷盘至中间件所部署的虚拟机/物理机的的文件系统来做持久化比较合适。

### Kafka相关
#### 常见名词及概念
- Borker：节点，Kafka 集群中，通过 Zookeeper 选举某个 Broker 作为 Controller ，用来进行 leader election 以及 各种 failover，保证高可用。
- Producer：消息提供者，根据指定算法，将消息发送到指定的分区中去。
- Consumer：消息消费者，负责消费消息。
- Consumer Group：每个 Consumer 都属于一个 Consumer group，每条消息只能被 Consumer group 中的一个 Consumer 消费，但可以被多个 Consumer group 消费
- Topic：特指 Kafka 处理的消息源（feeds of messages）的不同分类。
- Partition：Topic 物理上的分组（分区），一个 Topic 可以分为多个 Partition 。每个 Partition 都是一个有序的队列。Partition 中的每条消息都会被分配一个有序的 id（offset）每个replica分布在不同的broker节点上。多个partition需要选举出leader partition，leader partition负责读写，并由zookeeper负责 fail over。
- Segement：消息片段，一个Partition可以包含多个Segement。
- ISR：In Sync Replicas（正在同步的集合），只有在 ISR 中的副本才能被拿来替代主副本。ISR = Leader + 没有落后太多的副本。
- AR：副本全集，等于 ISR + OSR。
#### 常用配置
- acks：提供消息发送是否发送成功的可靠性保证，是写入leader分区完成就响应给producter成功，还是写入所有分区成功再响应。
- replication.factor：给topic设置的参数，表示分区的副本数，一般这个值都要大于1。
- retries：写入重试次数，一般设置为 MAX，表示无限重试。
- batch.size：采用分批发送消息的策略，减少请求数。此参数设置缓冲空间的大小。
- linger.ms：配置生产者在发送请求之前等待时间，希望更多的消息填补到未满的批中。在低负载的情况下，设置此值大于0，以少量的延迟代价换取更少、更有效的请求。
- buffer.memory：控制生产者可用的缓存总量，如果消息发送速度比其传输到服务器的快，将会耗尽这个缓存空间。当缓存空间耗尽，其他发送调用将被阻塞，阻塞时间的阈值通过max.block.ms设定，之后它将抛出一个TimeoutException。
- min.insync.replicas；在kafka服务端设置的参数，表示ISR分区集合的最小值，一般都要大于1，这样保证leader挂了还有一个follower。
#### Kafka会不会丢数据
> Broker
>> 比较常见的一种，leader分区的消息还没同步到replica分区，所在的Broker宕机了就会导致还未同步的消息丢失。可以通过给Topic设置replication.factor参数大于1，保证每个分区至少有两个副本。在kafka服务端设置min.insync.replicas参数大于1，保证leader分区感知到至少有一个follower还跟自己保持联系，没掉队。

> 提供者端
>> 在produer端设置acks=all，要求消息写入所有replica之后，才能认为是写入成功。同时设置retries=MAX，保持无限重试。这两个设置可以保证消息在提供者端一定不会丢失。

> 消费者端
>> 消息还没处理完成，kafka自动提交offset会导致消费者端丢消息。那么只要关闭自动提交offset，在处理完成之后自己手动提交offset，就可以保证数据不会丢。但是有可能导致重复消费的问题，比如消息刚处理完，还没提交offset，结果自己挂了，就会导致此条消息还会被消费一次，就得业务端做幂等判断了。

## Java基础
### HashMap(由数组和链表组合构成的数据结构)
![Java7](http://image.wyc1856.club/2019-08-21-20-39-53.png)
![Java8](http://image.wyc1856.club/2019-08-21-20-40-53.png)
对比:
- 发生hash冲突时，Java7会在链表头部插入，Java8会在链表尾部插入。
- 扩容后转移数据，Java7转移前后链表顺序会倒置（高并发场景下会导致死循环），Java8还是保持原来的顺序。
- Java8中当链表长度大于8时会把链表转换成红黑树，寻址时间复杂度从O(N)变成O(log(N))，很大程度上提升了性能。
### ConcurrentHashMap
Java 7中的ConcurrentHashMap的底层数据结构仍然是数组和链表。与HashMap不同的是，ConcurrentHashMap最外层不是一个大的数组，而是一个Segment的数组。每个Segment包含一个与HashMap数据结构差不多的链表数组。通过分段锁思想提高并发效率（有点类似LongAdder的做法）。整体数据结构如下图所示:
![Java7数据结构图](http://image.wyc1856.club/2019-08-22-14-08-04.png)
![Java7类图](http://image.wyc1856.club/2019-08-22-16-58-49.png)
#### 初始化操作：
入参:
- initialCapacity(初始化容量，默认16)
- loadFactor(负载因子，默认0.75)
- concurrencyLevel(并发等级，默认16)

具体操作:
- 计算segmentMask(段掩码，默认为15，二进制位1111)，首先计算segments数组大小(大于等于并发等级的最小二次幂),段掩码为segments.size()-1
- 计算segmentShift(段偏移量，默认为28)，32-sshift(计算segments数组大小时左移的次数)
- 计算每个segment对象中HashEntry<K,V>[]数组的大小cap=向上取整(initialCapacity/ssize)

总的来说就是通过concurrencyLevel决定分多少个片段，再结合initialCapacity决定每个片段下的链表个数。

#### put操作
首先根据(hash(key)>>>segmentShift)&segmentMask定位片段下标，然后加锁，再根据hash(key)&(cap-1)定位链表，后续操作类似hashMap。

#### get操作
get操作定位链表的方法和put一致，但是get方法没有加锁，原因是链表节点HashEntry中的变量通过volatile修饰，通过CAS完成更新操作，并且通过volatile修饰table来保证扩容时链表数组的内存可见性。

#### size操作
连续遍历2次Segment数组，将count的值，进行相加操作。如果遍历2次后的结果，都没有变化，那么就直接将count的和返回，如果此时发生了变化，那么就对整张hash表进行加锁处理。

Java 8为进一步提高并发性，摒弃了分段锁的方案，而是直接使用一个大的数组。同时为了提高哈希碰撞下的寻址性能，Java 8在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为O(N)）转换为红黑树（寻址时间复杂度为O(log(N))）。其数据结构如下图所示：
![Java8数据结构图](http://image.wyc1856.club/2019-08-22-17-17-28.png)
#### put操作
如果key对应的数组元素(即链表表头或者树的根元素)为null，则通过CAS操作将其设置为当前值。否则对该元素使用sychronized关键字申请锁，然后进行操作。

#### get操作
由于数组被volatile关键字修饰，因此不用担心数组的可见性问题。对于数组元素(即链表表头或者树的根元素)的可见性由Unsafe的getObjectVolatile方法保证。同时Node实例的Key值和hash值都由final修饰，不可变更，无需关心它们被修改后的可见性问题。并且其value和对下一个元素的引用由volatile修饰，可见性也有保障。
```java
static class Node<K,V> implements Map.Entry<K,V> {
  final int hash;
  final K key;
  volatile V val;
  volatile Node<K,V> next;
}
```
#### size操作
put方法和remove方法都会通过addCount方法维护Map的size，维护方式类似LongAdder。size方法返回sumCount方法的值(baseCount累加cell.value)。
