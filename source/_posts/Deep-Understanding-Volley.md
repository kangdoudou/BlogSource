---
title: 深入理解Volley
date: 2016-09-29 20:49:53
tags: [android, volley]
---

学习完了官方教程之后我们就学会如何在项目中使用Volley了，享受其带给我们的便捷。但作为一个有追求的程序猿我们决不能停留在使用的阶段，我们要去探索其中的奥秘，去学习优秀框架的设计思想，具体实现方法，甚至是学习作者的优秀编码习惯。
如何深入学习Volley呢？官方教程中声称Volley有八大优点，我们来探索一下这八大优点究竟是什么，在代码中是如何实现的。

# 自动调度网络请求
通过教程了解到，当需要执行一个请求的时候，只需要将其添加到`RequestQueue`中，然后就是等待响应了。
那么中间Volley都做了那些事情呢，官方给的Request一生这张图做了详细介绍。
![Request的一生](/2016/09/27/sending-a-simple-request/volley-request.png "Request的一生")

自动调度要分两个部分：自动 和 调度

## 自动
Volley内部有一个CacheDispatcher以及多个NetworkDispatcher，它们继承自Thread类，它们的业务实现是从各自的请求队列里不断拿Request并执行。所以我们只需要将我们的请求添加到RequestQueue中，这两类Dispatcher线程就会自动的去执行请求。

在RequestQueue的start()方法里创建与开启Dispatcher线程：

``` java
public void start() {
    stop();  // Make sure any currently running dispatchers are stopped.
    // Create the cache dispatcher and start it.
    mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
    mCacheDispatcher.start();

    // Create network dispatchers (and corresponding threads) up to the pool size.
    for (int i = 0; i < mDispatchers.length; i++) {
        NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
                mCache, mDelivery);
        mDispatchers[i] = networkDispatcher;
        networkDispatcher.start();
    }
}
```
一个RequestQueue只有一个CacheDispatched，与多个NetworkDispatcher，NetworkDispatcher数量可自定义，默认为4个。
因为一个RequestQueue中包含了多个thread, 这也是推荐使用单例的原因之一，过多的RequestQueue就会导致thread过多，因为thread调度问题导致性能下降。

## 调度
调度就是RequestQueue中同时有多个请求时，先执行哪个后执行哪个。
Volley将请求分为四个优先级：LOW、NORMAL、HIGH、IMMEDIATE，优先度以此增加，默认为NORMAL。
优先级高的先执行，同一优先级按照FIFO的顺序执行。
这种调度顺序在添加到RequestQueue的时候就被安排好了。
RequestQueue.add()方法根据是否使用缓存将Request添加到缓存的或者网络请求的PriorityBlockingQueue中。
来看一下PriorityBlockingQueue.add()的实现

``` java
public boolean add(E e) {
	return offer(e);
}

public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    int n, cap;
    Object[] array;
    while ((n = size) >= (cap = (array = queue).length))
        tryGrow(array, cap);
    try {
        Comparator<? super E> cmp = comparator;
        if (cmp == null) // Volley默认没有设置comparator
            siftUpComparable(n, e, array);
        else
            siftUpUsingComparator(n, e, array, cmp);
        size = n + 1;
        notEmpty.signal();
    } finally {
        lock.unlock();
    }
    return true;
}

private static <T> void siftUpComparable(int k, T x, Object[] array) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = array[parent];
        if (key.compareTo((T) e) >= 0)
            break;
        array[k] = e;
        k = parent;
    }
    array[k] = key;
}
```

PriorityBlockingQueue.add()方法保证了首先按照优先级排序，优先级一致的情况下按照FIFO的顺序。

可以通过重写Request的getPriority()方法自定义请求优先级。注：ImageRequest被重写默认为LOW优先级。

``` java
@Override
public Priority getPriority() {
    return Priority.LOW;
}
```

# 多并发网络连接

上面[自动]()小节提到，每一个RequestQueue中包含多个NetworkDispatcher（默认是4个），每一个NetworkDispatcher都是一个负责网络请求的线程。因此Volley能够同时处理多个网络请求。

# 具有标准缓存一致性的，透明的，内存、硬盘响应缓存
所谓透明的内存硬盘响应缓存的意思是，在我们使用的时候根本就没有感觉到有缓存的存在，完全由Volley负责为我们完成。
Volley默认提供了硬盘响应缓存DiskBasedCache，当然用户可以通过实现Cache接口自定义自己的缓存，比如内存硬盘双缓存。

默认Request是使用缓存的，可以通过Request.setShouldCache(boolean)来设置是否使用缓存。

## 每个响应存储为一个文件
DiskBasedCache将每个响应生成一个文件，保存到磁盘上，在内存中保存着每一个磁盘缓存的响应头。

``` java
public synchronized void put(String key, Entry entry) {
    pruneIfNeeded(entry.data.length); // *(1)如果剩余空间不足，删除一些原来的缓存
    File file = getFileForKey(key); // *(2)根据URL获取要缓存到的文件对象
    try {
        FileOutputStream fos = new FileOutputStream(file);
        CacheHeader e = new CacheHeader(key, entry);
        e.writeHeader(fos); // 输出响应头
        fos.write(entry.data); // 输出响应体
        fos.close();
        putEntry(key, e);
        return;
    } catch (IOException e) {
    }
    boolean deleted = file.delete();
    if (!deleted) {
        VolleyLog.d("Could not clean up file %s", file.getAbsolutePath());
    }
}
```

\*(1) 删除缓存的规则只是按照FIFO规则来的，跟缓存的命中率没有关系。直到删到能存下当前响应为止。
\*(2) 通过URL这个key生成文件名的时候Volley是这么实现的：

