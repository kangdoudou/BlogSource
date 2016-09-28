---
title: Volley官方教程翻译（四）：使用标准请求
date: 2016-09-28 10:23:44
tags: [android, volley]
---
> 原文连接：https://developer.android.com/training/volley/request.html

这节描述怎么使用Volley提供的一些常用类型的请求。
- `StringRequest`. 请求特定URL，响应为原生字符串。具体示例参见：[配置RequestQueue](配置RequestQueue)。
- `ImageRequest`. 请求特定URL，响应为图片。
- `JsonObjectRequest` 和 `JsonArrayRequest` (两个都是JsonRequest的子类). 请求特定URL，响应为Json对象或者数组。

如果你期望的响应是如上的类型的话，就不需要在自定义Request了。这节介绍怎么使用这些标准的Request。如果想了解如何自定义Request，请参见[自定义Request](自定义Request).

# 请求图片

Volley提供了如下几个请求图片使用的类。其实它们之间的关系是相互依赖，为我们提供不同层次级别的服务。

- `ImageRequest` 一个获取指定URL图片的Request，最终响应结果是一个解析了的Bitmap。也有比如改变图片大小等许多方便的功能。它最大的优点是所有的耗时重量级操作，比如解码、改变大小，都在工作线程执行。


- `ImageLoader` 一个负责加载与缓存远程图片的辅助类。`ImageLoader`可以管理大量的`ImageRequest`，比如，当往一个`List`中填充许多缩略图的时候。`ImageLoader`在Volley的缓存之上又提供了一层内存缓存，这一点能够有效避免图片的闪烁现象。如果一个图片存在内存缓存中，我们可以直接得到它，不用阻塞以及切离主线程，这一点Disk缓存是做不到的。`ImageLoader`能够对响应进行聚合处理。如果不聚合处理的话，每一个响应都会往一个View上设置一个Bitmap，导致一次布局调用。聚合能够定时把多个响应传送出来，提高了效率。

- `NetworkImageView` 基于ImageLoader实现。在图片是从网络上获取的情况下，能够高效替换ImageView。如果view被移除，NetworkImageView也负责管理request的取消操作。

## 使用ImageRequest

如果是一个使用ImageRequest的实例。展示了如何获取指定URL的图片，然后展示到应用上。值得注意的是如下代码段里的RequestQueue是通过单例类（详细讨论请参见：[配置RequestQueue](/2016/09/27/setting-up-a-requestqueue)）获得的。

``` java
ImageView mImageView;
String url = "http://i.imgur.com/7spzG.png";
mImageView = (ImageView) findViewById(R.id.myImage);
...

// 获取指定URL的图片，展示到UI上
ImageRequest request = new ImageRequest(url,
    new Response.Listener<Bitmap>() {
        @Override
        public void onResponse(Bitmap bitmap) {
            mImageView.setImageBitmap(bitmap);
        }
    }, 0, 0, null,
    new Response.ErrorListener() {
        public void onErrorResponse(VolleyError error) {
            mImageView.setImageResource(R.drawable.image_load_error);
        }
    });
// 通过单例类获得RequestQueue
MySingleton.getInstance(this).addToRequestQueue(request);
```

## 使用 ImageLoader 和 NetworkImageView

ImageLoader和NetworkImageView可以配合使用来高效管理譬如ListView多图的展示。在XML布局文件中，也可以像使用ImageView一样使用NetworkImageView，如下：

``` xml
<com.android.volley.toolbox.NetworkImageView
        android:id="@+id/networkImageView"
        android:layout_width="150dp"
        android:layout_height="170dp"
        android:layout_centerHorizontal="true" />
```

你也可以只使用ImageLoader来展示图片：

``` java
ImageLoader mImageLoader;
ImageView mImageView;
// 将被加载的图片的URL
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
...
mImageView = (ImageView) findViewById(R.id.regularImageView);

// 通过单例类获得ImageLoader
mImageLoader = MySingleton.getInstance(this).getImageLoader();
mImageLoader.get(IMAGE_URL, ImageLoader.getImageListener(mImageView,
         R.drawable.def_image, R.drawable.err_image));
```

下面是如何使用NetworkImageView来加载这个图片：

