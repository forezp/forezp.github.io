---
layout: post
title:  "AOP"
categories: j2ee
tags:  j2ee
---

* content
{:toc}

### 一、概述
Aop(aspect oriented programming面向切面编程），是spring框架的另一个特征。AOP包括切面、连接点、通知（advice）、切入点（pointCut) 。

<!--more-->

1.aop几个概念：

* 横切关注点： 对哪些方面进行拦截，拦截后怎么处理。
* 切面(aspect):切面是横切关注点的抽象。
* 连接点(joinpoint)：被拦截的方法
* 切入点(pointcut):对连接点进行拦截的定义。
* 通知(advice)：拦截到连接点之后要执行的代码
* 目标对象：代理的目标对象
* 织入
* 引入


2.主要功能：

*  日志记录
*  性能统计
*  安全控制
*  事物处理
*  异常处理

3.advice类型：

* 前置通知(before advice)
* 返回后通知(after  returning advice)
* 抛出异常后通知(after throwing advice)
* 后通知(after advice)
*  环绕通知(around advice)


4.Spring对AOP的支持

Spring中AOP代理由Spring的IOC容器负责生成、管理，其依赖关系也由IOC容器负责管理。因此，AOP代理可以直接使用容器中的其它bean实例作为目标，这种关系可由IOC容器的依赖注入提供。 

二、基于xml配置的aop

 在spring基于schemel中，aop需要声明一个切面aspect,一个pointcut,一个advisor.
 举个例子：
 
 切面：
 
 ```
 public class MoocAspect {
	
	public void before() {
		System.out.println("MoocAspect before.");
	}
	
	}
 
 ```
 
 切点：
 
 ```
 public class AspectBiz {
	
	public void biz() {
		System.out.println("AspectBiz biz.");
//		throw new RuntimeException();
	}
 
 ```
 
 配置文件：
 
 ```
 
 <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

	<bean id="moocAspect" class="com.imooc.aop.schema.advice.MoocAspect"></bean>
	
	<bean id="aspectBiz" class="com.imooc.aop.schema.advice.biz.AspectBiz"></bean>
	
	<aop:config>
		<aop:aspect id="moocAspectAOP" ref="moocAspect">
			<aop:pointcut expression="execution(* com.imooc.aop.schema.advice.biz.*Biz.*(..))" id="moocPiontcut"/>
			<aop:before method="before" pointcut-ref="moocPiontcut"/>
				</aop:aspect>
	</aop:config>

 </beans>

 
 ```
 
 单元测试：
 
 ```
 @RunWith(BlockJUnit4ClassRunner.class)
public class TestAOPSchemaAdvice extends UnitTestBase {
	
	public TestAOPSchemaAdvice() {
		super("classpath:spring-aop-schema-advice.xml");
	}
	
	@Test
	public void testBiz() {
		AspectBiz biz = super.getBean("aspectBiz");
		biz.biz();
	}
}
 
 ```
 
 运行：
 
 >MoocAspect before.
 
>AspectBiz biz.


这是 机遇 schemel配置使用aop，其实在spring 1.2版本是有api的，基于api配置aop很麻烦，但是也也应该了解下

### 三、基于spring api方式配置aop

直接上代码：

接口中有两个方法，一个基于aop,会被拦截 ，另外一个不会被拦截。

```
public interface IAopService {

	public void withAop() throws Exception;

	public void withoutAop() throws Exception;

}

```

实现类：

``` 
public class AopServiceImpl implements IAopService {

	private String name="forezp";

	public void withAop() throws Exception {

		System.out.println("with aop run: " + name);

		if (name.trim().length() == 0)
			throw new AccountException("name cannot be null");
	}

	public void withoutAop() throws Exception {
		System.out.println("without aop");
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

}

```

方法前拦截器，实现MethodBeforeAdvice,在制定方法前会被调用。

```

public class MethodBeforeInterceptor implements MethodBeforeAdvice {

	public void before(Method method, Object[] args, Object instance)
			throws Throwable {

		System.out.println("method invoke:" + method.getName());

		if (instance instanceof AopServiceImpl) {

			String name = ((AopServiceImpl) instance).getName();

			if (name == null)
				throw new NullPointerException("name cannot be null");
		}

	}

}
```

返回后拦截器，执行指定方法后会被调用。

```
public class MethodAfterInterceptor implements AfterReturningAdvice {

	public void afterReturning(Object value, Method method, Object[] args,
			Object instance) throws Throwable {

		System.out.println("method  " + method.getName() + "had finished and return value-" + value);

	}

}


```

异常拦截器，当出现异常时拦截：

```

public class ThrowsInterceptor implements ThrowsAdvice {

	public void afterThrowing(Method method, Object[] args, Object instance,
			AccountException ex) throws Throwable {

		System.out.println("method" + method.getName() + " throws exception:" + ex);
	}

	public void afterThrowing(NullPointerException ex) throws Throwable {

		System.out.println("the exception:" + ex);
	}

}


```

三个拦截器和Servce实现类需要配置到spring中。实际上，spring无法组装，需要借助代理类，把拦截器安装到NameMatchMethodPointcutAdvisor中，把自定义的bean安装到ProxyFactoryBean中，然后组装在一起：


```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd">

	<!-- 拦截器 在 withAop() 方法前运行 -->
	<bean id="aopMethodBeforeInterceptor"
		class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">
		<property name="advice">
			<bean
				class="com.imooc.aop.example2.MethodBeforeInterceptor" />
		</property>
		<property name="mappedName" value="withAop"></property>
	</bean>

	<!-- 拦截器 在 withAop() 返回后运行 -->
	<bean id="aopMethodAfterInterceptor"
		class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">
		<property name="advice">
			<bean
				class="com.imooc.aop.example2.MethodAfterInterceptor" />
		</property>
		<property name="mappedName" value="withAop"></property>
	</bean>

	<!-- 拦截器 在异常抛出后运行 -->
	<bean id="aopThrowsInterceptor"
		class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">
		<property name="advice">
			<bean
				class="com.imooc.aop.example2.ThrowsInterceptor" />
		</property>
		<property name="mappedName" value="withAop"></property>
	</bean>

	<bean id="aopService"
		class="org.springframework.aop.framework.ProxyFactoryBean">
		<!-- 拦截器 -->
		<property name="interceptorNames">
			<list>
				<value>aopMethodBeforeInterceptor</value>
				<value>aopMethodAfterInterceptor</value>
				<value>aopThrowsInterceptor</value>
			</list>
		</property>
		<!-- 被拦截的对象 -->
		<property name="target">
			<bean
				class="com.imooc.aop.example2.AopServiceImpl">
	
					<property name="name" value="forezp"></property>
			
			</bean>
		</property>
	</bean>

</beans>

```

单元测试：

```


```

> method invoke:withAop
> 
>with aop run: forezp
>
>method  withAophad finished and return value-forezp
>
>without aop