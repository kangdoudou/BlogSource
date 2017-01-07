---
title: EventBus官方教程翻译（二）：交付线程（线程模式ThreadMode）
date: 2016-12-13 22:13:25
tags: [android, eventbus]
---

> [原文连接](http://greenrobot.org/eventbus/documentation/delivery-threads-threadmode/)

EventBus能够帮助我们处理线程相关的事情：事件可以在同于发送线程中接受。最常用的情况就是UI改变。在Android中UI的改变必须在UI线程（主线程）中进行。另一方面，比如网络请求或者其他耗时任务不能够在主线程中执行。EventBus能够帮助我们处理这些耗时的任务，并将处理结果与同步给UI线程（免得去研究线程的切换，比如AsyncTask等）。

在EventBus中，一共有四种线程模式，用来定义事件处理方法应该在那个（类）线程中被调用执行。

# 线程模式：POSTING

订阅者会在事件发布的线程中被调用。这也是默认的线程模式。事件的穿件是被同步执行的，也就是说事件发布之后，马上就会挨个的调用订阅者。这种线程模式的开销最小，因为完全没有线程切换。所以对于短时间可以完成的且不需要主线程的小任务推荐使用这种县城模式。事件处理者必须尽快return，以免阻塞发布事件的线程，因为发布事件的线程也可能是UI线程。Example：

``` java
// Called in the same thread(default)
// ThreadMode is optionnal here
@Subscribe(threadMode = ThreadMode.POSTING)
public void onMessage(MessageEvent event) {
	log(event.message);
}
```

# 线程模式：MAIN
订阅者将会被在主线程（
UI线程）中被调用。如果发布事件的线程就是主线程的话，那就免了线程切换，事件处理者将会被直接调用（同步调用，这种情况与使用ThreadMode.POSTING相同）。使用这种模式的事件处理者需尽快return，以免阻塞主线程。Example:

``` java
// Called in Android UI's main thread
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessage(MessageEvent event) {
	textField.setText(event.message);
}
```

# 线程模式：BACKGROUND
订阅者将会被在后台线程中被调用。如果事件发布的线程不是主线程，事件处理者将会被在事件发布线程中直接同步调用。如果发布事件的线程是主线程，EventBus会创建一个后台线程，依次执行所有的该类型事件。事件处理者必须尽量尽快return，以免阻塞该后台线程。

``` java
// Called in the background thread
@Subscribe(threadMode = ThreadMode.BACKGROUND)
public void onMessage(MessageEvent event) {
	saveToDisk(event.message);
}
```
# 线程模式：ASYNC
事件处理者会被在单独的线程中调用。与事件发布线程和主线程独立开来。使用这种模式，发布事件不需要等待事件处理方法执行完毕。如果事件处理者所做的任务比较耗时的话，比如网络访问，需要使用这种模式。因为同时并发执行的线程数量有限（与cpu硬件相关），所以我们要避免同时触发大量独立线程来异步执行这些事件。EventBus使用线程池来高效服用这些线程。

``` java
// Called in a separate thread
@Subscribe(threadMode = ThreadMode.ASYNC)
public void onMessage(MessageEvent event) {
	backend.send(event.message);
}
```
