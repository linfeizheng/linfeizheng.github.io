---
title: 面试题
date: 2018-03-03 19:16:04
---
## Android
#### Android的四大组件是哪些，它们的作用？
Activity：Activity是Android程序与用户交互的窗口，是Android构造块中最基本的一种，它需要为保持各界面的状态，做很多持久化的事情，妥善管理生命周期以及一些跳转逻辑；
Service：后台服务于Activity，封装有一个完整的功能逻辑实现，接受上层指令，完成相关的事务，定义好需要接收的Intent提供同步和异步的接口；
ContentProvider：是Android提供的第三方应用数据的访问方案，可以派生Content Provider类，对外提供数据，可以像数据库一样进行选择排序，屏蔽内部数据的存储细节，向外提供统一的接口模型，大大简化上层应用，对数据的整合提供了更方便的途径；
BroadcastReceiver：接收一种或者多种Intent作触发事件，接收相关消息转换成一条Notification，统一了Android的事件广播模型。

#### Activity的生命周期?
onCreate() - onStart() - onResume() - onPause() - onstop() - onDestroy() - onRestart()

#### 后台的Activity被系统回收怎么办?
onSaveInstanceState()
``` java
protected void onSaveInstanceState(BundleoutState) {  
	super.onSaveInstanceState(outState);  
	outState.putLong("id",1234567890);  
}  
public void onCreate(BundlesavedInstanceState) {  
	//判断savedInstanceState是不是空.  
	//如果不为空就取出来  
	super.onCreate(savedInstanceState);  
}  
```

#### android 横竖屏切换生命周期?
不设置Activity的android:configChanges时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次；
设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次；
设置Activity的android:configChanges="orientation|screenSize"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法。

#### 什么是ANR，如何避免它?
用户操作在5秒内应用程序未做出响应，或者BroadcastReceiver在10秒内没有执行完毕。

#### Android的系统架构?
Android系统架构从上往下为Linux内核层、运行库、应用程序框架层、应用程序层。

#### 谈谈Android的IPC（进程间通信）机制?
Android中的IPC机制是为了让Activity和Service之间可以随时的进行交互。

#### 如何将SQLite数据库与apk文件一起发布?
可以将db文件复制到工程中的res\raw目录中，raw中的文件不会被压缩。

#### 常用到的设计模式?
1. 单例模式

懒汉式

``` java
//懒汉式单例类.在第一次调用的时候实例化自己   
public class Singleton {  
    private Singleton() {}  
    private static Singleton single=null;  
    //静态工厂方法   
    public static Singleton getInstance() {  
         if (single == null) {    
             single = new Singleton();  
         }    
        return single;  
    }  
}  
```

饿汉式

``` java
//饿汉式单例类.在类初始化时，已经自行实例化   
public class Singleton {  
    private Singleton() {}  
    private static final Singleton single = new Singleton();  
    //静态工厂方法   
    public static Singleton getInstance() {  
        return single;  
    }  
} 
```
2. 适配器模式
3. 观察者模式
4. 建造者模式

#### 4种启动模式?
1. standard：默认
2. singleTop：如果有对应的Activity实例正位于栈顶，则重复利用，不再生成新的实例。
3. singleTask：如果有对应的Activity实例，则使此Activity实例之上的其他Activity实例统统出栈，使此Activity实例成为栈顶对象。
4. singleInstance：启用一个新的栈，

#### Service的两种启动方式?
第一种：startService
> 
1.定义一个类继承Service
2.在Manifest.xml文件中配置该Service
3.使用Context的startService(Intent)方法启动该Service
4.不再使用时，调用stopService(Intent)方法停止该服务

使用这种start方式启动的Service的生命周期如下：
onCreate() ---> onStartCommand()（onStart()方法已过时） ---> onDestory()。
如果服务已经开启，不会重复的执行onCreate()，而是会调用onStart()和onStartCommand()。服务停止的时候调用 onDestory()。服务只会被停止一次。
**特点：**一旦服务开启跟调用者(开启者)就没有任何关系了。
开启者退出了，开启者挂了，服务还在后台长期的运行。
开启者不能调用服务里面的方法。

