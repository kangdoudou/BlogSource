---
title: Volley官方教程翻译（三）：配置RequestQueue
date: 2016-09-27 23:00:05
tags: [android, volley]
---
> 原文连接：https://developer.android.com/training/volley/requestqueue.html
通过上节我们知道，使用`Volley.newRequestQueue`方法可以很方便的得到一个默认配置的`RequestQueue`。这节介绍如何一步步的配置一个满足用户需求的`RequestQueue`。

此节中还推荐`RequestQueue`以单例的方式提供服务，这样整个App的生命周期内都可以使用Volley提供的网络请求服务了。

## 配置Network和Cache
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

请注意，在实例化`RequestQueue`的时候，使用ApplicationContext，不要使用Activity的Context。因为ApplicationContext的生命周期是整个App的生命周期。

如下是提供了`RequestQueue`和`ImageLoad`功能的一个单例实现例子。

``` java
public class MySingleton {
    private static MySingleton mInstance;
    private RequestQueue mRequestQueue;
    private ImageLoader mImageLoader;
    private static Context mCtx;

    private MySingleton(Context context) {
        mCtx = context;
        mRequestQueue = getRequestQueue();

        mImageLoader = new ImageLoader(mRequestQueue,
                new ImageLoader.ImageCache() {
            private final LruCache<String, Bitmap>
                    cache = new LruCache<String, Bitmap>(20);

            @Override
            public Bitmap getBitmap(String url) {
                return cache.get(url);
            }

            @Override
            public void putBitmap(String url, Bitmap bitmap) {
                cache.put(url, bitmap);
            }
        });
    }

    public static synchronized MySingleton getInstance(Context context) {
        if (mInstance == null) {
            mInstance = new MySingleton(context);
        }
        return mInstance;
    }

    public RequestQueue getRequestQueue() {
        if (mRequestQueue == null) {
            // getApplicationContext() 这句代码是关键
            // 因为mCtx可能不是ApplicationContext，结果就很可能泄露这个mCtx
            mRequestQueue = Volley.newRequestQueue(mCtx.getApplicationContext());
        }
        return mRequestQueue;
    }

    public <T> void addToRequestQueue(Request<T> req) {
        getRequestQueue().add(req);
    }

    public ImageLoader getImageLoader() {
        return mImageLoader;
    }
}
```

如下是如何使用这个单例执行网络请求。

``` java
// 获得RequestQueue
RequestQueue queue = MySingleton.getInstance(this.getApplicationContext()).
    getRequestQueue();

// ...

// 往RequestQueue中添加一个Request
MySingleton.getInstance(this).addToRequestQueue(stringRequest);
```



