---
title: Spring-retry重试机制
author: Kairou Zeng
date: 2019/02/01
tags: 
    - spring-retry
categories:
    - Spring
description: spring retry是从spring batch独立出来的一个功能，主要实现了重试和熔断，提供了自动并重复调用操作执行失败的能力。
---

## 框架介绍

`spring retry`是从`spring batch`独立出来的一个功能，主要实现了重试和熔断，提供了自动并重复调用操作执行失败的能力。

### 使用场景

调用第三方接口或消息队列及一些网络抖动、连接超时等网络异常，此时就需要重试。（区块链中提交交易发生失败时，也可以考虑用重试机制，降低服务接入方交易失败的概率）使程序/接口更健壮。

### 依赖

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

`spring retry`用到了aspect增强，因此可能需要引入以下依赖：

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```

### 常用注解

**@EnableRetry**：开启retry的拦截，一般在启动类（如：Application.java）上注解

**@Retryable**：注解在需要重试的方法上，被注解的方法发生异常时会重试，参数说明：

- value：抛出指定的异常才会重试，默认为空

- include：和value一样，默认为空，当exclude也为空时，所有异常都重试

- exclude：指定异常不重试，默认为空，当include也为空时，所有异常都重试

- maxAttempts：最大尝试次数，默认3次

- backoff：重试等待策略，默认使用@Backoff：
    
    - delay(等同于value)-指定延迟后重试，默认为1000L，
    当未设置multiplier时，表示每隔delay的时间重试，直到重试次数到达maxAttempts设置的最大允许重试次数，
    当设置了multiplier参数时，该值作为幂运算的初始值（delay = delay * multiplier）；

    - multiplier-指定延迟倍数，默认为0，表示固定暂停1秒后进行重试。比如delay=5000L,multiplier=2时，第一次重试为5秒后，第二次为10秒，第三次为20秒。

```java
@Retryable(
      value = {SQLException.class}, 
      maxAttempts = 2,
      backoff = @Backoff(delay = 5000L, multiplier=1.5)
      )
```

**@Recover**：用于`@Retryable`重试失败后的处理方法。当重试到达指定次数时，被注解的方法被回调，但需要注意只有入参类型和发生异常的类型一致时才会回调。

### 关键API

- **RetryTemplate**：`RetryTemplate`实现了`RetryOperations`接口，简化了操作。重试操作被封装在`RetryCallback`接口的实现中，并使用提供的其中一种`execute()`方法执行。

基本回调是一个简单的接口，允许插入一些需要重试的业务逻辑：

```java
public interface RetryCallback<T>{
    T doWithRetry(RetryContext context) throws Throwable;
}
```

`RetryOperations`最简单的通用实现是`RetryTemplate`:

```java
RetryTemplate template = new RetryTemplate();

TimeoutRetryPolicy policy = new TimeoutRetryPolicy();
policy.setTimeout(30000L);

template.setRetryPolicy(policy);

Foo result = template.execute(new RetryCallback<Foo>(){
    publicFoo doWithRetry(RetryContext context){
        //一些会出错的操作
        return result;
    }
});
```

- **RetryContext**：是`RetryCallback`的方法参数。许多回调只会忽略上下文，可以将其用作属性包，以便在迭代期间存储数据。如果在同一线程中正在进行嵌套重试，则`RetryContext`将具有父上下文。父上下文有时用于存储需要在要执行的调用之间共享的数据。

- **RecoveryCallback**：当重试耗尽时，`RetryOperations`可以将控制权传递给另一个回调，即`RecoveryCallback`。例如，要使用此特性，客户端只需将回调一起传递到相同的方法：

```java
Foo foo = template.execute(new RetryCallback<Foo>(){
    public Foo doWithRetry(RetryContext context){
        //business login here
    },
    new RecoveryCallback<Foo>(){
        Foo recover(RetryContext context) throws Exception{
            //recover login here
        }
    }
});

```

- **RetryOperations**：定义实现的基本的重试操作。

```java
package org.springframework.retry;
import org.springframework.retry.support.DefaultRetryState;

