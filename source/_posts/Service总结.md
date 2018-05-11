---
title: Service总结
date: 2017-09-27 20:14:37
tags: 控件
---
### 启动Service的方式

- Context.startService(Intent)：
``` java
Intent intent = new Intent(this, PlayService.class);
startService(intent);
```

- Context.bindService(Intent,ServiceConnection,BIND_AUTO_CREATE)
``` java
private ServiceConnection conn = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        Log.e("result","绑定成功");
        PlayService.MyBinder binder = (PlayService.MyBinder) service;
  	}
    @Override
    public void onServiceDisconnected(ComponentName name) {
        Log.e("result","解绑成功");
  	}
};
Intent intent = new Intent(this,PlayService.class);
bindService(intent,conn,BIND_AUTO_CREATE);
```

#### 两种启动方式区别：

1. startService启动之后，如果没有调用stopSelf()或者stopService()就会一直在后台运行。bindService启动Service之后，在启动它的组件被销毁后也会解绑并销毁。
2. startService()启动Service到结束，Service经历生命周期的是onCreate()、onStartCommand()、onDestory()。而bindService()启动Service，Service经历的生命周期是onCreate()、onBind()、onUnbind()、onDestory()
3. 如果即startService又bindService启动了Service，要分别unBindService()、stopSelf()/stopService()关闭Service。这里关闭没有顺序限制的，比如：先startService，后bindService。结束时先进行unBindService，stopService，还是顺序反过来都是没有问题的。
![](1.jpg)

#### 启动之后再次启动的生命周期

1. 在startService之后，再进行startService，只会再次执行Service的onStartCommond方法，而不会再执行onCreate
2. 在bindService之后，再进行bindService，不会再执行Service的onBind。

### 和Activity交互

- Activity在启动Service的时候，可以通过Intent.putExtra来给Service传递数据(两种方式均可)

- 通过bindService()启动Service，通过onBind()返回的Binder来指挥Service来进行操作
``` java
class MyBinder extends Binder{
    public void action()
}
```

- Service给Activity回传数据，可以通过在Service中发送广播，在Activity注册广播接收数据

### Service是运行在UI线程的

- 如果需要在Service中做耗时操作，需要另外起一个线程。
- 当然你还可以使用IntentService

### IntentService

**IntentService优点如下**

- Service中的事情处理完成之后，它会调用stopSelf()结束自己
- Service可以直接处理耗时操作

IntentService的基本使用如下：
``` java
public class DownloadService extends IntentService {
    public DownloadService() {
        super("DownLoadService");
    }
    @Override
    public void onCreate() {
        super.onCreate();
        Log.e("result", "onCreate");
    }
    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        super.onStart(intent, startId);
    }
    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        Log.e("result", "onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }
    @Override
    protected void onHandleIntent(Intent intent) {
        Log.e("result", "onHandleIntent");
    }
}
```

需要注意的是：它的构造方法必须给他提供一个无参构造方法，名字就命名为类名即可。

**IntentService的原理**

IntentService封装了HandlerThread和Handler，HandlerThread就是一个Thread（具体再分析），Handler handler = new Handler(handerThread.getLooper())，handler是由子线程中的Looper构造的，相当于在子线程中有个一个MessageQueue消息队列，我们每次startService()，就是sendMessage()添加一个消息到队列中，然后该childThread的Looper不断的从MessageQueue中取出消息去处理。

记住：Handler构造中的Looper在哪个线程，该Handler就为哪个线程服务。如果是无参构造就是MainThread，因为内部调用了looper.myLooper()。

### 补充

onStartCommand()方法的返回值问题：

1. START_STICKY：如果service进程被kill掉，保留service的状态为开始状态，但不保留递送的intent对象。随后系统会尝试重新创建service，由于服务状态为开始状态，所以创建服务后一定会调用onStartCommand(Intent,int,int)方法。如果在此期间没有任何启动命令被传递到service，那么参数Intent将为null。
2. START_NOT_STICKY：“非粘性的”。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统不会自动重启该服务。
3. START_REDELIVER_INTENT：重传Intent。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统会自动重启该服务，并将Intent的值传入。
4. START_STICKY_COMPATIBILITY：START_STICKY的兼容版本，但不保证服务被kill后一定能重启。