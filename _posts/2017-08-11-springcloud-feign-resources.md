---
layout: post
title:  "Feign源码解析"
categories: SpringCloud
tags:  SpringCloud Feign
---

* content
{:toc}

## 什么是Feign

Feign是由Retrofit，JAXRS-2.0和WebSocket启发的一个java到http客户端绑定。 Feign的主要目标是将Java Http Clients变得简单。Feign的源码地址：https://github.com/OpenFeign/feign

<!--more-->

## 写一个Feign

在我之前的博文有写到如何用Feign去消费服务，文章地址：http://blog.csdn.net/forezp/article/details/69808079  。

简单的实现一个Feign客户端，首先通过@FeignClient，客户端，其中value为调用其他服务的名称，FeignConfig.class为FeignClient的配置文件，代码如下：

```

@FeignClient(value = "service-hi",configuration = FeignConfig.class)
public interface SchedualServiceHi {
    @GetMapping(value = "/hi")
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}

```
其配置文件如下：

```
@Configuration
public class FeignConfig {

    @Bean
    public Retryer feignRetryer() {
        return new Retryer.Default(100, SECONDS.toMillis(1), 5);
    }
    
}


```

查看FeignClient的源码，其代码如下：

```

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FeignClient {

@AliasFor("name")
String value() default "";
	
@AliasFor("value")
String name() default "";
	
@AliasFor("value")
String name() default "";
String url() default "";
boolean decode404() default false;

Class<?>[] configuration() default {};
Class<?> fallback() default void.class;

Class<?> fallbackFactory() default void.class;
}

String path() default "";

boolean primary() default true;

```

feign 用于声明具有该接口的REST客户端的接口的注释应该是创建（例如用于自动连接到另一个组件。 如果功能区可用，那将是
用于负载平衡后端请求，并且可以配置负载平衡器
使用与伪装客户端相同名称（即值）@RibbonClient 。

其中value()和name()一样，是被调用的 service的名称。
url(),直接填写硬编码的url,decode404()即404是否被解码，还是抛异常；configuration()，标明FeignClient的配置类，默认的配置类为FeignClientsConfiguration类，可以覆盖Decoder、Encoder和Contract等信息，进行自定义配置。fallback(),填写熔断器的信息类。

## FeignClient的配置



默认的配置类为FeignClientsConfiguration，这个类在spring-cloud-netflix-core的jar包下，打开这个类，可以发现它是一个配置类，注入了很多的相关配置的bean，包括feignRetryer、FeignLoggerFactory、FormattingConversionService等,其中还包括了Decoder、Encoder、Contract，如果这三个bean在没有注入的情况下，会自动注入默认的配置。

* Decoder feignDecoder: ResponseEntityDecoder(这是对SpringDecoder的封装)
* Encoder feignEncoder: SpringEncoder
* Logger feignLogger: Slf4jLogger
* Contract feignContract: SpringMvcContract
* Feign.Builder feignBuilder: HystrixFeign.Builder

代码如下：

```
@Configuration
public class FeignClientsConfiguration {

...//省略代码

@Bean
	@ConditionalOnMissingBean
	public Decoder feignDecoder() {
		return new ResponseEntityDecoder(new SpringDecoder(this.messageConverters));
	}

	@Bean
	@ConditionalOnMissingBean
	public Encoder feignEncoder() {
		return new SpringEncoder(this.messageConverters);
	}

	@Bean
	@ConditionalOnMissingBean
	public Contract feignContract(ConversionService feignConversionService) {
		return new SpringMvcContract(this.parameterProcessors, feignConversionService);
	}

...//省略代码
}


```

重写配置：

你可以重写FeignClientsConfiguration中的bean，从而达到自定义配置的目的，比如FeignClientsConfiguration的默认重试次数为Retryer.NEVER_RETRY，即不重试，那么希望做到重写，代码如下：

```
@Configuration
public class FeignConfig {

    @Bean
    public Retryer feignRetryer() {
        return new Retryer.Default(100, SECONDS.toMillis(1), 5);
    }

}

```

在上述代码更改了该FeignClient的重试次数，重试间隔为100ms，最大重试时间为1s,重试次数为5次。


## Feign的工作原理