public interface RetryOperations {

	<T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback) throws E;

	<T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback) throws E;

	<T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RetryState retryState) throws E, ExhaustedRetryException;

	<T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback, RetryState retryState)
			throws E;

}
```

- **RetryCallback**：`RecoveryCallback`是`execute()`的一个参数，它是一个接口，允许插入在失败时需要重试的业务逻辑。

    - Configure a `RetryTemplate` bean in `@Configuration` class：

        ```java
        @Configuration
        public class AppConfig{
            @Bean
            public RetryTemplate retryTemplate(){
                RetryTemplate retryTemplate = new RetryTemplate();

                FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
                fixedBackOffPolicy.setBackOffPeriod(2000L);
                retryTemplate.setBackOffPolicy(fixedBackOffPolicy);

                SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
                retryPolicy.setMaxAttempts(2);
                retryTemplate.setRetryPolicy(retryPolicy);
            }
        }
        ```

        `RetryPolicy`决定了重试操作的时间。`SimpleRetryPolicy`用来固定次数的重试。

        `BackOffPolicy`用于控制重试之间的间隔。`FixedBackOffPolicy`会在继续之前暂停一段固定的时间。

    - Using the `RetryTemplate`

        要运行带有重试处理的代码，我们调用retryTemplate.execute():

        ```java
        retryTemplate.execute(new RetryCallback<Void, RuntimeException>(){
            @Overrid
            public void doWithRetry(RetryContext arg0){
                myService.templateRetryService();
                ...
            }
        });

        //或使用lambda表达式
        retryTemplate.execute(arg0 -> {
            myService.templateRetryService();
            return null;
        });
        
        ```

- **BackoffPolicy**：在暂时失败后重试时，在重试之前等待一段时间通常会有所帮助，因为失败通常是由一些问题引起的，而这些问题只能通过等待来解决。如果RetryCallback失败，RetryTemplate可以根据适当的BackoffPolicy暂停执行。

```java
public interface BackoffPolicy{

    BackOffContext start(RetryContext context);

    void backOff(BackOffContext backOffContext) throws BackOffInterruptedException;
}
```

backOff策略可以自由地以其选择的任何方式实现该backoff。一个常用的用例是，以指数增长的等待时间进行回退，以避免两次重试进入相同的锁定步骤，但都失败了。为此，`Spring Batch`提供`ExponentialBackoffPolicy`策略。

## 使用案例

`@EnableRetry`开启重试：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.retry.annotation.EnableRetry;

@SpringBootApplication
@EnableRetry
public class SpringRetryDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringRetryDemoApplication.class, args);
	}

}
```

`@Retryable`用于需要重试的业务，`@Recover`用于重试操作失败的方法：

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Recover;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;

import java.time.LocalTime;

@Service
@Slf4j
public class RetryTestService {

    private int retryTimes = 0;

    @Retryable(value = Exception.class, maxAttempts = 3, backoff = @Backoff(delay = 2000L, multiplier = 1.5))
    public void retry(int num) throws Exception{
        retryTimes = retryTimes + 1;
        log.info("开始第{}次", retryTimes);
        if(num <= 0){
            throw new IllegalArgumentException("num值有误");
        }
        log.info("结束：{}", LocalTime.now());
    }

    @Recover
    public void recover(Exception e){
        log.warn("重试操作失败,共执行{}次", retryTimes);
    }
}
```

Test:

```java
import com.kris.springretrydemo.service.RetryTestService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringRetryDemoApplicationTests {

	@Autowired
	private RetryTestService retryTestService;

	@Test
	public void retryTest() throws Exception{
		retryTestService.retry(-1);
	}
}
```

输出结果为：

```
开始第1次
开始第2次
开始第3次
重试操作失败,共执行3次
```


## 相关链接

[spring-retry官方项目github](https://github.com/spring-projects/spring-retry)
[官方文档](https://docs.spring.io/spring-batch/trunk/reference/html/retry.html)