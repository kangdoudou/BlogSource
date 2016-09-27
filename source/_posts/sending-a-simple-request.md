---
title: Volley官方教程翻译（二）：执行一个简单的请求
date: 2016-09-27 22:24:51
tags: android volley
---
> 原文连接：https://developer.android.com/training/volley/index.html

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

## 发送一个Request
想要发送执行一个`Request`，只需要像如上代码一样，构建一个`Request`对象，然后通过`add()`方法添加到`RequestQueue`中就可以了。`Request`被添加之后，接下来就等待被执行、解析原始响应数据并将结果转送到主线程执行。

Volley内部维护者一个缓存处理线程和一组网络调度线程。当一个`Request`被添加之后，它首先被缓存线程处理。如果缓存命中，则直接在缓存线程中对缓存的响应解析处理，然后转送到主线程。如果缓存线程不能处理，`Request`将被添加到网络请求的`Queue`中，由一个网络调度线程负责执行一次HTTP请求，解析响应数据，转送到主线程。

值得注意的一点是，重量级的操作，比如阻塞I/O、响应解码处理都是在工作线程上执行的。你可以在任意线程上添加`Request`，但响应总会被转送到主线程。

![Request的一生](/2016/09/20/volley-official-tutorials/volley-request.png "Request的一生")

## 取消请求
通过调用`Request`对象的`cancel()`方法，可以取消这一个`Request`。一旦一个`Request`被取消，那么它的响应监听将永远不会被调用。这就意味着，你可以在`Activity`的`onStop()`方法中取消所有还没有执行完的`Request`。这样的话，我们就不需要在`Request`的响应回调中判断是否`getActivity() == null`或者`onSaveInstanceState()`有没有已经被调用这些预防性的判断了，因为被取消后根本就不存在这种case了。

为了能够充分利用这一特点，我们需要追踪所有的`Request`以在适当的时机取消它们。Volley提供了一个很好的解决方案：将每一个Request与一个TAG对象关联起来，这样就可以根据TAG取消指定的一部分Request了。比如，你可以使用Activity对象作为所有Request的TAG，然后在`onStop`方法中执行`requestQueue.cancelAll(this)`。同样，在使用ViewPager的时候，将一个Tab中的所有Request打上该Tab的TAG，然后在切换Tab的时候取消上一个Tab的所有Request。

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