feign是一个伪客户端，即它不做任何的请求处理。Feign通过处理注解生成request，从而实现简化HTTP API开发的目的，即开发人员可以使用注解的方式定制request api模板，在发送http request请求之前，feign通过处理注解的方式替换掉request模板中的参数，这种实现方式显得更为直接、可理解。

通过包扫描注入FeignClient的bean，该源码在FeignClientsRegistrar类：
首先在启动配置上检查是否有@EnableFeignClients注解，如果有该注解，则开启包扫描，扫描被@FeignClient注解接口。代码如下：

```
private void registerDefaultConfiguration(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
		Map<String, Object> defaultAttrs = metadata
				.getAnnotationAttributes(EnableFeignClients.class.getName(), true);

		if (defaultAttrs != null && defaultAttrs.containsKey("defaultConfiguration")) {
			String name;
			if (metadata.hasEnclosingClass()) {
				name = "default." + metadata.getEnclosingClassName();
			}
			else {
				name = "default." + metadata.getClassName();
			}
			registerClientConfiguration(registry, name,
					defaultAttrs.get("defaultConfiguration"));
		}
	}


```

扫描到FeignClient后，将信息取出，以bean的形式注入到ioc容器中，源码如下：


```
public void registerFeignClients(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
		ClassPathScanningCandidateComponentProvider scanner = getScanner();
		scanner.setResourceLoader(this.resourceLoader);

		Set<String> basePackages;

		Map<String, Object> attrs = metadata
				.getAnnotationAttributes(EnableFeignClients.class.getName());
		AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
				FeignClient.class);
		final Class<?>[] clients = attrs == null ? null
				: (Class<?>[]) attrs.get("clients");
		if (clients == null || clients.length == 0) {
			scanner.addIncludeFilter(annotationTypeFilter);
			basePackages = getBasePackages(metadata);
		}
		else {
			final Set<String> clientClasses = new HashSet<>();
			basePackages = new HashSet<>();
			for (Class<?> clazz : clients) {
				basePackages.add(ClassUtils.getPackageName(clazz));
				clientClasses.add(clazz.getCanonicalName());
			}
			AbstractClassTestingTypeFilter filter = new AbstractClassTestingTypeFilter() {
				@Override
				protected boolean match(ClassMetadata metadata) {
					String cleaned = metadata.getClassName().replaceAll("\\$", ".");
					return clientClasses.contains(cleaned);
				}
			};
			scanner.addIncludeFilter(
					new AllTypeFilter(Arrays.asList(filter, annotationTypeFilter)));
		}

		for (String basePackage : basePackages) {
			Set<BeanDefinition> candidateComponents = scanner
					.findCandidateComponents(basePackage);
			for (BeanDefinition candidateComponent : candidateComponents) {
				if (candidateComponent instanceof AnnotatedBeanDefinition) {
					// verify annotated class is an interface
					AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
					AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
					Assert.isTrue(annotationMetadata.isInterface(),
							"@FeignClient can only be specified on an interface");

					Map<String, Object> attributes = annotationMetadata
							.getAnnotationAttributes(
									FeignClient.class.getCanonicalName());

					String name = getClientName(attributes);
					registerClientConfiguration(registry, name,
							attributes.get("configuration"));

					registerFeignClient(registry, annotationMetadata, attributes);
				}
			}
		}
	}


private void registerFeignClient(BeanDefinitionRegistry registry,
			AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
		String className = annotationMetadata.getClassName();
		BeanDefinitionBuilder definition = BeanDefinitionBuilder
				.genericBeanDefinition(FeignClientFactoryBean.class);
		validate(attributes);
		definition.addPropertyValue("url", getUrl(attributes));
		definition.addPropertyValue("path", getPath(attributes));
		String name = getName(attributes);
		definition.addPropertyValue("name", name);
		definition.addPropertyValue("type", className);
		definition.addPropertyValue("decode404", attributes.get("decode404"));
		definition.addPropertyValue("fallback", attributes.get("fallback"));
		definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
		definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

		String alias = name + "FeignClient";
		AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();

		boolean primary = (Boolean)attributes.get("primary"); // has a default, won't be null

		beanDefinition.setPrimary(primary);

		String qualifier = getQualifier(attributes);
		if (StringUtils.hasText(qualifier)) {
			alias = qualifier;
		}

		BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
				new String[] { alias });
		BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
	}



```

注入bean之后，通过jdk的代理，当请求Feign Client的方法时会被拦截，代码在ReflectiveFeign类，代码如下：

