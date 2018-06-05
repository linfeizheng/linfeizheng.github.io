---
title: 图文详解 变换操作符
date: 2018-05-24 12:04:18
---
![](index/1.png)

**Map( )**

作用
对 被观察者发送的每1个事件都通过 指定的函数 处理，从而变换成另外一种事件

> 即， 将被观察者发送的事件转换为任意的类型事件。

应用场景
数据类型转换

下面以将 使用Map（） 将事件的参数从 整型 变换成 字符串类型 为例子说明
![](index/2.png)

``` java
// 采用RxJava基于事件流的链式操作
Observable.create(new ObservableOnSubscribe<Integer>() {
    // 1. 被观察者发送事件 = 参数为整型 = 1、2、3
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        emitter.onNext(1);
        emitter.onNext(2);
        emitter.onNext(3);
    }
    // 2. 使用Map变换操作符中的Function函数对被观察者发送的事件进行统一变换：整型变换成字符串类型
}).map(new Function<Integer, String>() {
    @Override
    public String apply(Integer integer) throws Exception {
        return "使用 Map变换操作符 将事件" + integer + "的参数从 整型" + integer + " 变换成 字符串类型" + integer;
    }
}).subscribe(new Consumer<String>() {
    // 3. 观察者接收事件时，是接收到变换后的事件 = 字符串类型
    @Override
    public void accept(String s) throws Exception {
        Log.d(TAG, s);
    }
});
```
![](index/3.png)

**FlatMap( )**

作用
将被观察者发送的事件序列进行 拆分 & 单独转换，再合并成一个新的事件序列，最后再进行发送

原理

1. 为事件序列中每个事件都创建一个 Observable 对象；
2. 将对每个 原始事件 转换后的 新事件 都放入到对应 Observable对象；
3. 将新建的每个Observable 都合并到一个 新建的、总的Observable 对象；
4. 新建的、总的Observable 对象 将 新合并的事件序列 发送给观察者（Observer）
![](index/4.png)

应用场景
无序的将被观察者发送的整个事件序列进行变换

``` java
// 采用RxJava基于事件流的链式操作
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        emitter.onNext(1);
        emitter.onNext(2);
        emitter.onNext(3);
    }
    // 采用flatMap（）变换操作符
}).flatMap(new Function<Integer, ObservableSource<String>>() {
    @Override
    public ObservableSource<String> apply(Integer integer) throws Exception {
        final List<String> list = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            list.add("我是事件 " + integer + "拆分后的子事件" + i);
            // 通过flatMap中将被观察者生产的事件序列先进行拆分，再将每个事件转换为一个新的发送三个String事件
            // 最终合并，再发送给被观察者
        }
        return Observable.fromIterable(list);
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.d(TAG, s);
    }
});
```
![](index/5.png)

> 注：新合并生成的事件序列顺序是无序的，即 与旧序列发送事件的顺序无关

**ConcatMap( )**

作用
类似FlatMap（）操作符

与FlatMap（）的 区别在于：拆分 & 重新合并生成的事件序列 的顺序 = 被观察者旧序列生产的顺序

原理
![](index/6.png)

应用场景
有序的将被观察者发送的整个事件序列进行变换

``` java 
// 采用RxJava基于事件流的链式操作
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
        emitter.onNext(1);
        emitter.onNext(2);
        emitter.onNext(3);
    }
    // 采用concatMap（）变换操作符
}).concatMap(new Function<Integer, ObservableSource<String>>() {
    @Override
    public ObservableSource<String> apply(Integer integer) throws Exception {
        final List<String> list = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            list.add("我是事件 " + integer + "拆分后的子事件" + i);
            // 通过concatMap中将被观察者生产的事件序列先进行拆分，再将每个事件转换为一个新的发送三个String事件
            // 最终合并，再发送给被观察者
        }
        return Observable.fromIterable(list);
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(String s) throws Exception {
        Log.d(TAG, s);
    }
});
```
![](index/7.png)

> 注：新合并生成的事件序列顺序是有序的，即 严格按照旧序列发送事件的顺序
> 做个通俗的总结，变量转List使用FlatMap或ConcatMap，其他时候使用Map
> https://blog.csdn.net/guiyu_1985/article/details/54647434

**Buffer( )**

作用
定期从 被观察者（Obervable）需要发送的事件中 获取一定数量的事件 & 放到缓存区中，最终发送

原理
![](index/8.png)

应用场景
缓存被观察者发送的事件

``` java 
// 被观察者 需要发送5个数字
Observable.just(1, 2, 3, 4, 5)
        .buffer(3, 1) // 设置缓存区大小 & 步长
        // 缓存区大小 = 每次从被观察者中获取的事件数量
        // 步长 = 每次获取新事件的数量
        .subscribe(new Observer<List<Integer>>() {
            @Override
            public void onSubscribe(Disposable d) {
            }
            @Override
            public void onNext(List<Integer> stringList) {
                Log.d(TAG, " 缓存区里的事件数量 = " + stringList.size());
                for (Integer value : stringList) {
                    Log.d(TAG, " 事件 = " + value);
                }
            }
            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "对Error事件作出响应");
            }
            @Override
            public void onComplete() {
                Log.d(TAG, "对Complete事件作出响应");
            }
        });
```
![](index/9.png)

过程解释

![](index/10.png)

至此，关于RxJava2中主要的变换操作符已经讲解完毕