---
title: ThreadLocal的实现
date: 2016-08-24 20:11:50
tags: java
---

# 1. ThreadLocal的使用
ThreadLocal类可以让你创建的变量只被同一个线程进行读和写操作。因此，尽管有两个线程同时执行一段相同的代码，而且这段代码又有一个指向同一个ThreadLocal变量的引用，但是这两个线程依然不能看到彼此的ThreadLocal变量域。
ThreadLocal的使用方法也特别简单：

``` java
ThreadLocal<String> strThreadLocal = new ThreadLocal<String>(); // 声明ThreadLocal对象
strThreadLocal.set("My thread id is:" + Thread.currentThread().getId() ); // 存储
String str = strThreadLocal.get(); // 获取
```
通过ThreadLocal<T>可以很方便的让线程拥有自己的"私有对象"。下面来看看它是怎么为线程创造出来的这个"私有对象"。整个实现主要涉及到Thread，ThreadLocalMap以及ThreadLcal, 他们的具体分工如下：

Thread："私有对象"，即泛型对象T实际存储位置。
ThreadLocalMap：程序中可以有多个ThreadLocal<T>对象，这就意味着，Thread对象中要存储多个对象T，ThreadLocalMap就是对象T在Thread中存储的数据结构
ThreadLocal：线程中获取"私有对象"的中介。

# 2. ThreadLocal
## 2.1 set() get()的实现

``` java
public void set(T value) {
    Thread t = Thread.currentThread(); // 获取当前线程
    ThreadLocalMap map = getMap(t); // 拿到当前线程中的ThreadLocalMap对象
    if (map != null)
        map.set(this, value); // 如果ThreadLocalMap已经存在，则以该ThreadLocal对象为Key存储。
    else
        createMap(t, value); // 如果ThreadLocalMap还未创建，创建并存储
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this); // 获取Map的Entry
        if (e != null)
            return (T)e.value; // 获取存储的对象
    }
    return setInitialValue(); // 返回默认值。通过复写initialValue方法设置默认值。
}
```
可以看出set get方法就是往ThreadLocalMap存与取。

# 3. ThreadLocalMap
ThreadLocalMap可以说是一个自定义的HashMap。那么为什么不直接用HashMap作为存储结构呢？因为会出现内存泄露。
因为对象的存储位置是Thread这个对象中，一旦对象使用的生命周期小于Thread的生命周期就会导致，对象无法被GC回收，内存泄露。
## 使用弱引用作为Map的Entry
为了防止内存泄露，Entry采用了很巧妙的一种办法，让Entry实现为ThreadLocal的弱引用，已ThreadLocal对象为Key，内部包裹一个Value对象。
``` java
static class Entry extends WeakReference<ThreadLocal> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal k, Object v) {
        super(k);
        value = v;
    }
}
```
这样当除了ThreadLocalMap以外如果没有其他引用到ThreadLocal对象的话，那么在GC发生的时候，弱引用Entry所指向的ThreadLocal对象就会被回收。
而且以Entry.get()得到ThreadLocal对象作为Key， Entry.value作为当前线程，对应该key的ThreadLocal的线程“私有对象”。

## 3.1 hashcode的计算
hash算法的宗旨就是尽量的将较多的key均匀分布在有限的槽中。在ThreadLocalMap中的key是ThreadLocal，自然hashcode的计算就落到的ThreadLocal身上。
结合考虑ThreadLocal使用的常见场景，ThreadLocal对象都会被连续的添加到每个线程的ThreadLocalMap中，所以设计者采用了一种“自增”的实现方法。自增的起始值是0x61c88647。也就是说第一个被new出来的ThreadLocal对象的hashcode为0x61c88647，第二被new出来的ThreadLocal对象的hashcode就是0x61c88648，以此类推。而且这种实现方法在那些并不常见的场景中效果也是很好的。

## 3.2 一维存储结构
注意到Entry的实现与HashMap中不同，没有Entry.next，所以其无法实现HashMap的那种二维存储，只能是一维存储结构。
那么在发生hash冲突的时候，解决办法也就是简单的后移。贴上ThreadLocalMap 的set 与 get 方法

``` java
private void set(ThreadLocal key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1); // hashcode 与 数组长度结合初步计算出存储位置

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) { // 查看该位置是否已经存储了Entry
        ThreadLocal k = e.get();

        if (k == key) { // 如果该位置存放的key value与要设的一直，直接return
            e.value = value;
            return;
        }

        if (k == null) { // 如果该位置存放的ThreadLocal已经被回收，替换它
            replaceStaleEntry(key, value, i);
            return;
        }
                             // 不是以上情况，就查看下一个位置是否能够存储 
    }

    tab[i] = new Entry(key, value); // 找到合适位置存储
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold) // 是否要扩充
        rehash();
}


private Entry getEntry(ThreadLocal key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key) //当前Entry不为空， 且key相等说明命中
        return e;
    else
        return getEntryAfterMiss(key, i, e); //没有命中继续再找
}

private Entry getEntryAfterMiss(ThreadLocal key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal k = e.get();
        if (k == key) // 命中返回
            return e;
        if (k == null) // ThreadLocal已经被回收，将当前Slot消去
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len); // 没有命中，挪到下一个
        e = tab[i];
    }
    return null;
}

```

# 4. Thread
Thread是ThreadLocalMap的载体，结合其实际作用，不难想象。也没有太多好说的。
