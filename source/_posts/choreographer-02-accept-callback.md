---
title: Choreographer - 02 - 接收回调
date: 2016-10-12 15:38:57
tags: android
---

# 三种回调的添加
## INPUT
![Input 回调时序图](/2016/10/12/choreographer-02-accept-callback/Callback_Input.png)

scheduleConsumeBatchedInput 方法：

``` java
void scheduleConsumeBatchedInput() {
    if (!mConsumeBatchedInputScheduled) {
        mConsumeBatchedInputScheduled = true;
        mChoreographer.postCallback(Choreographer.CALLBACK_INPUT,
                mConsumedBatchedInputRunnable, null);
    }
}
```

## ANIMATION
![Animation 回调时序图](/2016/10/12/choreographer-02-accept-callback/Callback_Animator.png)

scheduleAnimation 方法：

``` java
private void scheduleAnimation() {
    if (!mAnimationScheduled) {
        mChoreographer.postCallback(Choreographer.CALLBACK_ANIMATION, this, null);
        mAnimationScheduled = true;
    }
}
```

## TRAVERSAL
![Traversal 回调时序图](/2016/10/12/choreographer-02-accept-callback/Callback_Traversal.png)

scheduleTraversals 方法：

``` java
void scheduleTraversals() {
    if (!mTraversalScheduled) {
        mTraversalScheduled = true;
        mTraversalBarrier = mHandler.getLooper().postSyncBarrier();
        mChoreographer.postCallback(
                Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        scheduleConsumeBatchedInput();
    }
}
```


# 添加回调的方法postCallback 与 postFrameCallback

## Choreographer.postCallback

``` java
public void postCallback(int callbackType, Runnable action, Object token) {
    postCallbackDelayed(callbackType, action, token, 0);
}
```

交给postCallbackDelayed处理， delay时间为0

``` java
public void postCallbackDelayed(int callbackType, Runnable action, Object token, long delayMillis) {
    if (action == null) {
        throw new IllegalArgumentException("action must not be null");
    }
    if (callbackType < 0 || callbackType > CALLBACK_LAST) {
        throw new IllegalArgumentException("callbackType is invalid");
    }

    postCallbackDelayedInternal(callbackType, action, token, delayMillis);
}
```

进行安全检查，然后交给postCallbackDelayedInternal内部方法处理

``` java
private void postCallbackDelayedInternal(int callbackType, Object action, Object token, long delayMillis) {
    if (DEBUG) {
        Log.d(TAG, "PostCallback: type=" + callbackType
                + ", action=" + action + ", token=" + token
                + ", delayMillis=" + delayMillis);
    }

    synchronized (mLock) { // 同步锁
        final long now = SystemClock.uptimeMillis();
        final long dueTime = now + delayMillis; // 计算该回调应该执行的时间
        mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token); // 添加到相应的CallbackQueue中

        if (dueTime <= now) { // 现在就到了执行时间，安排执行
            scheduleFrameLocked(now);
        } else { // 没有到执行时间， 使用消息机制推迟安排执行
            Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
            msg.arg1 = callbackType;
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, dueTime);
        }
    }
}
```

## Choreographer.postFrameCallback

``` java
public interface FrameCallback {
    public void doFrame(long frameTimeNanos);
}
```

FrameCallback接口与Runnable的不同是其能够接收时间参数frameTimeNanos。

具体调用顺序为：

``` java
public void postFrameCallback(FrameCallback callback) {
    postFrameCallbackDelayed(callback, 0);
}
```

交给postFrameCallbackDelayed方法，delay时间为0

``` java
public void postFrameCallbackDelayed(FrameCallback callback, long delayMillis) {
    if (callback == null) {
        throw new IllegalArgumentException("callback must not be null");
    }

    postCallbackDelayedInternal(CALLBACK_ANIMATION, callback, FRAME_CALLBACK_TOKEN, delayMillis);
}
```

指定Callback的类型为ANIMATION，添加到CALLBACK_ANIMATION的CallbackQueue中。
FrameCallback会在渲染每一帧的时候调用，通过向Choreographer中添加FrameCallback，我们可以与Choreographer的渲染过程进行交互。


# 添加到队列 并 请求回调执行
不管是通过postCallback方法还是通过postFrameCallback方法添加回调，最终都是通过postCallbackDelayedInternal向回调队列里添加一个回调，并请求回调的执行

``` java
private void postCallbackDelayedInternal(int callbackType,
        Object action, Object token, long delayMillis) {
    if (DEBUG) {
        Log.d(TAG, "PostCallback: type=" + callbackType
                + ", action=" + action + ", token=" + token
                + ", delayMillis=" + delayMillis);
    }

    synchronized (mLock) {
        final long now = SystemClock.uptimeMillis();
        final long dueTime = now + delayMillis;
        mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token); // 添加到回调队列

        if (dueTime <= now) { // 执行时间到， 马上安排一次回调执行，即渲染一帧
            scheduleFrameLocked(now);
        } else { // 执行时间未到，使用消息机制定时执行回调
            Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
            msg.arg1 = callbackType;
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, dueTime);
        }
    }
}
```
关于scheduleFrameLocked方法是怎样出发回调执行，以及回调具体是怎样执行的在之后详细介绍。

## addCallbackLocked实现
``` java
public void addCallbackLocked(long dueTime, Object action, Object token) {
    CallbackRecord callback = obtainCallbackLocked(dueTime, action, token); //生成callback对象
    CallbackRecord entry = mHead;
    if (entry == null) {
        mHead = callback;
        return;
    }
    if (dueTime < entry.dueTime) {
        callback.next = entry;
        mHead = callback;
        return;
    }
    while (entry.next != null) { // 根据执行时间添加到合适的位置
        if (dueTime < entry.next.dueTime) {
            callback.next = entry.next;
            break;
        }
        entry = entry.next;
    }
    entry.next = callback;
}
```