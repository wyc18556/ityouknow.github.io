---
layout: post
title: Spring事务的传播机制
category: springboot
tags: [springboot]
no-post-nav: true
---

# Spring事务的传播机制

## Spring事务实现方式
spring启动时会为声明了事务管理的类及方法创建代理类，在方法调用时通过spring aop(借助动态代理实现)在业务处理前添加开启事务的切点，在逻辑处理后添加提交/回滚事务的切点；

## 事务的嵌套概念
所谓事务嵌套就是两个以上事务方法之间的相互调用，故会引入事务传播的问题，spring的事务传播机制就是为了解决这个问题而设计的。

__注意__：由于spring事务是通过代理机制实现的，所以在同一个类中一个方法（未声明为事务管理）直接通过this调用另一个方法（声明为事务管理），事务将失效。原因是调用第一个方法时未通过代理类，然后直接通过this调用声明事务的方法实际上也未经过代理类调用，故事务不生效。

### spring事务的6种传播方式

- PROPAGATION_REQUIRED -- 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，@Transactional注解propagation的默认值。

- PROPAGATION_SUPPORTS -- 支持当前事务，如果当前没有事务，就以非事务方式执行。 

- PROPAGATION_MANDATORY -- 支持当前事务，如果当前没有事务，就抛出异常。 

- PROPAGATION_REQUIRES_NEW -- 新建事务，如果当前存在事务，把当前事务挂起。（内外事务无任何关系） 

- PROPAGATION_NOT_SUPPORTED -- 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 

- PROPAGATION_NEVER -- 以非事务方式执行，如果当前存在事务，则抛出异常。 

- PROPAGATION_NESTED -- 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则新建事务，与PROPAGATION_REQUIRED类似的操作。

> 支持当前事务:内外事务在一个事务内，任意一个事务回滚，则都会回滚；

> 嵌套事务：外部事务回滚则内部事务跟着回滚，内部事务回滚，外部事务无需跟着回滚（外影响内，内不影响外）；