---
title: springboot使用SpringRetry
date: 2018-08-07
tags: springboot
categories: 编程
---
在分布式系统中，为了保证数据分布式事务的强一致性，大家在调用RPC接口或者发送MQ时，针对可能会出现网络抖动请求超时情况采取一下重试操作。大家用的最多的重试方式就是MQ了，但是如果你的项目中没有引入MQ，那就不方便了，本文主要介绍一下如何使用Spring Retry实现重试操作

## 1、引入Spring Retry依赖
``` java
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
    <version>1.1.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.9</version>
</dependency>
```
## 2、在启动入口加入重试配置

``` java
@SpringBootApplication
@EnableRetry
public class RetryApplication {
	public static void main(String[] args) throws Exception{
		SpringApplication.run(RetryApplication.class, args);
	}
}
```
## 3、编写测试service
``` java
@Service
public class RetryService {
    @Retryable(value= {RemoteAccessException.class},maxAttempts = 5,backoff = @Backoff(delay = 5000l,multiplier = 1))
    public void retryTest(Integer id) throws Exception {
        System.out.println("do something...");
        throw new RemoteAccessException("RemoteAccessException....");
    }
    @Recover
    public void recover(RemoteAccessException e,Integer id) {
        System.out.println(e.getMessage());
        System.out.println("recover....");
    }
}
```
其中，RemoteAccessException可以是自定义异常，

@Retryable 标注此注解的方法在发生异常时会进行重试
  * value：指定处理的异常类
  * maxAttempts：最大重试次数。默认3次
  * backoff： 重试等待策略。默认使用@Backoff注解

@Backoff 重试等待策略
  * delay 重试等待的时间
  * multiplier 即指定延迟倍数，比如delay=5000l,multiplier=2,则第一次重试为5秒，第二次为10秒，第三次为20秒……

@Recover 用于@Retryable重试失败后处理方法，此注解的方法参数和返回值一定要和@Retryable的一致，见上面代码。



