---
layout: post
title: Spring基础知识
category: note
tags: [spring, basic]
keywords: spring
no-post-nav: true
---

Spring IoC、Spring AOP、Spring Transaction相关内容

## Spring IoC
Spring 框架的核心就是Spring IoC容器，容器创建Bean，将它们组装在一起，配置它们并管理它们的完整生命周期。
- Spring 容器使用 __依赖注入__ 的方式来管理Beans
- Spring可通过XML配置文件、注解配置、Java Config三种方式来定义和配置源数据 Bean Definition，用于Spring 容器进行对象的实例化、配置和组装。
- 常见的Spring 容器有：BeanFactory(初级容器)、ApplicationContext(高级容器)
- 简单来说，SpringIoC的实现原理，就是 __工厂模式__ 和 __反射机制__。
- 可以设置Bean属性"lazy-init=true"实现延迟加载。

## Spring AOP
### 理论概念:
- join point:连接点，在 Spring AOP 中指的都是方法节点
- point cut:切点，用来修饰 join point，指定匹配的条件
- advice:匹配 point cut，对目标方法（proxy）提供增强
- aspect:切面，由 point cut 和 advice 组成，它既包括了横切逻辑的定义，也包括了连接点的定义
- weaving:织入，将 aspect 和其他对象连接起来，创建 advised object 的过程
- target:织入 advice 后所产生的代理类，也被称为 advised object

总结：advice 是在 join point 上执行的, 而 point cut 规定了哪些 join point 可以执行哪些 advice。

### AOP织入的实现方式
- 静态代理：指通过AOP框架提供的命令进行编译，有编译时编织（特殊编译器实现）、类加载时编织（特殊的类加载器实现）。
- 动态代理：运行时在内存中“临时”生成AOP动态代理，有JDK动态代理（需实现至少一个接口，只能为接口方法提供增强）、CGLIB动态代理（使用继承方式实现，类不能被final修饰）

### Aspect使用流程
 __声明 point cut__ ，一个 point cut 由两部分组成：
- 一个方法签名
- 一个point cut 表达式，用来指定那些方法执行是我们感兴趣的。一个 point cut 表达式由标志符和操作参数组成。常用的标志符有：execution、within、bean、@annotation。

__声明 advice__ ，常用的advice类型有：
- @Before，在 join point 方法之前执行
- @AfterReturning，在 join point 方法正常返回退出后执行
- @AfterThrowing，在 join point 方法抛出异常退出后执行
- @After，在 join point 方法退出后执行，无论是正常返回还是抛出异常
- @Around，在 join point 方法之前和之后执行

## Spring Transaction
### 事务的特性:
事务有四大特性，简称ACID
- 原子性（atomic）：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
- 一致性（consistence）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的数据必须完全符合所有的预设约束、触发器、级联回滚等。
- 隔离性（isolation）：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化。
- 持久性（durability）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

### Spring事务实现方式
spring启动时会为声明了事务管理的类及方法创建代理类，在方法调用时通过spring aop(借助动态代理实现)在业务处理前添加开启事务的切点，在逻辑处理后添加提交/回滚事务的切点；

### 事务的嵌套概念
所谓事务嵌套就是两个以上事务方法之间的相互调用，故会引入事务传播的问题，spring的事务传播机制就是为了解决这个问题而设计的。

__注意__：由于spring事务是通过代理机制实现的，所以在同一个类中一个方法（未声明为事务管理）直接通过this调用另一个方法（声明为事务管理），事务将失效。原因是调用第一个方法时未通过代理类，然后直接通过this调用声明事务的方法实际上也未经过代理类调用，故事务不生效。

### 事务的隔离级别
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

### 事务的传播级别
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