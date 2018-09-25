---
layout: post
title:  "JWT如何在Spring Cloud微服务系统中在服务相互调时传递"
categories: SpringCloud 
tags:  SpringCloud JWT
---

* content
{:toc}


在微服务系统中，为了保证微服务系统的安全,常常使用jwt来鉴权，但是服务内部的相互调用呢。经常有人在微信上问我，我给出一个解决办法，采用Feign的拦截器。

<!--more-->

在Feign中开启了hystrix，hystrix默认采用的是线程池作为隔离策略。线程隔离有一个难点需要处理，即隔离的线程无法获取当前请求线程的Jwt，这用ThredLocal类可以去解决，但是比较麻烦，所以我才用的是信号量模式。
在application.yml配置文件中使用一下配置：


```
hystrix.command.default.execution.isolation.strategy: SEMAPHORE
```

写一个Feign的拦截器，Feign在发送网络请求之前会执行以下的拦截器，代码如下：

```

import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.stereotype.Component;

/**
 * Created by fangzhipeng on 2017/7/28.
 */
@Component
public class JwtFeignInterceptor implements RequestInterceptor {

    private final String key = "Authorization";


    @Override
    public void apply(RequestTemplate template) {

        if (!template.headers().containsKey(key)) {
            String currentToken = UserUtils.getCurrentToken();
            if (!StrUtil.isEmpty(currentToken)){
                template.header(key, currentToken);
            }
        }
    }
}
```