+++
author = "pikachu"
title = "Spring AOP实现自动读写分离"
date = "2019-05-29"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java"
]
categories = [
    "IT"
]
+++


## 一、实现原理

> 利用Spring AOP面向切面编程的特性，在请求进入service应用层之前，根据请求的service层的方法名，判断使用读库或者写库，如`find、get、list、query`开头的标记为读库，其他的标记为写库，实现读写的自动分离。

![1](https://user-images.githubusercontent.com/38284818/58495662-9e31d480-81aa-11e9-8cb3-472068a0513b.png)

&nbsp;

## 二、代码实现

**DataSource.java**
```
/**
 * 设置数据源注解//TODO
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface DataSource {

	// 写数据源
	public static final String WRITE = "write";
	
	// 读数据源
	public static final String READ = "read";

	String value() default "write";

}
```

**DataSourceHolder.java**
```
/**
 * 数据源操作
 */
public class DataSourceHolder {

	// 线程本地环境
	private static final ThreadLocal<String> dataSources = new ThreadLocal<String>();

	// 设置数据源
	public static void setDataSource(String customerType) {
		dataSources.set(customerType);
	}

	// 设置数据源为写库
	public static void setWrite() {
		dataSources.set("write");;
	}

	// 设置数据库为读库
	public static void setRead() {
		dataSources.set("read");
	}
	
	// 设置数据库为考勤数据库
	public static void setAttendance() {
		dataSources.set("attendanceDataSource");
	}

	// 获取数据源
	public static String getDataSource() {
		return (String) dataSources.get();
	}

	// 清除数据源
	public static void clearDataSource() {
		dataSources.remove();
	}

}
```

**DynamicDataSource.java**
```
/**
 * 获取数据源（依赖于spring）
 */
public class DynamicDataSource extends AbstractRoutingDataSource {

	@Override
	protected Object determineCurrentLookupKey() {
		return DataSourceHolder.getDataSource();
	}
	
}
```

**DataSourceAspect.java**
```
/**
 * 定义数据源的AOP切面，通过Service的方法名判断是应该走读库还是写库
 *
 * @since 2019/05/26
 * @author 曾博佳
 */
public class DataSourceAspect {
	
	private Logger log = LoggerFactory.getLogger(DataSourceAspect.class);
	
	/**
	 * 在进入
	 * 
	 * @param point
	 */
	public void before(JoinPoint point) {
		// 获取到当前执行的方法名
		String methodName = point.getSignature().getName();
		if(isRead(methodName)) {
			// 设置为读库
			DataSourceHolder.setRead();
			log.info("标记为读库");
		}else {
			// 设置为写库
			DataSourceHolder.setWrite();
			log.info("标记为写库");
		}
	}
	
	/**
	 * 判断是否为读操作
	 * 
	 * @param methodName
	 * @return
	 */
	public Boolean isRead(String methodName) {
		// 方法名以query、find、get、list开头的使用读库
		return StringUtils.startsWithAny(methodName, "query","find","get","list");
	}
}

```

**spring-service.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:aop="http://www.springframework.org/schema/aop"

	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/tx
	http://www.springframework.org/schema/tx/spring-tx.xsd
	http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd">
    
    <!-- 配置事务管理器 -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!-- 注入数据库连接池 -->
		<property name="dataSource" ref="dataSource" />
	</bean>
	<!-- 定义事务策略 -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<!--定义查询方法都是只读的 -->
			<tx:method name="query*" read-only="true" />
			<tx:method name="find*" read-only="true" />
			<tx:method name="get*" read-only="true" />
			<tx:method name="list*" read-only="true" />
 
			<!-- 主库执行操作，事务传播行为定义为默认行为 -->
			<tx:method name="save*" propagation="REQUIRED" />
			<tx:method name="update*" propagation="REQUIRED" />
			<tx:method name="delete*" propagation="REQUIRED" />
 
			<!--其他方法使用默认事务策略 -->
			<!-- <tx:method name="*" /> -->
		</tx:attributes>
	</tx:advice>
	
	<!-- 定义AOP切面处理器 -->
	<bean class="com.zhsj.common.db.DataSourceAspect" id="dataSourceAspect" />
	
	<aop:config>
		<!-- 定义切面，所有包的service的所有方法 -->
		<aop:pointcut id="txPointcut" expression="execution(* com.zhsj.*.service.*.*(..))" />
		<!-- 应用事务策略到Service切面 -->
		<aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
		
		<!-- 将切面应用到自定义的切面处理器上，-9999保证该切面优先级最高执行 -->
		<aop:aspect ref="dataSourceAspect" order="-9999">
			<aop:before method="before" pointcut-ref="txPointcut" />
		</aop:aspect>
	</aop:config>

```