第二种：bindService
> 
1.定义一个类继承Service
2.在Manifest.xml文件中配置该Service
3.使用Context的bindService(Intent, ServiceConnection, int)方法启动该Service
4.不再使用时，调用unbindService(ServiceConnection)方法停止该服务

使用这种start方式启动的Service的生命周期如下：
onCreate() ---> onBind() ---> onunbind() ---> onDestory()
**特点：**bind的方式开启服务，绑定服务，调用者挂了，服务也会跟着挂掉。
绑定者可以调用服务里面的方法。

#### 两种注册、发送广播的方式?
第一种：代码中动态注册
``` java
//new出上边定义好的BroadcastReceiver
MyBroadCastReceiver yBroadCastReceiver = new MyBroadCastReceiver();

//实例化过滤器并设置要过滤的广播  
IntentFilter intentFilter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");

//注册广播   
myContext.registerReceiver(smsBroadCastReceiver,intentFilter, 
             "android.permission.RECEIVE_SMS", null);
```

第二种：Manifest.xml中静态注册
``` xml
<receiver android:name=".MyBroadCastReceiver">  
            <!-- android:priority属性是设置此接收者的优先级（从-1000到1000） -->
            <intent-filter android:priority="20">
            <action android:name="android.provider.Telephony.SMS_RECEIVED"/>  
            </intent-filter>  
</receiver>
```
**两种注册广播的不同**
第一种不是常驻型广播，也就是说广播跟随程序的生命周期。
第二种是常驻型，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行。
**有两种方式分别发送两种不同的广播：**
通过mContext.sendBroadcast(Intent)或mContext.sendBroadcast(Intent, String)发送的是无序广播(后者加了权限)；
通过mContext.sendOrderedBroadcast(Intent, String, BroadCastReceiver, Handler, int, String, Bundle)发送的是有序广播。

#### HTTPS和HTTP的区别?
HTTPS = HTTP + SSL
HTTP 和 HTTPS 的不同之处
1. HTTP 是不安全的，而 HTTPS 是安全的
2. HTTP 标准端口是 80 ，而 HTTPS 的标准端口是 443
3. 在 OSI 网络模型中，HTTP 工作于应用层，而 HTTPS 工作在传输层
4. HTTP 无需加密，而 HTTPS 对传输的数据进行加密
5. HTTP 无需证书，而 HTTPS 需要认证证书

#### 排序算法(选择，冒泡)
一：冒泡排序
![](冒泡.jpg)
基本思想：两个数比较大小，较大的数下沉，较小的数冒起来。
过程：比较相邻的两个数据，如果第二个数小，就交换位置。
从后向前两两比较，一直到比较最前两个数据。最终最小数被交换到起始的位置，这样第一个最小数的位置就排好了。
继续重复上述过程，依次将第2.3...n-1个最小数排好位置。

``` java
public static void sort(int[] arr) {
    int temp;//临时变量
    for (int i = 0; i < arr.length - 1; i++) {//表示趟数，一共arr.length-1次。
        for (int j = arr.length - 1; j > i; j--) {
            if (arr[j] < arr[j - 1]) {
                temp = arr[j];
                arr[j] = arr[j - 1];
                arr[j - 1] = temp;
            }
        }
    }
}
```
二：选择排序
基本思想：
在长度为N的无序数组中，第一次遍历n-1个数，找到最小的数值与第一个元素交换；
第二次遍历n-2个数，找到最小的数值与第二个元素交换；
第n-1次遍历，找到最小的数值与第n-1个元素交换，排序完成。

过程：
![](选择.jpg)
``` java
public static void sort(int array[]) {
    for (int i = 0; i < array.length - 1; i++) {
        int minIndex = i;
        for (int j = i + 1; j < array.length; j++) {
            if (array[j] < array[minIndex]) {
                minIndex = j;
            }
        }
        if (minIndex != i) {
            int temp = array[i];
            array[i] = array[minIndex];
            array[minIndex] = temp;
        }
    }
}
```

#### 5种进程的优先级?
- 空进程
- 后台进程
- 服务进程
- 可见进程
- 前台进程

#### FragmentPageAdapter和FragmentPageStateAdapter的区别
- FragmentPageAdapter在每次切换页面的的时候，是将Fragment进行分离，适合页面较少的Fragment使用以保存一些内存，对系统内存不会多大影响。
- FragmentPageStateAdapter在每次切换页面的时候，是将Fragment进行回收，适合页面较多的Fragment使用，这样就不会消耗更多的内存。



