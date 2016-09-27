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
- 取消请求接口。你可以单独取消一个请求，你也可以通过划定范围取消多个请求
- 方便定制。比如说：重试和退避请求
- 强大的排序功能，使得你能够很容易的使用从网络异步获取的数据填充你的UI
- 调试与追踪工具

Volley尤其擅长用来填充UI的RPC调用操作，比如获取一组结构化的搜索结果。它很容易与其他协议集成，而且已经实现了常用的、原始的String，Image与Json协议请求。Volley提供了许多在日常开发中所需要的网络层的功能，避免我们去重复造轮子，专注于业务逻辑代码的实现。

Volley不是用于大文件下载或者数据流的操作，因为在处理响应的时候，Volley把整个响应都保存在内存中。如果要操作大文件，请考虑使用DownloadManager。

Volley库源码位于AOSP仓库的frameworks/volley目录下，它的核心内容包括请求调度的流水线以及“toolbox”目录内的一些常用工具类的实现。将Volley应用与你的项目的最简单的方法就是克隆Volley仓库，然后将其设置为类库项目。

1.  执行如下指令，将Volley项目克隆到本地。当然需要梯子了，没梯子的请自行百度下载。
`git clone https://android.googlesource.com/platform/frameworks/volley`
2. 将下载的源码作为类库项目导入到你的项目中，具体做法[传送门](https://developer.android.com/studio/projects/android-library.html)。

# 执行一个简单的请求
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

## 使用单例模式

如果你的App要经常使用网络的话，那么让`RequestQueue`与你的App生命周期一样是最高效的解决办法。实现的方法多种多样，我们推荐实现一个单例类来封装`RequestQueue`以及Volley的其它功能。另外一个方法是，实现一个Application的子类，然后在`Application.onCreate()`方法中配置`RequestQueue`。第二种方法不推荐使用，因为这两中方法实现的功能一样，不过单例模式更能体现模块化的思想。

请注意，在实例化`RequestQueue`的时候，使用ApplicationContext，不要使用Activity的Context



# 自定义Request

如果Volley自带的JsonRequest、StringRequest等不能满足我们的需求，那么我就需要自定义Request了。

## 实现一个Request
在toolbox中Volley已经帮我们实现好了常用的Request。如果你的请求响应是String、Json抑或Image的话，你大概不需要去实现Request。
如果你真的需要实现一个自己的Request的话。你只需要做如下的事情：
- 继承`Request<T>`类，这里泛型`<T>`是你期待的解析后的请求的响应类型。比如，如果你希望将响应解析为字符串的话，只需要继承`Request<String>`就可以了。这里你可以参考toolbox里面的`StringRequest`和`ImageRequest`，参考以下如何继承`Request<T>`.

- 实现`parseNetworkResponse()`和`deliverResponse()`这两个抽象方法，下面详细介绍这两个方法。

### parseNetworkResponse

Volley用`Response<T>`封装`Request<T>`所指定类型(String、Json)的响应。如下是`parseNetworkResponse()`的一个实现示例：

``` java
@Override
protected Response<T> parseNetworkResponse(
        NetworkResponse response) {
    try {
        String json = new String(response.data,
        HttpHeaderParser.parseCharset(response.headers));
    return Response.success(gson.fromJson(json, clazz),
    HttpHeaderParser.parseCacheHeaders(response));
    }
    // 错误处理
...
}
```

流程说明：

- 方法`parseNetworkResponse()`的参数为`NetworkResponse`，其内容包括二进制格式的响应负载、HTTP状态码以及响应头。
- 实现的方法必须的返回结果必须是`Response<T>`。如果解析成功返回结果中将包含指定类型的响应对象以及缓存的元数据，如果失败结果里就是失败的信息。

如果你的协议中缓存的要求与标准缓存不同，你可以自定义自己的缓存实现。不过大多数请求都可以按照如下写法：

``` java
return Response.success(myDecodedObject,
        HttpHeaderParser.parseCacheHeaders(response));
```

Volley只会在工作线程调用`parseNetworkResponse()`方法。这一点保证了诸如将JPEG图片解析为一个Bitmap对象这样的耗时操作不会阻塞UI线程。

### deliverResponse

Volley会将`parseNetworkResponse()`方法返回的对象转移到主线程调用。大多数请求都会在这里调用回调接口。示例：

``` java
protected void deliverResponse(T response) {
        listener.onResponse(response);
```

## Example: GsonRequest

Gson库使用反射技术，能够把Json字符串转换成Java对象，也可以反过来将Java对象转换成Json字符串。Java对象的属性与Json的key需要对应起来，Gson才能够进行填充。如下是使用Gson解析请求响应结果的完整实现示例：

``` java
public class GsonRequest<T> extends Request<T> {
    private final Gson gson = new Gson();
    private final Class<T> clazz;
    private final Map<String, String> headers;
    private final Listener<T> listener;

    /**
     * 构造一个GET请求，其返回结果是解析自JSON字符串的对象
     *
     * @param url 请求的URL
     * @param clazz 解析JSON字符串后生成该类的对象
     * @param headers Map of request headers
     */
    public GsonRequest(String url, Class<T> clazz, Map<String, String> headers,
            Listener<T> listener, ErrorListener errorListener) {
        super(Method.GET, url, errorListener);
        this.clazz = clazz;
        this.headers = headers;
        this.listener = listener;
    }

    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        return headers != null ? headers : super.getHeaders();
    }

    @Override
    protected void deliverResponse(T response) {
        listener.onResponse(response);
    }

    @Override
    protected Response<T> parseNetworkResponse(NetworkResponse response) {
        try {
            String json = new String(
                    response.data,
                    HttpHeaderParser.parseCharset(response.headers));
            return Response.success(
                    gson.fromJson(json, clazz),
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        }
    }
}
```

Volley已经帮我们实现好了`JsonArrayRequest`和`JsonArrayObject`，可以方便使用。详情请参见[使用标准请求](使用标准请求)。