``` java
private String getFilenameForKey(String key) {
    int firstHalfLength = key.length() / 2;
    String localFilename = String.valueOf(key.substring(0, firstHalfLength).hashCode());
    localFilename += String.valueOf(key.substring(firstHalfLength).hashCode());
    return localFilename;
}
```

将URL从中间隔断，分别计算hashcode，然后在将两个hashcode拼接起来。猜想这样做大概是为了降低hash冲突的概率吧。

响应头的输出严格按照特定的顺序：

``` java
public boolean writeHeader(OutputStream os) {
    try {
        writeInt(os, CACHE_MAGIC); // 版本号，与java序列化的serialVersionUID功能一致
        writeString(os, key); // 缓存的唯一标志，即URL
        writeString(os, etag == null ? "" : etag); // etag 缓存一致性使用
        writeLong(os, serverDate); // 服务器生成该响应的时间
        writeLong(os, ttl); // [Time to Live](https://en.wikipedia.org/wiki/Time_to_live)
        writeLong(os, softTtl);
        writeStringStringMap(responseHeaders, os); // 输出Header中的其他键值对
        os.flush();
        return true;
    } catch (IOException e) {
        VolleyLog.d("%s", e.toString());
        return false;
    }
}
```

总的来说将响应序列化到磁盘，响应内容的输出是严格按照特定的格式与顺序的，所以在读缓存的时候也要严格按照这种格式与顺序。

# 支持请求优先级
见[调度]()

# 取消请求接口
RequestQueue中有三个集合是用来保存Request的：

``` java
private final Set<Request> mCurrentRequests = new HashSet<Request>();
private final PriorityBlockingQueue<Request> mCacheQueue = new PriorityBlockingQueue<Request>();
private final PriorityBlockingQueue<Request> mNetworkQueue = new PriorityBlockingQueue<Request>();
```
通过RequestQueue.add()添加的所有请求都会被添加到mCurrentRequests Set中，mCurrentRequests是所有请求的集合，如果请求需要缓存处理就会被添加到mCacheQueue中，如果请求不需要缓存处理或者缓存处理不了就会添加到mNetworkQueue。
mCacheQueue被CacheDispatcher提取，mNetworkQueue被NetworkDispatcher提取。

Volley提供了通过Tag来取消请求的接口，方便大家合理的取消一个或者多个请求。

``` java
public void cancelAll(final Object tag) {
    if (tag == null) {
        throw new IllegalArgumentException("Cannot cancelAll with a null tag");
    }
    cancelAll(new RequestFilter() {
        @Override
        public boolean apply(Request<?> request) {
            return request.getTag() == tag;
        }
    });
}

public void cancelAll(RequestFilter filter) {
    synchronized (mCurrentRequests) {
        for (Request<?> request : mCurrentRequests) {
            if (filter.apply(request)) {
                request.cancel();
            }
        }
    }
}
```

在mCurrentRequests这个所有请求的集合中，遍历带有tag的请求，然后取消。

# 方便定制。比如说：重试和退避请求
该特性说的是Volley提供了一套默认机制，以及方便的替换自定义方法。retry 与 backoff机制便是其中之一。

Volley中使用的不管是Request还是RetryPolicy都提供了接口或者基类，我们只需要实现或者集成就可以方便的定制复合我们业务逻辑的特性。

提到了retry 与 backoff， 下面就叙述一下其实现

## Retry and Backoff

retry即当请求发生超时或者连接异常的时候，再一次执行该请求。backoff，翻译为退避方法，即在retry请求是，超时时间应如何处理。
Retry机制的接口定义为：RetryPolicy

``` java
public interface RetryPolicy {
	// 获取当前超时时间
    public int getCurrentTimeout();
    // 获取当前已retry次数
    public int getCurrentRetryCount();
    // 执行retry
    public void retry(VolleyError error) throws VolleyError;
}
```

默认实现为：DefaultRetryPolicy，其中有三个重要的属性：mCurrentTimeoutMs、mMaxNumRetries、mBackoffMultiplier，分别是当前请求超时时间、最多retry次数、backoff算法的乘数。默认值分别为2500、1、1。

DefaultRetryPolicy的retry方法实现：

``` java
@Override
public void retry(VolleyError error) throws VolleyError {
    mCurrentRetryCount++;
    mCurrentTimeoutMs += (mCurrentTimeoutMs * mBackoffMultiplier);
    if (!hasAttemptRemaining()) {
        throw error;
    }
}

protected boolean hasAttemptRemaining() {
    return mCurrentRetryCount <= mMaxNumRetries;
}
```
在每次执行retry的时候，超时时间都会变化，具体方法为：mCurrentTimeoutMs += (mCurrentTimeoutMs * mBackoffMultiplier);
按照默认值首次执行请求超时时间为：2500。2500时间内没有完成请求的话，将发生一次retry，超时时长就被重置为：5000。如果这个5000毫秒内也有收到响应的话，因为最多retry次数就1次，就不会再执行这个请求了，这个请求就失败了。

retry请求是连续执行，并不是再次把请求放入请求队列中。
具体位于BasicNetwork的performRequest中：

```
public NetworkResponse performRequest(Request<?> request) throws VolleyError {
	...
	while(true){
		try{
			httpResponse = mHttpStack.performRequest(request, headers);
		}catch(TimeoutException e){
			attemptRetryOnException(request, new TimeoutError()); // 执行retry方法，判断retry次数是否达到最大值，重新计算timeout时间
		}
		...
	}
}
```

