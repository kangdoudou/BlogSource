---
title: Choreographer - 04 - 执行回调
date: 2016-10-25 17:41:56
tags: android
---

回调的执行的方法是doFrame。而且上一节我们知道，有两种方式请求执行回调。一种是不使用垂直同步的、另外一种使用垂直同步。
两种方法最后都将doFrame的执行抛到了Loop线程（一般就是主线程）。

# 理想情况

在理想的情况下，我们期望Choreographer执行回调，在收到dispatchVSync的时候立即执行doFrame，是这样的：

![理想情况](/2016/10/25/choreographer-04-exec-callback/1.jpg)

> 注：使用VSync。sv表示scheduleVSync， dv表示dispatchVSync， df表示doFrame

# 半理想情况
然而因为运行在Loop线程，所以doFrame对应的消息必须得等前面消息执行完了才能够执行，既然这样那我们就期望如下吧：

![半理想情况](/2016/10/25/choreographer-04-exec-callback/2.jpg)

> 注：橙色块表示其他消息。

# 跳帧
但是呢，如果doFrame之前的消息执行了很长时间，导致等到执行doFrame这个消息的时候时间已经超过了其应该执行时间很久，如下图：

![跳帧](/2016/10/25/choreographer-04-exec-callback/3.jpg)

这就出现了我们所说的卡帧跳帧的情况。上图中只跳了一帧。如果doFrame之前的消息执行了特别长的时间，导致跳了超过SKIPPED_FRAME_WARNING_LIMIT（系统配置，默认30帧）帧，我们就会收到如下log："Skipped xxx frames! The application may be doing too much work on its main thread."。

# 连帧
我们每一帧doFrame渲染的成果到下一帧到来时才会呈现在屏幕上，那么如果现两个doFrame消息在同一帧内完成怎么办，如下图

![连帧](/2016/10/25/choreographer-04-exec-callback/4.jpg)

如果发生这种连帧的情况，Choreographer会把后面那次渲染推迟到下一帧执行如上图下方。

还有另外一种情况也会导致跳帧，就是回调本身的执行时间过久，导致下一帧不能及时执行，情况都差不多，就不细说了。

# doFrame源码
来看一下doFrame的具体代码：

``` java
void doFrame(long frameTimeNanos, int frame) {
    final long startNanos;
    synchronized (mLock) {
        if (!mFrameScheduled) { // @@@ 确认一下是否有帧被安排了
            return; // no work to do
        }

        startNanos = System.nanoTime();
        final long jitterNanos = startNanos - frameTimeNanos;

        // @@@ 超时时长大于一个周期
        // @@@ 警告Log跳帧
        // @@@ 修正frameTimeNaos，使超时在一个周期以内
        if (jitterNanos >= mFrameIntervalNanos) {
            final long skippedFrames = jitterNanos / mFrameIntervalNanos;
            if (skippedFrames >= SKIPPED_FRAME_WARNING_LIMIT) {
                Log.i(TAG, "Skipped " + skippedFrames + " frames!  "
                        + "The application may be doing too much work on its main thread.");
            }
            final long lastFrameOffset = jitterNanos % mFrameIntervalNanos;
            frameTimeNanos = startNanos - lastFrameOffset;
        }

        // @@@ 由于跳帧导致上一帧的完成事件大于这一帧安排的时间
        // @@@ 这一帧现在就不能执行了，这次时间脉冲就没用了
        // @@@ 所以需要多安排一次时间脉冲
        if (frameTimeNanos < mLastFrameTimeNanos) {
            scheduleVsyncLocked();
            return;
        }

        mFrameScheduled = false;
        mLastFrameTimeNanos = frameTimeNanos;
    }
    // @@@ 以此执行三种类型的回调
    doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);
    doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);
    doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);
}

```

doFrame方法首先进行了跳帧确认修正时间，其次又确认了一下是否需要重新安排执行改回调，最后才是真正执行三种回调。

doCallbacks的实现也特简单，CallbackRecord在CallbackQueue中是以链表形式存储的，根据回调类型与当前时间，把到执行时间CallbackRecord链提取出来，挨个执行。代码如下：

``` java
void doCallbacks(int callbackType, long frameTimeNanos) {
    CallbackRecord callbacks;
    synchronized (mLock) {
        // We use "now" to determine when callbacks become due because it's possible
        // for earlier processing phases in a frame to post callbacks that should run
        // in a following phase, such as an input event that causes an animation to start.
        final long now = SystemClock.uptimeMillis();
        // @@@ 根据当前事件提取CallbackRecord链
        callbacks = mCallbackQueues[callbackType].extractDueCallbacksLocked(now); 
        if (callbacks == null) {
            return;
        }
        mCallbacksRunning = true;
    }
    try {
    	// @@@ 循环执行CallbackRecord
        for (CallbackRecord c = callbacks; c != null; c = c.next) {
            c.run(frameTimeNanos);
        }
    } finally {
        synchronized (mLock) {
            mCallbacksRunning = false;
            // @@@ 回收CallbackRecord
            do {
                final CallbackRecord next = callbacks.next;
                recycleCallbackLocked(callbacks);
                callbacks = next;
            } while (callbacks != null);
        }
    }
}
```