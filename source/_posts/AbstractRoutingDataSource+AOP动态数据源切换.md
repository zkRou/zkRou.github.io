---
title: AbstractRoutingDataSource+AOP动态数据源切换
author: Kairou Zeng
date: 2019/05/24
tags: 
	- Mybatis
	- Database
categories: 
	- Database
description: 项目开发过程中，难免会遇到需要配置多个数据源的情况，使用AbstractRoutingDataSource结合切面可以很方便地实现动态切换。
---
# 为什么写

项目开发过程中，出现一个项目需要连接多个不同数据源的情况，最近加入的一个新的项目发现使用`AbstractRoutingDataSource`结合`AOP`利用`ThreadLocal`（每一个线程都可以独立改变自己的副本，而不会影响其他线程所对应的副本）的方式可以实现动态数据源切换，因此学习一下。


# 思路

- 定义一个类，继承`AbstractRoutingDataSource`并重写`determineCurrentLookupKey()`方法
- 定义切面类（具体怎么切可以自己定义）
- 定义多个`DataSource`，并注入到`SqlSessionFactory`，指定`@Mapper`的`sqlSessionFactoryRef`
- 定义数据源的名称常量类，并使用`ThreadLocal`定义一个获取和设置上下文环境的类，负责改变上下文数据源的名称

# 实现

