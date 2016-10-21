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

# 的
Choreographer的英文翻译是`编舞者`，

# 工作流程
Choreographer的工作流程大概可以分为以下三步：

1. 接收回调
2. 安排回调定时执行
3. 回调执行

下面将详细介绍一下这三步。