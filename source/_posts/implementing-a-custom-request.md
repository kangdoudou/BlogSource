---
title: Volley官方教程翻译（五）：自定义Request
date: 2016-09-28 10:27:51
tags: [android, volley]
---
> 原文连接：https://developer.android.com/training/volley/request-custom.html

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