---
layout: post
title:  "IOC"
categories: j2ee
tags:  j2ee
---

* content
{:toc}

###一.Spring概况

* spring是一个开源框架
* 是一个轻量的控制反转和面向切面的容器框架
* 大小和开销都是轻量的。
* 通过控制反转技术可以达到松耦合的目的
* 切面编程，允许通过分离应用的业务逻辑。
* 包含并管理应用对象的配置和生命周期，是一个容器，并且能够组装。

<!--more-->

### 二、IoC

ioc控制反转：控制权转移，应用程序本身不负责依赖对象的创建和维护，而是由外部容器负责和维护。ioc的目的是创建对象并且组装对象之间的关系。

####1.bean容器初始化

*  --org.springframework.beans

* --org.springframework.context
*  beanfactory 提供配置结构和基本功能，加载并初始化bean
*  applicationContext 保存bean对象并在应用中被应用

#### 2.spring注入：

* spring 注入是指在启动 spring容器加载bean配置的时候，完成对变量的赋值行为。
* 常见的注入方式:设值注入、构建注入

举个例子，构建注入

dao层

```
public interface InjectionDAO {
	
	public void save(String arg);
	
}



public class InjectionDAOImpl implements InjectionDAO {
	
	public void save(String arg) {
		//模拟数据库保存操作
		System.out.println("保存数据：" + arg);
	}

}

```

service 层接口：


```
public interface InjectionService {
	
	public void save(String arg);
	
}


```
service 层实现：

```
public class InjectionServiceImpl implements InjectionService {
	
	private InjectionDAO injectionDAO;
	
	//构造器注入
	public InjectionServiceImpl(InjectionDAO injectionDAO1) {
		this.injectionDAO = injectionDAO1;
	}
	
	//设值注入
	public void setInjectionDAO(InjectionDAO injectionDAO) {
		this.injectionDAO = injectionDAO;
	}

	public void save(String arg) {
		//模拟业务操作
		System.out.println("Service接收参数：" + arg);
		arg = arg + ":" + this.hashCode();
		injectionDAO.save(arg);
	}

```

在spring xml中的配置

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
		<bean id="injectionService" class="com.forezp.ioc.injection.service.InjectionServiceImpl">
        	<constructor-arg name="injectionDAO1" ref="injectionDAO"></constructor-arg>
        </bean>

        <bean id="injectionDAO" class="com.forezp.ioc.injection.dao.InjectionDAOImpl"></bean>

 </beans>


```

单元测试

本系列文章单元测试基类


```

public class UnitTestBase {
	
	private ClassPathXmlApplicationContext context;
	
	private String springXmlpath;
	
	public UnitTestBase() {}
	
	public UnitTestBase(String springXmlpath) {
		this.springXmlpath = springXmlpath;
	}
	
	@Before
	public void before() {
		if (StringUtils.isEmpty(springXmlpath)) {
			springXmlpath = "classpath*:spring-*.xml";
		}
		try {
			context = new ClassPathXmlApplicationContext(springXmlpath.split("[,\\s]+"));
			context.start();
		} catch (BeansException e) {
			e.printStackTrace();
		}
	}
	
	@After
	public void after() {
		context.destroy();
	}
	
	@SuppressWarnings("unchecked")
	protected <T extends Object> T getBean(String beanId) {
		try {
			return (T)context.getBean(beanId);
		} catch (BeansException e) {
			e.printStackTrace();
			return null;
		}
	}
	
	protected <T extends Object> T getBean(Class<T> clazz) {
		try {
			return context.getBean(clazz);
		} catch (BeansException e) {
			e.printStackTrace();
			return null;
		}
	}

}

```

单元测试：

```

@RunWith(BlockJUnit4ClassRunner.class)
public class TestInjection extends UnitTestBase {
	
	public TestInjection() {
		super("classpath:spring-injection.xml");
	}
	
	@Test
	public void testSetter() {
		InjectionService service = super.getBean("injectionService");
		service.save("这是要保存的数据");
	}
	
	@Test
	public void testCons() {
		InjectionService service = super.getBean("injectionService");
		service.save("这是要保存的数据");
	}
	
}


