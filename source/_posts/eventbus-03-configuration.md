---
title: EventBus官方教程翻译（三）：配置Configuration
date: 2017-01-05 22:59:28
tags: [android, eventbus]
---

EventBusBuilder这个类负责来配置EventBus的方方面面。例如，下面是如何配置当事件没有接受者的时候不输出任何日志：

``` java
EventBus eventBus = EventBus.builder()
	.logNoSubscriberMessages(false)
	.sendNoSubscriberEvent(false)
	.build();
```
接下来这个例子，当订阅者方法抛出异常时，如何处理：

``` java
EventBus eventBus = EventBus.builder().throwSubscriberException(true).build();
```
> 说明：默认配置，EventBus会捕捉订阅者方法抛出的所有异常，然后发送出一个`SubscriberExceptionEvent`事件，这个事件不一定非要被处理。

查看 [EventBusBuilder类及其说明文档](http://greenrobot.org/files/eventbus/javadoc/3.0/org/greenrobot/eventbus/EventBusBuilder.html)，了解其所有配置

# 配置EventBus默认实例
在你App的任何地方，你都可以通过`EventBus.getdefault()`这种简便的方法获取一个共享的EventBus实例。通过方法`installDefaulteventBus()`同样可以对这个单例进行配置。

例如，通过配置，我们可以将捕捉到的观察者方法抛出的异常重新抛出。但是建议，只是在DEBUG版本中使用这样的配置，因为这很容易造成crash。

``` java
EventBus.builder().throwSubscriberException(BuildConfig.DEBUG).installDefaultEventBus();
```

> 说明：这个操作只能被执行一次，且在第一次使用默认EventBus单例之前。后续调用`installDefaultEventBus()`会导致异常抛出。这种机制保证了App的一致性。一般都是在Application这个类中来配置默认EventBus单例。