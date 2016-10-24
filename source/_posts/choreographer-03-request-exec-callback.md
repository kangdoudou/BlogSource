---
title: Choreographer - 03 - 请求执行回调
date: 2016-10-12 15:38:57
tags: android
---
既然手机有固定的频率，那么屏幕内容的刷新肯定不是跟随我们随意调用invalidate、requestLayout、requestFocus这些方法走的。
Choreographer内的scheduleFrameLocked方法是用来请求一次回调执行，其内部有两种实现：使用垂直同步，不用垂直同步

# 垂直同步
屏幕的刷新有其一定的同步机制。这里使用的是Vertical Sync - 垂直同步（下记V-Sync）

V-Sync是加在两帧之间。它指示着前一帧的结束，和新一帧的开始。这也就意味着每秒内最多有60个V-Sync，每一次V-Sync到来的时候，如果需要重新绘制，就执行绘制流程刷新屏幕。

V-Sync由底层SurfaceFlinger产生，但其不自主均匀的每秒钟产生60个，是在需要刷新的时候去请求SurfaceFlinger产生V-Sync。负责与SurfaceFlinger交互的是DisplayEventReceiver，其中涉及的主要方法是：

``` java
/**
 * Schedules a single vertical sync pulse to be delivered when the next
 * display frame begins.
 */
public void scheduleVsync() {
    if (mReceiverPtr == 0) {
        Log.w(TAG, "Attempted to schedule a vertical sync pulse but the display event "
                + "receiver has already been disposed.");
    } else {
        nativeScheduleVsync(mReceiverPtr);
    }
}

// Called from native code.
@SuppressWarnings("unused")
private void dispatchVsync(long timestampNanos, int builtInDisplayId, int frame) {
    onVsync(timestampNanos, builtInDisplayId, frame);
}

/**
 * Called when a vertical sync pulse is received.
 * The recipient should render a frame and then call {@link #scheduleVsync}
 * to schedule the next vertical sync pulse.
 *
 * @param timestampNanos The timestamp of the pulse, in the {@link System#nanoTime()}
 * timebase.
 * @param builtInDisplayId The surface flinger built-in display id such as
 * {@link SurfaceControl#BUILT_IN_DISPLAY_ID_MAIN}.
 * @param frame The frame number.  Increases by one for each vertical sync interval.
 */
public void onVsync(long timestampNanos, int builtInDisplayId, int frame) {
}
```

1. 通过scheduleVsync请求V-Sync
2. SurfaceFlinger收到V-Sync请求，在下一次V-Sync的时间点调用dispatchVsync方法
3. dispatchVsync紧接着调用onSync方法，onSync内实现具体逻辑

需要注意的是第1步和第2步并不是连续的，收到V-Sync请求之后需要等到下一次V-Sync开始的时间才去回调dispatchVsync方法，而且在同一个V-Sync时间内无论请求几次V-Sync，都只会在下一次V-Sync开始的时候收到一次dispatchVsync回调。

![v-sync](/2016/10/12/choreographer-03-request-exec-callback/v-sync.jpg)