```

运行打印：


>
>Service接收参数：这是要保存的数据

>保存数据：这是要保存的数据:1247298779
>


这个例子说明，我们可以通过ClassPathXmlApplicationContext.getBean()获取到了service，这个service 是通过xml配置注入到容器中，并且注入的时候通过构造函数的设置了成员变量dao。

### 三.bean的配置项

####3.1 bean常见的配置项，如下：

* Id
* Class
* Scope 
* Constructor arguments
* Properties
* Autowiring mode
* lazy-initialization mode
* Initialization/destruction method


#### 3.2 bean的作用域

* singleton: 单列
*  prototype每次使用都会创建新实例
*  request :每次http请求创建一个实例，仅在当前　request有效
*  session ： 当前session有效


举个例子：
测试sinleton和prototype

创建bean实例

```  
public class BeanScope {
	
	public void say() {
		System.out.println("BeanScope say : " + this.hashCode());
	}
	
}

```
在xml中配置,作用域为singleton


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
        <bean id="beanScope" class="com.imooc.bean.BeanScope" scope="singleton"></bean>
        
 </beans>


```

单元测试：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanScope extends UnitTestBase {
	
	public TestBeanScope() {
		super("classpath*:spring-beanscope.xml");
	}
	
	@Test
	public void testSay() {
		BeanScope beanScope = super.getBean("beanScope");
		beanScope.say();
		
		BeanScope beanScope2 = super.getBean("beanScope");
		beanScope2.say();
	}
	

}


```

运行单元测试：

>
>BeanScope say : 1113008012
>
BeanScope say : 1113008012

在xml中配置,作用域为prototype


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
        
        <bean id="beanScope" class="com.imooc.bean.BeanScope" scope="prototype"></bean>
        
 </beans>


```

运行单元测试

>
>BeanScope say : 144724468
>
BeanScope say : 1432645272

由此可发现sington在bean容器是一个实例，而prototype创建了二个实例。

### 四.bean的生命周期
包括以下几个方面：

* 定义，在xml中配置
* 初始化
* 使用
* 销毁

#### 初始化

有两种方式

* 实现 InitializingBeean接口，覆盖afterPropertiesSet()
* 配置init-method方法

#### 销毁

也有两种方式：

* 实现DisposableBean接口，覆盖destroy();
* 配置 destroy-method

举个例子:

创建bean实例：

```
public class BeanLifeCycle implements InitializingBean, DisposableBean {
	
	public void defautInit() {
		System.out.println("Bean defautInit.");
	}
	
	public void defaultDestroy() {
		System.out.println("Bean defaultDestroy.");
	}

	@Override
	public void destroy() throws Exception {
		System.out.println("Bean destroy.");
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("Bean afterPropertiesSet.");
	}
	
	public void start() {
		System.out.println("Bean start .");
	}
	
	public void stop() {
		System.out.println("Bean stop.");
	}
	
}

```

bean的配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-init-method="defautInit" default-destroy-method="defaultDestroy">
        
        <bean id="beanLifeCycle" class="com.imooc.lifecycle.BeanLifeCycle"  init-method="start" destroy-method="stop"></bean>
	
 </beans>


```

单元测试：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanLifecycle extends UnitTestBase {
	
	public TestBeanLifecycle() {
		super("classpath:spring-lifecycle.xml");
	}
	
	@Test
	public void test1() {
		super.getBean("beanLifeCycle");
	}
	
}


```

运行：

>Bean afterPropertiesSet.
>
>Bean start .
>
Bean destroy.

>Bean stop.

同时实现两种方式的初始化方法的执行顺序： 接口实现优先于xml中的配置。

### 五.bean的自动装配（Autowiring）


* No: 不做任何操作
* byname:根据属性名自动装配
* byType:如果容器存在一个与指定类型相同的bean，则自动装配，如果存在多个，则抛出异常。
* constructor:与 byType类似，不同之处它在于构造器的参数。

举例子：
1.byName方式：

创建一个dao:

```
public class AutoWiringDAO {
	
	public void say(String word) {
		System.out.println("AutoWiringDAO : " + word);
	}

}


```