请移步至[GitHub](https://github.com/zkRou/abstract-routing-data-source-demo)

# 相关类剖析
## AbstractRoutingDataSource分析

- 在`org.springframework.jdbc.datasource.lookup`包中
- 继承了`AbstractDataSource`类，而`AbstractDataSource`类实现了`DataSource`接口
- 获取连接的方法中，`determineTargetDataSource()`方法是关键，而该方法中`determineCurrentLookupKey()`方法是关键
- 继承该类，重写`determineCurrentLookupKey()`方法，该方法返回所要用的数据源dataSource的key值，应用根据key来动态切换到真正的`DataSource`上，来实现数据源的切换。`resolvedDataSource`的`Map`，存储了多个`DataSource`，如果没有指定的那个key，就会使用默认的`DefaultDataSource`，返回默认的`DataSource`
- 类图：

![AbstractRoutingDataSource类图](https://raw.githubusercontent.com/zkRou/note/master/AbstractRoutingDataSource-class-diagram.PNG)

- 源码：
```java
/**
 * Abstract {@link javax.sql.DataSource} implementation that routes {@link #getConnection()}
 * calls to one of various target DataSources based on a lookup key. The latter is usually
 * (but not necessarily) determined through some thread-bound transaction context.
 *
 * @author Juergen Hoeller
 * @since 2.0.1
 * @see #setTargetDataSources
 * @see #setDefaultTargetDataSource
 * @see #determineCurrentLookupKey()
 */
public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean {

	@Nullable
	private Map<Object, Object> targetDataSources;

	@Nullable
	private Object defaultTargetDataSource;

	private boolean lenientFallback = true;

	private DataSourceLookup dataSourceLookup = new JndiDataSourceLookup();

	@Nullable
	private Map<Object, DataSource> resolvedDataSources;

	@Nullable
	private DataSource resolvedDefaultDataSource;

	/**
	 * Specify the map of target DataSources, with the lookup key as key.
	 * The mapped value can either be a corresponding {@link javax.sql.DataSource}
	 * instance or a data source name String (to be resolved via a
	 * {@link #setDataSourceLookup DataSourceLookup}).
	 * <p>The key can be of arbitrary type; this class implements the
	 * generic lookup process only. The concrete key representation will
	 * be handled by {@link #resolveSpecifiedLookupKey(Object)} and
	 * {@link #determineCurrentLookupKey()}.
	 */
	 //调用setTargetDataSources()方法把Map填进去
	public void setTargetDataSources(Map<Object, Object> targetDataSources) {
		this.targetDataSources = targetDataSources;
	}

	/**
	 * Specify the default target DataSource, if any.
	 * <p>The mapped value can either be a corresponding {@link javax.sql.DataSource}
	 * instance or a data source name String (to be resolved via a
	 * {@link #setDataSourceLookup DataSourceLookup}).
	 * <p>This DataSource will be used as target if none of the keyed
	 * {@link #setTargetDataSources targetDataSources} match the
	 * {@link #determineCurrentLookupKey()} current lookup key.
	 */
	 //setDefaultTargetDataSource()方法吧DefaultDataSource设置好
	public void setDefaultTargetDataSource(Object defaultTargetDataSource) {
		this.defaultTargetDataSource = defaultTargetDataSource;
	}
        ...

    	/**
	 * Retrieve the current target DataSource. Determines the
	 * {@link #determineCurrentLookupKey() current lookup key}, performs
	 * a lookup in the {@link #setTargetDataSources targetDataSources} map,
	 * falls back to the specified
	 * {@link #setDefaultTargetDataSource default target DataSource} if necessary.
	 * @see #determineCurrentLookupKey()
	 */
	protected DataSource determineTargetDataSource() {
		Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
		Object lookupKey = determineCurrentLookupKey();

        //从resolvedDataSources取出对应key的DataSource，如果找不到就用默认的数据源
		DataSource dataSource = this.resolvedDataSources.get(lookupKey);
		if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
			dataSource = this.resolvedDefaultDataSource;
		}
		if (dataSource == null) {
			throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
		}
		return dataSource;
	}

	/**
	 * Determine the current lookup key. This will typically be
	 * implemented to check a thread-bound transaction context.
	 * <p>Allows for arbitrary keys. The returned key needs
	 * to match the stored lookup key type, as resolved by the
	 * {@link #resolveSpecifiedLookupKey} method.
	 */
	@Nullable
	protected abstract Object determineCurrentLookupKey();

}

```

## DataSourceBuilder

- 在`org.springframework.boot.jdbc`包中
- 方便构建自定义的`DataSource`

```java
@Configuration
public class DataSourceConfig{
	@Bean
	public DataSource getDataSource(){
		DataSourceBuilder builder = DataSourceBuilder.create();
		builder.driverClassName("org.hr.Driver");
		builder.url("jdbc:h2:mem:test");
		builder.username("username");
		builder.password("password");
		return builder.build();
	}
}
```

或，使用`application.properties`文件外部化`DataSource`配置：

```java
@Bean
public DataSource getDataSource(){
	DataSourceBuilder builder = DataSourceBuilder.create().build();
}
```

`application.propertites`:

```properties
spring.datasource.url = jdbc:h2:mem:test
spring.datasource.driver-class-name = org.h2.Driver
spring.datasource.name = username
spring.datasource.password = password
```

- 类图：

![DataSourceBuilder-class-diagram类图](https://raw.githubusercontent.com/zkRou/note/master/DataSourceBuilder-class-diagram.PNG)

## DataSource数据源

- 在`javax.sql`包中，JDBC 2.0 API 中的新增内容。作为`DriverManager`工具的替代项，使用`DataSource`对象是连接到数据源的首选方法
- 负责建立与数据库的连接，获取连接
- 传统JDBC的工作流：
	- 加载数据库驱动程序(Mysql,SQL Server等)
	- 通过`DriverManager`获取`Connection`对象
	- 获取`Statement`对象
	- 执行SQL语句
	- 得到操作结果集`ResultSet`
	- 关闭资源
- 当使用`Spring Boot`并且使用`Spring Boot`默认的数据库配置时，`Spring Boot`的`DataSource`会自动配置，完成所有重型基础设施管道。这包括`H2 DataSource`实现，该实现将由`HikariCP`、`Apache Tomcat`或`Commons DBCP`自动处理，并设置内存数据库实例。此外，我们甚至不需要创建`application.properties`文件，因为`Spring Boot`也会提供一些默认的数据库设置。
- 类图：

![DataSource类图](https://raw.githubusercontent.com/zkRou/note/master/DataSource-class-diagram.PNG)

- 源码：

```java
public interface DataSource  extends CommonDataSource, Wrapper {
  
  Connection getConnection() throws SQLException;

  Connection getConnection(String username, String password) throws SQLException;
}

```

## SqlSession

- 在`org.apache.ibatis.session`包中
- `SqlSession`是`Mybatis`最重要的构建之一，通过`SqlSession`可以实现增删改查
- 生命周期是存在于请求数据库处理事务的过程中，是一个非线程安全的对象，即存活于一个应用的请求和申请，可以执行多条SQL保证事务的一致性

## SqlSessionFactory

- 在`org.apache.ibatis.session`包中
- `SqlSessionFactory`的作用是创建`SqlSession`，而`SqlSession`相当于JDBC的一个Connection对象，每次应用程序需要访问数据库，就要通过`SqlSessionFactory`创建一个`SqlSession`，所以`SqlSessionFactory`在整个Mybatis整个生命周期中（每个数据库对应一个`SqlSessionFactory`，是单例产生的）

## SqlSessionFactoryBuilder

- 在`org.apache.ibatis.session`包中
- 作用就是创建一个构建器，一旦创建了`SqlSessionFactory`，它的任务就完成了，可以回收

## SqlSessionFactoryBean

- 在`org.mybatis.spring`包中
- 在基本的`Mybatis`中，`SqlSessionFactory`可以使用`SqlSessionFactoryBuilder`来创建，而`mybatis-spring`则使用`SqlSessionFactoryBean`来创建

	- XML配置方式：
	```xml
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="mapperLocations" value="classpath*:mappers/**/*.xml" />
	</bean>
	```
	- Java方式：
	```java
	SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
	sqlSessionFactoryBean.setDataSource(dataSource);
	PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
	sqlSessionFactoryBean.setMapperLocations(resolver.getResource("classpath*:mybatis/mappers/*.xml"));
	SqlSessionFactory sessionFactory = sqlSessionFactoryBean.getObject();
	```
- 类图：

![SqlSessionFactoryBean类图](https://raw.githubusercontent.com/zkRou/note/master/SqlSessionFactoryBean-class-diagram.PNG)


# 相关链接
[A Guide to Spring AbstractRoutingDatasource](https://www.baeldung.com/spring-abstract-routing-data-source)