```
 public <T> T newInstance(Target<T> target) {
    Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);
    Map<Method, MethodHandler> methodToHandler = new LinkedHashMap<Method, MethodHandler>();
    List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList<DefaultMethodHandler>();

    for (Method method : target.type().getMethods()) {
      if (method.getDeclaringClass() == Object.class) {
        continue;
      } else if(Util.isDefault(method)) {
        DefaultMethodHandler handler = new DefaultMethodHandler(method);
        defaultMethodHandlers.add(handler);
        methodToHandler.put(method, handler);
      } else {
        methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
      }
    }
    InvocationHandler handler = factory.create(target, methodToHandler);
    T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(), new Class<?>[]{target.type()}, handler);

    for(DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {
      defaultMethodHandler.bindTo(proxy);
    }
    return proxy;
  }

```

在SynchronousMethodHandler类进行拦截处理，当被拦截会根据参数生成RequestTemplate对象，该对象就是http请求的模板，代码如下：

```
 @Override
  public Object invoke(Object[] argv) throws Throwable {
    RequestTemplate template = buildTemplateFromArgs.create(argv);
    Retryer retryer = this.retryer.clone();
    while (true) {
      try {
        return executeAndDecode(template);
      } catch (RetryableException e) {
        retryer.continueOrPropagate(e);
        if (logLevel != Logger.Level.NONE) {
          logger.logRetry(metadata.configKey(), logLevel);
        }
        continue;
      }
    }
  }

```

其中有个executeAndDecode()方法，该方法是通RequestTemplate生成Request请求对象，然后根据用client获取response。

```
  Object executeAndDecode(RequestTemplate template) throws Throwable {
    Request request = targetRequest(template);
    ...//省略代码
    response = client.execute(request, options);
    ...//省略代码

}

```

## Client组件

其中Client组件是一个非常重要的组件，Feign最终发送request请求以及接收response响应，都是由Client组件完成的，其中Client的实现类，只要有Client.Default，该类由HttpURLConnnection实现网络请求，另外还支持HttpClient、Okhttp.

首先来看以下在FeignRibbonClient的自动配置类，FeignRibbonClientAutoConfiguration ，主要在工程启动的时候注入一些bean,其代码如下：

```
@ConditionalOnClass({ ILoadBalancer.class, Feign.class })
@Configuration
@AutoConfigureBefore(FeignAutoConfiguration.class)
public class FeignRibbonClientAutoConfiguration {

@Bean
	@ConditionalOnMissingBean
	public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
			SpringClientFactory clientFactory) {
		return new LoadBalancerFeignClient(new Client.Default(null, null),
				cachingFactory, clientFactory);
	}

}

```

在缺失配置feignClient的情况下，会自动注入new Client.Default(),跟踪Client.Default()源码，它使用的网络请求框架为HttpURLConnection，代码如下：

```
  @Override
    public Response execute(Request request, Options options) throws IOException {
      HttpURLConnection connection = convertAndSend(request, options);
      return convertResponse(connection).toBuilder().request(request).build();
    }

```

怎么在feign中使用HttpClient，查看FeignRibbonClientAutoConfiguration的源码

```
@ConditionalOnClass({ ILoadBalancer.class, Feign.class })
@Configuration
@AutoConfigureBefore(FeignAutoConfiguration.class)
public class FeignRibbonClientAutoConfiguration {
...//省略代码

@Configuration
	@ConditionalOnClass(ApacheHttpClient.class)
	@ConditionalOnProperty(value = "feign.httpclient.enabled", matchIfMissing = true)
	protected static class HttpClientFeignLoadBalancedConfiguration {

		@Autowired(required = false)
		private HttpClient httpClient;

		@Bean
		@ConditionalOnMissingBean(Client.class)
		public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
				SpringClientFactory clientFactory) {
			ApacheHttpClient delegate;
			if (this.httpClient != null) {
				delegate = new ApacheHttpClient(this.httpClient);
			}
			else {
				delegate = new ApacheHttpClient();
			}
			return new LoadBalancerFeignClient(delegate, cachingFactory, clientFactory);
		}
	}

...//省略代码
}

```

从代码@ConditionalOnClass(ApacheHttpClient.class)注解可知道，只需要在pom文件加上HttpClient的classpath就行了，另外需要在配置文件上加上feign.httpclient.enabled为true，从	@ConditionalOnProperty注解可知，这个可以不写，在默认的情况下就为true.