创建一个service


```
public class AutoWiringService {
	
	private AutoWiringDAO autoWiringDAO;


	public void setAutoWiringDAO(AutoWiringDAO autoWiringDAO) {
		System.out.println("setAutoWiringDAO");
		this.autoWiringDAO = autoWiringDAO;
	}
	
	public void say(String word) {
		this.autoWiringDAO.say(word);
	}

}

```

在xml中配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="byName">
        
        <bean id="autoWiringService" class="com.imooc.autowiring.AutoWiringService" ></bean>
        
        <bean id="autoWiringDAO" class="com.imooc.autowiring.AutoWiringDAO" ></bean>
	
 </beans>

```

单元测试：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestAutoWiring extends UnitTestBase {
	
	public TestAutoWiring() {
		super("classpath:spring-autowiring.xml");
	}
	
	@Test
	public void testSay() {
		AutoWiringService service = super.getBean("autoWiringService");
		service.say(" this is a test");
	}

}

```

运行：

> setAutoWiringDAO
> 
AutoWiringDAO :  this is a test

通过default-autowire="byName";
AutoWiringService 自动获取了autoWiringDAO的实例。


2.byTYpe

将在xml中配置改为byType ：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="byName">
        
        <bean id="autoWiringService" class="com.imooc.autowiring.AutoWiringService" ></bean>
        
        <bean id="autoWiringDAO" class="com.imooc.autowiring.AutoWiringDAO" ></bean>
	
 </beans>

```
其他不变，运行和byName 一样。

3.constructor

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" 
        default-autowire="constructor">
        
        <bean id="autoWiringService" class="com.imooc.autowiring.AutoWiringService" ></bean>
        
        <bean id="autoWiringDAO" class="com.imooc.autowiring.AutoWiringDAO" ></bean>
	
 </beans>

```

AutoWiringService 中增加构造器

```

public class AutoWiringService {
	
	private AutoWiringDAO autoWiringDAO;
	
	public AutoWiringService(AutoWiringDAO autoWiringDAO) {
		System.out.println("AutoWiringService");
		this.autoWiringDAO = autoWiringDAO;
	}

	public void setAutoWiringDAO(AutoWiringDAO autoWiringDAO) {
		System.out.println("setAutoWiringDAO");
		this.autoWiringDAO = autoWiringDAO;
	}
	
	public void say(String word) {
		this.autoWiringDAO.say(word);
	}

}


```

允行：

> AutoWiringDAO :  this is a test
> 


### 六.classPath扫描与组件管理

从spring 3.0开始，spring javaConfig 项目提供了许多特性，包括使用java而不是xml

1.比如注解

> @Configuration
> @Bean
> @Import
> @DependsOn
> 
> @Component 是一个通用注解，应用于任何bean
> 
> @Reposity注解DAO
> 
> @Service注解service
> 
> @Controller注解controller
> 
> 


2.spring可以自动检测类并注册bean到applicationContext中。比如
@Service  @Reposity等

3.< context:annoation-config />会查找applicationContext中bean的注解。

扫描 ：< context:component-scan>  包含< context:annoation-config/>，通常只需要使用前者。

```
<context:component-scan base-package="com.forezp" > 
```



举个例子：
通过扫描获取bean,在xml中的配置：


```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" >
        
        <context:component-scan base-package="com.forezp.beanannotation"></context:component-scan>
        
 </beans>

```
定义一个bean类：
其中scope 注解表示bean的作用域，默认singleton。Component默认类名并将第一个字母小写。

```
@Scope
@Component
public class BeanAnnotation {
	
	public void say(String arg) {
		System.out.println("BeanAnnotation : " + arg);
	}
	
	public void myHashCode() {
		System.out.println("BeanAnnotation : " + this.hashCode());
	}
	
}

```

