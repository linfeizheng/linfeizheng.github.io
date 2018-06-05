---
title: Android面试题
date: 2018-03-03 19:16:04
---

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

#### Activity如与Service通信？
可以通过bindService的方式，先在Activity里实现一个ServiceConnection接口，并将该接口传递给bindService()方法，在ServiceConnection接口的onServiceConnected()方法里执行相关操作。

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
第一种不是常驻型广播，跟随程序的生命周期。
第二种是常驻型，不受组件生命周期影响，即便应用退出，广播还是可以被接收，耗电、占内存。
**有两种方式分别发送两种不同的广播：**
通过mContext.sendBroadcast(Intent)或mContext.sendBroadcast(Intent, String)发送的是无序广播(后者加了权限)；
通过mContext.sendOrderedBroadcast(Intent, String, BroadCastReceiver, Handler, int, String, Bundle)发送的是有序广播。

#### 5种进程的优先级?
- 空进程
- 后台进程
- 服务进程
- 可见进程
- 前台进程

#### FragmentPageAdapter和FragmentPageStateAdapter的区别
- FragmentPageAdapter在每次切换页面的的时候，是将Fragment进行分离，适合页面较少的Fragment使用以保存一些内存，对系统内存不会多大影响。
- FragmentPageStateAdapter在每次切换页面的时候，是将Fragment进行回收，适合页面较多的Fragment使用，这样就不会消耗更多的内存。