在pom文件加上：

```

<dependency>
    <groupId>com.netflix.feign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>RELEASE</version>
</dependency>

```
 
 
同理，如果想要feign使用Okhttp，则只需要在pom文件上加上feign-okhttp的依赖：

```
<dependency>
    <groupId>com.netflix.feign</groupId>
    <artifactId>feign-okhttp</artifactId>
    <version>RELEASE</version>
</dependency>

```

## feign的负载均衡是怎么样实现的呢？


通过上述的FeignRibbonClientAutoConfiguration类配置Client的类型(httpurlconnection，okhttp和httpclient)时候，可知最终向容器注入的是LoadBalancerFeignClient，即负载均衡客户端。现在来看下LoadBalancerFeignClient的代码：

```
	@Override
	public Response execute(Request request, Request.Options options) throws IOException {
		try {
			URI asUri = URI.create(request.url());
			String clientName = asUri.getHost();
			URI uriWithoutHost = cleanUrl(request.url(), clientName);
			FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
					this.delegate, request, uriWithoutHost);

			IClientConfig requestConfig = getClientConfig(options, clientName);
			return lbClient(clientName).executeWithLoadBalancer(ribbonRequest,
					requestConfig).toResponse();
		}
		catch (ClientException e) {
			IOException io = findIOException(e);
			if (io != null) {
				throw io;
			}
			throw new RuntimeException(e);
		}
	}
```

其中有个executeWithLoadBalancer()方法，即通过负载均衡的方式请求。

```
  public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
        RequestSpecificRetryHandler handler = getRequestSpecificRetryHandler(request, requestConfig);
        LoadBalancerCommand<T> command = LoadBalancerCommand.<T>builder()
                .withLoadBalancerContext(this)
                .withRetryHandler(handler)
                .withLoadBalancerURI(request.getUri())
                .build();

        try {
            return command.submit(
                new ServerOperation<T>() {
                    @Override
                    public Observable<T> call(Server server) {
                        URI finalUri = reconstructURIWithServer(server, request.getUri());
                        S requestForServer = (S) request.replaceUri(finalUri);
                        try {
                            return Observable.just(AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));
                        } 
                        catch (Exception e) {
                            return Observable.error(e);
                        }
                    }
                })
                .toBlocking()
                .single();
        } catch (Exception e) {
            Throwable t = e.getCause();
            if (t instanceof ClientException) {
                throw (ClientException) t;
            } else {
                throw new ClientException(e);
            }
        }
        
    }	

```

其中服务在submit()方法上，点击submit进入具体的方法,这个方法是LoadBalancerCommand的方法：


```
     Observable<T> o = 
                (server == null ? selectServer() : Observable.just(server))
                .concatMap(new Func1<Server, Observable<T>>() {
                    @Override
                    // Called for each server being selected
                    public Observable<T> call(Server server) {
                        context.setServer(server);
    
        }}

```

上述代码中有个selectServe()，该方法是选择服务的进行负载均衡的方法，代码如下：

```
    private Observable<Server> selectServer() {
        return Observable.create(new OnSubscribe<Server>() {
            @Override
            public void call(Subscriber<? super Server> next) {
                try {
                    Server server = loadBalancerContext.getServerFromLoadBalancer(loadBalancerURI, loadBalancerKey);   
                    next.onNext(server);
                    next.onCompleted();
                } catch (Exception e) {
                    next.onError(e);
                }
            }
        });
    }

```

最终负载均衡交给loadBalancerContext来处理，即之前讲述的Ribbon，在这里不再重复。

## 总结

总到来说，Feign的源码实现的过程如下：

* 首先通过@EnableFeignCleints注解开启FeignCleint
* 根据Feign的规则实现接口，并加@FeignCleint注解
* 程序启动后，会进行包扫描，扫描所有的@ FeignCleint的注解的类，并将这些信息注入到ioc容器中。
* 当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplate
* RequesTemplate在生成Request
* Request交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp
* 最后Client被封装到LoadBalanceClient类，这个类结合类Ribbon做到了负载均衡。

## 参考资料

https://github.com/OpenFeign/feign

https://blog.de-swaef.eu/the-netflix-stack-using-spring-boot-part-3-feign/