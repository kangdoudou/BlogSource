---
title: Binder
date: 2016-07-12 18:58:37
tags: android
---

稍微深入了解Android知识的时候就会知道有Binder这个东西的存在，也知道它的作用是用来进程间通信的，那么它是怎么实现进程间通信的呢？下面就是java层面，了解一下Binder是怎么实现进程间通信的。

# transact() 与 onTransact()
Binder的进程间过程可以简单的概括为以下三个步骤：
1. Client进程获取Server进程的Binder (系统服务通过Context.getSystemService, 自定义服务一般是通过bindService)
2. Client进程调用Binder.transact()。
3. Server进程的Binder.onTransact()被触发。(至于怎么被触发的就是Binder底层驱动做的事情了)

# 四个参数
transact() 与 onTransact()有四个参数(int code, Parcel data, Parcel reply, int flags)， 分别介绍一下每一个的意思：

1.code 每一个Server提供的服务可能不止一个，当Client发起transact时，code充当的是请求代码的角色。
2.data 一个Parcel对象，存储了所以的请求参数
3.reply 一个Parcel对象，存储返回值
4.flags 附加的操作标志参数。0代表一次正常的RPC调用，1代表单方向的RPC调用(调用直接返回，不等待调用结果)

# 接口抽象
虽然通过Binder我们可以轻松实现IPC通信，但如果不对Binder做任何封装，每次IPC依然是复杂的，Client和Server端不单每次要记住每种请求的code，还要明确每一个请求参数和返回值。

> 例:Text服务，一共有两个简单的方法echo和reverse

``` java
public interface IText extends IInterface{

	// 供客户端调用
	String echo(String text);
	String reverse(String text);
	
	// 服务端的真正实现
	String onEcho(String text);
	String onReverse(String text);
	
}
```

``` java
public class TextManager extends Binder implements IText{
	
	private static final int ECHO = Binder.FIRST_CALL_TRANSACTION;
	private static final int REVERSE = Binder.FIRST_CALL_TRANSACTION+1;
	@Override
	public IBinder asBinder() {
		return this;
	}
	
	@Override
	public String echo(String text) {
		String result = null;
		
		Parcel data = Parcel.obtain();
		Parcel reply = Parcel.obtain();
		data.writeString(text);
		this.transact(ECHO, data, reply, 0);
		result = reply.readString();
		
		return result;
	}

	@Override
	public String reverse(String text) {
		String result = null;
		
		Parcel data = Parcel.obtain();
		Parcel reply = Parcel.obtain();
		data.writeString(text);
		this.transact(ECHO, data, reply, 0);
		result = reply.readString();
		
		return result;
	}
	
	@Override
	protected boolean onTransact(int code, Parcel data, Parcel reply, int flags)
			throws RemoteException {
		
		switch (code) {
		case ECHO:
			String result1 = onEcho(data.readString());
			reply.writeString(result1);
			return true;
		case REVERSE:
			String result2 = onReverse(data.readString());
			reply.writeString(result2);
			return true;
		default:
			return super.onTransact(code, data, reply, flags);
		}
	}
	
	@Override
	public String onEcho(String text) {
		return text;
	}

	@Override
	public String onReverse(String text) {
		return (String) TextUtils.getReverse(text, 0, text.length());
	}
}
```