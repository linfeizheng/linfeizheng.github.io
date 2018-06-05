---
title: Android性能优化
date: 2017-11-12 14:06:01
tags: 优化
---
Android性能优化，是Android开发中的重中之中，本文将讲述如何进行性能优化。

实际项目中的Android性能优化主要有如下几个方面：

1. 编写高效代码—开发中总结出的一些小的性能Tips
2. Layout布局优化
3. 内存优化

### 编写高效代码

编写高效代码的两个原则
1. 不要写不需要的代码
2. 不要分配不必要的内存

以上两个原则，似乎感觉是废话，但确实是编程的最高境界，也是我们编写代码的过程中时刻需要思考和注意的两个方面。

那么如何做到如上两点呢？下面列出了一些实际开发中的例子。

**避免产生不必要的对象**

例如：

- int的数组比Integer对象数组要好得多。两个平行的int数组要比一个（int,int）型的对象数组高效。这对于其他任何基本数据类型的组合都通用

- 两个平行数组Foo[],Bar[]会优于一个（Foo,Bar）对象的数组

- 通常来讲，尽量避免创建短时零时对象.少的对象创建意味着低频的垃圾回收

对象的分配和回收都是需要代价的；分配的内存越多，就会引起强制的内存回收；给用户体验增加小的停顿间隙，从而影响用户体验。

用户能感觉到卡顿的时间延迟是100ms ~ 200ms。

**用静态代替虚拟**

如果方法不需要访问某对像的字段，将该方法设置为静态，调用速度会提升15%~20%

对于常量使用static final 

static final int i = 1;         
static final String s = "a";

> 注：这种优化仅仅是针对基本数据类型和String类型常量的，而非任意的引用类型。但尽可能的将常量声明为static final是一种好的做法。

**避免内部的getter和setter**

**使用增强for循环**

增强for循环要比普通循环快3倍

**避免使用浮点数**

通常，浮点数会比整型慢2/3

**在没有JIT的设备上，调用方法所传递的对象采用具体的类型而非接口类型会更高效**

- void methodA(List<String> list);
- void methodA(ArrayList<String> list);

如上，后一种比前一种更高效。

**数据库操作方法的优化
尽量利用原生的SQL语句**

原生的SQL省去了拼接sql语句的步骤，要比SqliteDatabase提供的insert、query、 update、delete等函数效率高。当数据库越大，差别也越大

当操作条数较多时，利用事务进行批处理

这样SQLite将把全部要执行的SQL语句先缓存在内存当中，然后等到COMMIT的时候一次性的写入数据库，这样数据库文件只被打开关闭了一次，效率自然大大的提高

``` java
db.beginTransaction();        
for(Collection c:colls){
    insert(db, c);
} 
db.setTransactionSuccessful();
```

**Http请求方式的选择**
Android 内置了两种HTTP方式:HttpURLConnection 和 Apache HttpClient。这两种都支持HTTPS、流式上传和下载、可配置超时、IPv6和连接池。在Gingerbread或者更高版本时，推荐使用HttpURLConnection。

这是因为： HttpURLConnection API 更简单，包更小。同时对传输数据的压缩和响应的缓存处理减少了网络带宽、提高了速度，也节省了电量。

### 优化布局
Layouts是Android应用里直接影响用户体验的一个关键部分。如果Layout设计的不好，可能导致你的应用大量的内存占用从而导致UI响应很慢。Android SDK提供了工具帮助你分析你的Layouts的性能问题。

**使用Hierarchy Viewer**

Hierarchy Viewer工具位于SDK \tools\目录下，该工具能分析出你的布局不合理和可以优化的地方。

大多数情况下，布局渲染时间差别较大的原因是在LinaerLayout里使用了layout_weight。这将会增加测量(Measure)的时间。你应该仔细的考虑是否有必要使用layout weight。

**使用Lint**

使用Lint — 查看你的view层级哪些地方可以优化

- 使用compound drawables - 一个包含了ImageView与TextView的LinearLayout可以被当作一个compound drawable来处理

- 使用<merge/> - 如果FramLayout仅仅是一个纯粹的（没有设置背景，间距等）布局根元素，我们可以使用merge标签来当作根标签

- 无用的分支 - 如果一个layout并没有任何子组件，那么可以被移除，这样可以提高效率

- 无用的父控件 - 如果一个layout只有子控件，没有兄弟控件，并且不是一个ScrollView或者根节点，而且没有设置背景，那么我们可以移除这个父控件，直接把子控件提升为父控件

- 深层次的layout - 尽量减少内嵌的层级，考虑使用更多平级的组件 RelativeLayout or GridLayout来提升布局性能，默认最大的深度是10

**其他一些布局要点**

- 使用include标签
- 使用ViewStub标签

### 优化App内存
为了垃圾回收器能回收你系统的内存，你应该避免引起内存泄露，而且要在合适的时间点释放被引用的对象。

**慎用Service**

1. Service执行完后台任务后要停止

2. 使用IntentService

IntentService不同于普通的Service之处是：

- 提交的task系统会post到子线程运行

- 当后台运行的task完成时，系统会stop掉IntentService

**Release memory when your user interface becomes hidden**

例如，在该onStop（）里做释放资源（例如网络连接、注销广播等）的工作

**使用优化后的集合容器**

例如：SparseArray、SparseBooleanArray、LongSpareArray …..

**尽量避免使用枚举**

相比于静态常量，枚举会有超过其两倍以上的内存开销，在android中需严格避免使用枚举

**避免使用依赖注入框架**

**使用ProGuard消除没有使用的代码**

**使用zipalign优化和对齐你的apk**

**使用MAT分析和优化内存**

1. I/O使用后需要关闭，数据库和Cursor等使用后要关闭

2. 使用finalize()+MAT 分析内存泄露

end 

Android优化主要就是内存、布局和性能的优化，本文总结了Android中优化的一些知识点。如果还有其他我没有讲到的，欢迎给我留言。