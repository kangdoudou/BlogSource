---
title: EventBus官方教程翻译（一）：三步开启EventBus
date: 2016-12-13 20:18:23
tags: [android, eventbus]
---
EventBus的api特别简单，只需要三步。

在开始之前，请确认你的项目已经添加了EventBus库依赖。

# 第一步：定义事件

事件是没有任何特殊限制的POJO（plain old Java object）对象

``` java
public class MessageEvent {
	public final String message;
	public MessageEvent(String message) {
		this.message = message;
	}
}
```

# 第二部：准备订阅者

订阅者现实了时间处理方法（也被称为”订阅者方法“），当订阅的事件产生时，这个方法就会被调用。使用@Subscribe注解来声明订阅者方法。
特别说明：EventBus 3的订阅者方法可以随意取（没有EventBus 2d的命名约束）

``` java
// This method will be called when a MessageEvent is posted (in the UI thread for Toast)
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessageEvent(MessageEvent event) {
	Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
}

// This method will be called when a SomeOtherEvent is posted
@Subscribe
public void handleSomethingElse(SomeOtherEvent event) {
	doSomethingWith(event);
}
```

订阅者需要向EventBus注册以及取消注册自己。只有被注册了，订阅者才能收到事件。通常在Android中，一般都是在activities和fragments的适当的生命周期中注册。

``` java
@Override
public void onStart() {
	super.onStart();
	EventBus.getDefault().register(this);
}

@Overide
public void onStop() {
	EventBus.getDefault().unregister(this);
	super.onStop();
}
```

# 第三步：发布事件

在你代码的任何地方你都可以发布事件。所有匹配该事件的订阅者都会收到这事件。

``` java
EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
```