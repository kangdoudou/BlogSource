---
title: Choreographer - 03 - 请求执行回调
date: 2016-10-17 15:38:57
tags: android
---

请求回调执行，请求-对应的方法是Choreographer.scheduleFrameLocked, 执行-对应的方法Choreographer.doFrame

通过上一节我们知道，无论是消费INPUT、动画开始还是View的invalidate、requestLayout、requestFocus这些方法都会最终调用scheduleFrameLocked方法请求执行回调刷新屏幕。

既然手机有固定的频率，那么屏幕内容的刷新肯定不是跟随我们随意调用scheduleFrameLocked执行的，scheduleFrameLocked方法具体分成两种实现：使用以及不使用垂直同步

变量USE_VSYNC标记是否要使用垂直同步，这个值是从系统配置中读取的。

``` java
private static final boolean USE_VSYNC = SystemProperties.getBoolean(
        "debug.choreographer.vsync", true);
```

scheduleFrameLocked代码：

``` java
private void scheduleFrameLocked(long now) {
    if (!mFrameScheduled) {
        mFrameScheduled = true; // 在doFrame执行后会置为false
        						// 因此多次调用只有第一次会真的请求执行doFrame
        if (USE_VSYNC) {	//使用垂直同步
            if (DEBUG) {
                Log.d(TAG, "Scheduling next frame on vsync.");
            }

            // If running on the Looper thread, then schedule the vsync immediately,
            // otherwise post a message to schedule the vsync from the UI thread
            // as soon as possible.
            if (isRunningOnLooperThreadLocked()) {
                scheduleVsyncLocked();
            } else {
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_VSYNC);
                msg.setAsynchronous(true);
                mHandler.sendMessageAtFrontOfQueue(msg);
            }
        } else { // 不使用垂直同步
            final long nextFrameTime = Math.max(
                    mLastFrameTimeNanos / NANOS_PER_MS + sFrameDelay, now);
            if (DEBUG) {
                Log.d(TAG, "Scheduling next frame in " + (nextFrameTime - now) + " ms.");
            }
            Message msg = mHandler.obtainMessage(MSG_DO_FRAME);
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, nextFrameTime);
        }
    }
}
```

# 不使用垂直同步
在不使用垂直同步的情况下，两帧之间有最小事件间隔`sFrameDelay`， 默认为10ms。因此，理论上在不使用垂直同步的情况下最大的屏幕最大的刷新频率为100HZ。

首先是根据上一帧执行的时间结合sFrameDelay与系统当前时间对比计算出下一帧执行时间。
然后使用消息机制，发送定时消息到主线程来执行doFrame。


# 使用垂直同步

垂直同步(下记V-Sync)是加在两帧之间。它指示着前一帧的结束，和新一帧的开始。这也就意味着每秒内最多有60个V-Sync，每一次V-Sync到来的时候，如果需要重新绘制，就执行绘制流程刷新屏幕。

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

需要注意的是第1步和第2步并不是连续的，收到V-Sync请求之后需要等到下一次V-Sync开始的时间dispatchVsync方法才会被调用，而且在同一个V-Sync时间内无论请求几次V-Sync，都只会在下一次V-Sync开始的时候收到一次dispatchVsync回调。

![v-sync](/2016/10/17/choreographer-03-request-exec-callback/v-sync.jpg)

DisplayEventReceiver的具体实现类是FrameDisplayEventReceiver，其内主要实现是：

``` java
@Override
public void onVsync(long timestampNanos, int builtInDisplayId, int frame) {
	...
    long now = System.nanoTime();
    if (timestampNanos > now) {
        Log.w(TAG, "Frame time is " + ((timestampNanos - now) * 0.000001f)
                + " ms in the future!  Check that graphics HAL is generating vsync "
                + "timestamps using the correct timebase.");
        timestampNanos = now;
    }

    if (mHavePendingVsync) {
        Log.w(TAG, "Already have a pending vsync event.  There should only be "
                + "one at a time.");
    } else {
        mHavePendingVsync = true;
    }

    // @@@ 这里是FW垂直同步脉冲生成器的调用
    // @@@ 不是Loop线程
    // @@@ 抛到Loop线程执行doFrame
    mTimestampNanos = timestampNanos;
    mFrame = frame;
    Message msg = Message.obtain(mHandler, this);
    msg.setAsynchronous(true);
    mHandler.sendMessageAtTime(msg, timestampNanos / NANOS_PER_MS);
}

@Override
public void run() {
    mHavePendingVsync = false;
    doFrame(mTimestampNanos, mFrame); // doFrame执行
}
```
