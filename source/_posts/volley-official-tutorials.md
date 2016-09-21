---
title: Volley官方教程翻译
date: 2016-09-20 15:18:16
tags: android volley
---

一直以来我都认为官方教程和源码是快速学习技术与深入理解的最佳途径。
所以我们先粗略翻译一下官方教程，然后在通过源码分析一下具体的实习细节。

# 使用Volley来完成网络数据请求

> 原文连接：https://developer.android.com/training/volley/index.html

Volley是一个HTTP库，它使得Android应用程序的网络访问变得更加方便、快速。你可以通过[AOSP](https://android.googlesource.com/platform/frameworks/volley)(Android Open Source Project)获取其源码。
Volley包含在Android源码之中，具体路径为：android源码/frameworks/volley

Volley有以下优点：
- 自动调度网络请求
- 多并发网络连接
- 具有标准缓存一致性的，透明的，内存、硬盘响应缓存
- 支持请求优先级
- 取消请求接口。你可以单独取消一个请求，你也可以通过划定范围取消多个请求。
- 方便定制。比如说：重试和退避请求。
- 强大的排序功能，使得你能够很容易的使用从网络异步获取的数据填充你的UI。
- 调试与追踪工具。

Volley尤其擅长用来填充UI的RPC调用操作，比如获取一组结构化的搜索结果。它很容易与其他协议集成，而且已经实现了常用的、原始的String，Image与Json协议请求。Volley提供了许多在日常开发中所需要的网络层的功能，避免我们去重复造轮子，专注于业务逻辑代码的实现。

Volley不是用于大文件下载或者数据流的操作，因为在处理响应的时候，Volley把整个响应都保存在内存中。如果要操作大文件，请考虑使用DownloadManager。

Volley库源码位于AOSP仓库的frameworks/volley目录下，它的核心内容包括请求调度的流水线以及“toolbox”目录内的一些常用工具类的实现。将Volley应用与你的项目的最简单的方法就是克隆Volley仓库，然后将其设置为类库项目。

1.  执行如下指令，将Volley项目克隆到本地。当然需要梯子了，没梯子的请自行百度下载。
`git clone https://android.googlesource.com/platform/frameworks/volley`
2. 将下载的源码作为类库项目导入到你的项目中，具体做法[传送门](https://developer.android.com/studio/projects/android-library.html)。

# 执行一个简单的请求

Volley框架的使用特别简单，只需要创建一个`RequestQueue`，然后将`Request`对象交给他就行了。`RequestQueue`维护了一组工作线程，这些工作线程的职责包括：执行网络请求、从缓存中读数据、往缓存中写数据、解析响应。`Request`对象会对其响应进行解析处理，然后Volley将解析过的响应转移到主线程处理。

通过`Volley.newRequestQueue`，可以方便的得到一个默认配置的`RequestQueue`，这节课将使用这个默认配置的`RequestQueue`发送一个`Request`。如果想了解如何自定义一个`RequestQueue`请见下节：[配置RequestQueue](#配置RequestQueue)。

这节主要介绍如何将一个`Request`添加到`RequestQueue`，以及如何取消一个`Request`

## 添加网络权限
要使用Volley，我们必须在我们App的manifest文件中添加`android.permission.INTERNET`权限。不添加的话，我们的App就无法访问网络。

## 使用newRequestQueue方法
Volley提供了一个便捷的方法`Volley.newRequestQueue`，通过这个方法，我们可以快速的获得一个默认配置的`RequestQueue`，并将其开启。
``` java
final TextView mTextView = (TextView) findViewById(R.id.text);
...

// 实例化一个RequestQueue
RequestQueue queue = Volley.newRequestQueue(this);
String url ="http://www.baidu.com";

// 请求制定URL的字符串响应
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
            new Response.Listener<String>() {
    @Override
    public void onResponse(String response) {
        // 展示响应字符串的前五百个字符
        mTextView.setText("Response is: "+ response.substring(0,500));
    }
}, new Response.ErrorListener() {
    @Override
    public void onErrorResponse(VolleyError error) {
        mTextView.setText("That didn't work!");
    }
});
// 将Request添加到RequestQueue
queue.add(stringRequest);
```
响应解析之后，Volley会将其转送到主线程。运行在主线程的好处就是我们可以很方便的使用得到的数据来填充UI控件，可以直接在请求的回调监听中进行UI操作。这一机能的实现对于存在其他重要功能尤其是取消功能的Volley来说是尤其苛刻的。

如果你不想使用`Volley.newRequestQueue`这个便捷的方法获取`RequestQueue`，请详见[配置RequestQueue](#配置RequestQueue)小节了解如何自定义`RequestQueue`，使其与你的项目更完美的结合。

如下是一个使用字符串作为TAG的例子

1. 定义TAG并添加到`Request`中
``` java
public static final String TAG = "MyTag";
StringRequest stringRequest; // 假定已经存在
RequestQueue mRequestQueue;  // 假定已经存在

// 添加TAG到Request中
stringRequest.setTag(TAG);

// 将Request添加到RequestQueue
mRequestQueue.add(stringRequest);
```

2. 在Activity的`onStop()`方法中，取消拥有此TAG的所有`Request`
``` java
@Override
protected void onStop () {
    super.onStop();
    if (mRequestQueue != null) {
        mRequestQueue.cancelAll(TAG);
    }
}
```
如果你的响应回调中包含譬如开启一个新的逻辑操作等业务逻辑的时候，在取消这个`Request`需要特别注意。

# 配置RequestQueue

通过上节我们知道，使用`Volley.newRequestQueue`方法可以很方便的得到一个默认配置的`RequestQueue`。这节介绍如何一步步的配置一个满足用户需求的`RequestQueue`。

此节中还推荐`RequestQueue`以单例的方式提供服务，这样整个App的生命周期内都可以使用Volley提供的网络请求服务了。

## 配置NetWork和Cache
`RequestQueue`在执行网络请求的时候主要以来两个东西：Network负责网络请求调用，Cache负责缓存处理。Network和Cache在Volley的toolbox中提供了两个标准实现：`DiskBasedCache`将每个响应作保存为一个文件，并在内存在提供索引。`BasicNetwork`基于优选的Http客户端负责请求用。

`BasicNetWork`是Volley的`NewWork`的默认实现。`BasicNetwork`初始的时候需要一个Http Client，通常是`HttpURLConnection`，通过这个Client真正连接网络。

如下代码片段展示了如何一步步配置Requestqueue
``` java
RequestQueue mRequestQueue;

// 实例化Cache
Cache cache = new DiskBasedCache(getCacheDir(), 1024 * 1024); // 1MB 容量

// 使用HttpURLConnection作为Http Client实例化Network
Network network = new BasicNetwork(new HurlStack());

// 使用上面的cache和network实例化Requestqueue
mRequestQueue = new RequestQueue(cache, network);

// 开启RequestQueue
mRequestQueue.start();

String url ="http://www.example.com";

// 构建Request并处理Response
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
        new Response.Listener<String>() {
    @Override
    public void onResponse(String response) {
        // 处理相应业务逻辑
    }
},
    new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            // 请求出错处理
    }
});

// 将Request添加到RequestQueue中
mRequestQueue.add(stringRequest);

// ...
```
如果你只想用Volley来完成一个一次性的请求，而且在请求完成后不想丢下Volley中的线程池不管，那么你可以在需要调用网络请求的任何地方创建一个`RequestQueue`，然后在收到请求响应或者收到错误的时候调用`stop`方法来停止`RequestQueue`。不过在具体的使用场景中，在整个App的生命周期内，我们随时都有可能调用网络请求，所以推荐`RequestQueue`使用单例的形式提供服务。下面详细介绍一下。