单元测试：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestBeanAnnotation extends UnitTestBase {
	
	public TestBeanAnnotation() {
		super("classpath*:spring-beanannotation.xml");
	}
	
	@Test
	public void testSay() {
		BeanAnnotation bean = super.getBean("beanAnnotation");
		bean.say("This is test.");
		
		//bean = super.getBean("bean");
		//bean.say("This is test.");
	}

}
```

运行：
 >BeanAnnotation : This is test.
 
  
### 七、Autowired
 
* @Autowired可以用于setter方法上
* 可以用于成员变量
* 可以用于构造器

举个例子：

采用包扫描：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd" >
        
        <context:component-scan base-package="com.forezp.beanannotation"></context:component-scan>
        
 </beans>


```

采用注解：DAO层

```
@Repository
public class InjectionDAOImpl implements InjectionDAO {
	
	public void save(String arg) {
		//模拟数据库保存操作
		System.out.println("保存数据：" + arg);
	}

}


```


service层：

```
@Service
public class InjectionServiceImpl implements InjectionService {
	
//	@Autowired
	private InjectionDAO injectionDAO;
	
	@Autowired
	public InjectionServiceImpl(InjectionDAO injectionDAO) {
		this.injectionDAO = injectionDAO;
	}
	
//	@Autowired
	public void setInjectionDAO(InjectionDAO injectionDAO) {
		this.injectionDAO = injectionDAO;
	}



	public void save(String arg) {
		//模拟业务操作
		System.out.println("Service接收参数：" + arg);
		arg = arg + ":" + this.hashCode();
		injectionDAO.save(arg);
	}
	
}

```

 单元测试：
 
 ```
 @RunWith(BlockJUnit4ClassRunner.class)
public class TestInjection extends UnitTestBase {
	
	public TestInjection() {
		super("classpath:spring-beanannotation.xml");
	}
	
	@Test
	public void testAutowired() {
		InjectionService service = super.getBean("injectionServiceImpl");
		service.save("This is autowired.");
	}
}
 
 ```
 
 运行：
 
 > Service接收参数：This is autowired.
 
 > 保存数据：This is autowired.:1641742937
 
 
 
### 八、基于java的容器注解@Bean

* @Bean 标识一个用于配置和初始化一个由springIoC容器管理的新对象的方法，类似于　xml配置文件的</ bean>

举个例子：
用注解去代替xml文件


```
@Configuration

public class StoreConfig {
	

@Bean(name = "stringStore", initMethod="init", destroyMethod="destroy")
	public Store stringStore() {
		return new StringStore();
	}
	
```

javabean StringStore类

```

public class StringStore implements Store<String> {
	
	public void init() {
		System.out.println("This is init.");
	}
	
	public void destroy() {
		System.out.println("This is destroy.");
	}
	
}

```
单元测试：

```
@RunWith(BlockJUnit4ClassRunner.class)
public class TestJavabased extends UnitTestBase {
	
	public TestJavabased() {
		super("classpath*:spring-beanannotation.xml");
	}
	
	@Test
	public void test() {
		Store store = super.getBean("stringStore");
		System.out.println(store.getClass().getName());
	}
	
	}

```

另外可以用ImportResource注解类获取资源文件信息：

```
@Configuration
@ImportResource("classpath:config.xml")
public class StoreConfig {
	
	
	@Value("${url}")
	private String url;
	
	@Value("${jdbc.username}")
	private String username;
	
	@Value("${password}")
	private String password;
	
```

### 九、JSR-250

* spring支持jsr-250 
* @Resource注解变量或者setter 方法
* Resource注解有一个name属性，默认该值作为被注入bean的名称。

举个例子：

Dao层：


```
@Repository
public class JsrDAO {
	
	public void save() {
		System.out.println("JsrDAO invoked.");
	}
	
}


```

service层：

```


@Service
public class JsrServie {
	
	@Resource
	private JsrDAO jsrDAO;
	
//	@Resource
//	public void setJsrDAO(@Named("jsrDAO") JsrDAO jsrDAO) {
	//	this.jsrDAO = jsrDAO;
	//}
	
	@PostConstruct
	public void init() {
		System.out.println("JsrServie init.");
	}
	
	@PreDestroy
	public void destroy() {
		System.out.println("JsrServie destroy.");
	}

	public void save() {
		jsrDAO.save();
	}
	
}

```