## Java
#### Switch能否用string做参数？
在 Java 7之前，switch 只能支持 byte、short、char、int或者其对应的封装类以及 Enum 类型。在 Java 7中，String支持被加上了。

#### equals与==的区别?
==比较的是2个对象的地址，而equals比较的是2个对象的内容。

#### String、StringBuffer与StringBuilder的区别?
1. StringBuilder：线程非安全的 、StringBuffer：线程安全的
a.如果要操作少量的数据用 = String
b.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder 
c.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

#### Collection与Collections的区别?
Collections是个java.util下的类，它包含有各种有关集合操作的静态方法。
Collection是个java.util下的接口，它是各种集合结构的父接口。

#### Override和Overload的含义去区别?
Overload：函数里面有相同的函数名但是参数名、返回值、类型不相同。
Override：在子类继承父类的时候，将方法继承过来。

#### interface与abstract类的区别?
1. 抽象类可以有构造方法，接口中不能有构造方法。
2. 抽象类中可以包含非抽象的普通方法，接口中的所有方法必须都是抽象的。
3. 抽象类中可以包含静态方法，接口中不能包含静态方法。
4. 一个类可以实现多个接口，但只能继承一个抽象类。
5. 都不能被实例化。
6. 接口里只能定义常量，不能定义普通成员变量；抽象类里则即可以定义普通成员变量，也可以定义静态常量。

#### 解析XML的几种方式?
DOM、SAX、PULL

#### 设计模式六大原则?
要点 | 定义 | 描述
---|---|---
单一职责原则 | 不要存在多于一个导致类变更的原因。通俗的说，即一个类只负责一项职责。 | 问题由来：类T负责两个不同的职责：职责P1，职责P2。当由于职责P1需求发生改变而需要修改类T时，有可能会导致原本运行正常的职责P2功能发生故障。解决方案：遵循单一职责原则。分别建立两个类T1、T2，使T1完成职责P1功能，T2完成职责P2功能。这样，当修改类T1时，不会使职责P2发生故障风险；同理，当修改T2时，也不会使职责P1发生故障风险。
里氏替换原则 |    定义1：如果对每一个类型为 T1的对象 o1，都有类型为 T2 的对象o2，使得以 T1定义的所有程序 P 在所有的对象 o1 都代换成 o2 时，程序 P 的行为没有发生变化，那么类型 T2 是类型 T1 的子类型。定义2：所有引用基类的地方必须能透明地使用其子类的对象。 | 问题由来：有一功能P1，由类A完成。现需要将功能P1进行扩展，扩展后的功能为P，其中P由原有功能P1与新功能P2组成。新功能P由类A的子类B来完成，则子类B在完成新功能P2的同时，有可能会导致原有功能P1发生故障。解决方案：当使用继承时，遵循里氏替换原则。类B继承类A时，除添加新的方法完成新增功能P2外，尽量不要重写父类A的方法，也尽量不要重载父类A的方法。
依赖倒置原则 | 高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。 | 问题由来：类A直接依赖类B，假如要将类A改为依赖类C，则必须通过修改类A的代码来达成。这种场景下，类A一般是高层模块，负责复杂的业务逻辑；类B和类C是低层模块，负责基本的原子操作；假如修改类A，会给程序带来不必要的风险。解决方案：将类A修改为依赖接口I，类B和类C各自实现接口I，类A通过接口I间接与类B或者类C发生联系，则会大大降低修改类A的几率。
接口隔离原则 | 客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上。 | 问题由来：类A通过接口I依赖类B，类C通过接口I依赖类D，如果接口I对于类A和类B来说不是最小接口，则类B和类D必须去实现他们不需要的方法。解决方案：将臃肿的接口I拆分为独立的几个接口，类A和类C分别与他们需要的接口建立依赖关系。也就是采用接口隔离原则。
迪米特法则 | 一个对象应该对其他对象保持最少的了解。 | 问题由来：类与类之间的关系越密切，耦合度越大，当一个类发生改变时，对另一个类的影响也越大。解决方案：尽量降低类与类之间的耦合。










