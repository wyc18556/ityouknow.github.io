---
layout: post
title: mybatis整合spring-boot配置多数据源
category: framework
tags: [mybatis,spring-boot]
keywords: spring
no-post-nav: true
---

## 初始化项目

快速创建[springboot项目](https://start.spring.io/)，添加 __MySQL__ __Web__ __MyBatis__ __Lombok__ 依赖，生成项目。解压后导入 _idea_。

## maven依赖

添加druid 连接池依赖

	<!-- druid 连接池 -->
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>druid</artifactId>
		<version>${druid.version}</version>
	</dependency>

## 配置数据源

```sh
@Configuration
@MapperScan(basePackages = Wyc1856DataSourceConfig.MAPPER_PACKAGE, sqlSessionFactoryRef = "wyc1856SqlSessionFactory")
public class Wyc1856DataSourceConfig {

    /**
     * 该数据源下对应的mapper.xml文件地址
     */
    static final String MAPPER_XML_LOCATION = "classpath:mapper/wyc1856/*.xml";
    /**
     * 该数据源下对应的mapper类包路径
     */
    static final String MAPPER_PACKAGE = "club.wyc1856.mybatisspringboot.mapper.wyc1856";

    /**
     * 数据源
     */
    @Bean
    @ConfigurationProperties(prefix = "wyc1856.datasource")
    public DataSource wyc1856DataSource(){
        return new DruidDataSource();
    }

    /**
     * 事务管理器
     */
    @Bean
    public DataSourceTransactionManager wyc1856TransactionManager(){
        return new DataSourceTransactionManager(wyc1856DataSource());
    }

    /**
     * sessionFactory
     */
    @Bean
    public SqlSessionFactory wyc1856SqlSessionFactory(@Qualifier("wyc1856DataSource") DataSource wyc1856DataSource) throws Exception{
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(wyc1856DataSource);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_XML_LOCATION));
        return sqlSessionFactoryBean.getObject();
    }
}
```
## 补充

1. 相关注解说明
    
- @Configuration：将类声明为一个spring所管理的bean,多用在配置类上;

- @MapperScan：扫描mapper接口，多数据源情况下指定sessionFactory和sessionTemplate;

- @ConfigurationProperties：使用prefix属性指定前缀，自动绑定属性值，生成相应的bean。具体使用方式参考[这篇文章](https://www.cnblogs.com/duanxz/p/4520571.html)

- @Qualifier：spring默认采用byType注入，当存在多了类型相同的bean时就会注入失败，使用此注解可实现byName方式注入;

2. 具体示例代码可参考[mybatis-spring-boot](https://github.com/wyc18556/spring-boot-demo/tree/master/mybatis-spring-boot)
    