``` java
ImageLoader mImageLoader;
NetworkImageView mNetworkImageView;
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
...

// 获得布局文件中定义的NetworkImageView
mNetworkImageView = (NetworkImageView) findViewById(R.id.networkImageView);

// 通过单例类获得ImageLoader
mImageLoader = MySingleton.getInstance(this).getImageLoader();

// 把URL和加载该URL图片的ImageLoader设置给NetworkImageView
mNetworkImageView.setImageUrl(IMAGE_URL, mImageLoader);
```

上面代码片段通过单例类获得RequestQueue和ImageLoader，这部分在[配置RequestQueue](/2016/09/27/setting-up-a-requestqueue)这节有详细描述。使用单例类这种方法整个App只有一个RequestQueue和ImageLoader实例，而且生命周期与App一致。这样做的一个很重要的原因是内存缓存能够似的在设备旋转的时候无闪烁现象。单例模式能够使得缓存高于Activity而存在。如果你选择了在Activity中穿件ImageLoader的话，那么在旋转设备的时候，它会随Activity一起重建，导致图片闪烁现象。

### LRU缓存示例

Volley的工具盒中提供了一个基于DiskBasedCache类的标准实现。这个类直接将响应生产文件缓存到硬盘的指定目录下。但是使用ImageLoader，需要提供一个实现ImageLoader.ImageCache接口的内存缓存。你大概趋向于用单例模式来配置缓存。更多关于单例模式的讨论，请参见[配置RequestQueue](/2016/09/27/setting-up-a-requestqueue)。

如下是一个内存缓存的简单实现类LruBitmapCache，它继承自LruCache类实了ImageLoader.ImageCache接口：

``` java
import android.graphics.Bitmap;
import android.support.v4.util.LruCache;
import android.util.DisplayMetrics;
import com.android.volley.toolbox.ImageLoader.ImageCache;

public class LruBitmapCache extends LruCache<String, Bitmap>
        implements ImageCache {

    public LruBitmapCache(int maxSize) {
        super(maxSize);
    }

    public LruBitmapCache(Context ctx) {
        this(getCacheSize(ctx));
    }

    @Override
    protected int sizeOf(String key, Bitmap value) {
        return value.getRowBytes() * value.getHeight();
    }

    @Override
    public Bitmap getBitmap(String url) {
        return get(url);
    }

    @Override
    public void putBitmap(String url, Bitmap bitmap) {
        put(url, bitmap);
    }

    // Returns a cache size equal to approximately three screens worth of images.
    public static int getCacheSize(Context ctx) {
        final DisplayMetrics displayMetrics = ctx.getResources().
                getDisplayMetrics();
        final int screenWidth = displayMetrics.widthPixels;
        final int screenHeight = displayMetrics.heightPixels;
        // 4 bytes per pixel
        final int screenBytes = screenWidth * screenHeight * 4;

        return screenBytes * 3;
    }
}
```

如下是如何使用这个内存缓存实例话ImageLoader的示例：

``` java
RequestQueue mRequestQueue; // assume this exists.
ImageLoader mImageLoader = new ImageLoader(mRequestQueue, new LruBitmapCache(
            LruBitmapCache.getCacheSize()));
```

# 请求JSON数据

Volley提供了如下两个类来完成Json请求：
- JsonArrayRequest 一个获取指定URL的响应为Json数组的请求
- JsonObjectRequest 一个获取指定URL的响应为Json对象的请求，请求体可以包含一个Json对象。

以上两个类都基于JsonRequest类。它们的使用方法与其他类型的Request一样。如下的代码片段展示了如何获取Json数据以及将它们作为文本显示在UI上：

``` java
TextView mTxtDisplay;
ImageView mImageView;
mTxtDisplay = (TextView) findViewById(R.id.txtDisplay);
String url = "http://my-json-feed";

JsonObjectRequest jsObjRequest = new JsonObjectRequest
        (Request.Method.GET, url, null, new Response.Listener<JSONObject>() {

    @Override
    public void onResponse(JSONObject response) {
        mTxtDisplay.setText("Response: " + response.toString());
    }
}, new Response.ErrorListener() {

    @Override
    public void onErrorResponse(VolleyError error) {
        // TODO Auto-generated method stub

    }
});

// Access the RequestQueue through your singleton class.
MySingleton.getInstance(this).addToRequestQueue(jsObjRequest);
```
下一节[自定义Request](/2016/09/28/implementing-a-custom-request)，将会介绍如何基于Gson自定义一个Json请求。