---
title: Choreographer - 02 - 接收回调
date: 2016-10-12 15:38:57
tags: android
---

# 

# 三种回调
Choreographer一共执行三种类型的回调，分别是：

``` java
/**
 * Callback type: Input callback.  Runs first.
 * @hide
 */
public static final int CALLBACK_INPUT = 0;

/**
 * Callback type: Animation callback.  Runs before traversals.
 * @hide
 */
public static final int CALLBACK_ANIMATION = 1;

/**
 * Callback type: Traversal callback.  Handles layout and draw.  Runs last
 * after all other asynchronous messages have been handled.
 * @hide
 */
public static final int CALLBACK_TRAVERSAL = 2;
```

