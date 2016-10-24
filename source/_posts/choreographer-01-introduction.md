---
title: Choreographer - 01 - 简介
date: 2016-10-11 17:59:25
tags: android
---

我们都知道invalidate、requestLayout、requestFocus这些方法可以是View重新布局、绘制，但当我们去看源码的时候，发现这些方法就是设置了些标志位，这些方法是如何触发我们熟悉的View的绘制流程的呢？

在使用属性动画我们最多就是配置一个与布局或者绘制有关的属性的插值器，设置一下持续时间就可以来执行动画了，那么动画又是由谁驱动起来的呢？

我们知道通常我们手机画面的刷新频率是60HZ，那么这个频率是谁控制的？

在日常开发中如，如果在主线程做了写稍微耗时的操作，就会看到log输出："Skipped xxx frames! The application may be doing too much work on its main thread."，这些跳帧情况是怎么发生的呢？

上述问题都可以在Choreographer中找到答案。

# 工作流程
Choreographer的工作流程大概可以分为以下三步：

1. 接收回调
2. 安排回调定时执行
3. 回调执行

之后将详细介绍一下这三步。

# 基于Looper线程

``` java
private static final ThreadLocal<Choreographer> sThreadInstance =
        new ThreadLocal<Choreographer>() {
    @Override
    protected Choreographer initialValue() {
        Looper looper = Looper.myLooper();
        if (looper == null) {
            throw new IllegalStateException("The current thread must have a looper!");
        }
        return new Choreographer(looper);
    }
};
```

Choreography只能存在在Looper线程中，而且每个线程只有一个Choreography对象。

# 回调与回调队列
Choreography可以接收Runnable和FrameCallback两种类型的回调，并将回调封装成CallbackRecord对象像保存在CallbackQueue中。

CallbackRecord与CallbackQueue的实际与实现与Android消息机制的Message与MessageQueue非常类似。

``` java
private static final class CallbackRecord {
    public CallbackRecord next;
    public long dueTime;
    public Object action; // Runnable or FrameCallback
    public Object token;

    public void run(long frameTimeNanos) {
        if (token == FRAME_CALLBACK_TOKEN) {
            ((FrameCallback)action).doFrame(frameTimeNanos);
        } else {
            ((Runnable)action).run();
        }
    }
}
```

CallbackRecord以链表结构按照执行时间有序保存在CallbackQueue中，CallbackQueue持有链表头的引用，负责CallbackRecord的添加与提取。

## 三种回调
Android一共有三种类型的回调分别是：CALLBACK_INPUT、CALLBACK_ANIMATION、CALLBACK_TRAVERSAL，分别对应输入、动画、遍历相关的回调。Choreography为每种类型的回调都实例化了一个CallbackQueue来保存特定类型的回调。

在每一帧执行的时候严格按照CALLBACK_INPUT、CALLBACK_ANIMATION、CALLBACK_TRAVERSAL的顺序来执行回调。
