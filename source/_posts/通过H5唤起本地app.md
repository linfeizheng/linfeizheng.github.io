---
title: 通过H5唤起本地app
date: 2017-05-08 21:33:43
tags: 其他
---

H5如何打开或者说唤起手机本地的app，有以下两种：

第一种方式：

通过在html的a标签里面的href中直接配置android端的schema，当然，如果有host其他的配置，跟在后面就可以了，android端配置和代码如下：

android端配置：
``` java
<activity android:name = ".MainActivity">
    <intent-filter>
        <action android:name = "android.intent.action.MAIN"/>
        <category android:name = "android.intent.category.LAUNCHER"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:host="baidu.com"
           android:scheme="testapp"/>
    </intent-filter>
</activity>
```
注：如果这个是配置在启动页要和标签并列在一起，不然运行后手机app的图标会没有；注意schema协议要小写,否则会有不能响应的异常!

html代码：

``` html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Insert title here</title>
    </head>
	<body>
    	<a href="testapp://baidu.com/?pid=1">打开app</a>
	</body>
</html>
```

这里我们来看看schema拼接协议的格式：

< a href="[scheme]://[host]/[path]?[query]">启动应用程序< /a>

各个项目含义如下所示：

scheme：判别启动的App。

host：适当记述

path：传值时必须的key ※没有也可以

query：获取值的Key和Value ※没有也可以

以上就能实现打开本地的app了，当然是在app存在的情况下，否则的话没有反应。

我们有些时候在唤起本地app的时候可能会向app传递一些参数，这些参数我们就可以配置在path或者query里，我们只需要在oncreate里面获取就可以了。

代码如下：

``` java
Intent intent = getIntent();
    Uri uri = intent.getData();
    if (uri != null) {
        String pid = uri.getQueryParameter("pid");
}
```

如果还想要获取android里面配置的schema协议的话，还可以这样:

``` java
Uri uri = getIntent().getData();
if(uri != null) { 
    // 完整的url信息
    String url = uri.toString();
    Log.e(TAG, "url: "  + uri); 
    // scheme部分
    String scheme = uri.getScheme();
    Log.e(TAG, "scheme: "  + scheme); 
    // host部分
    String host = uri.getHost();
    Log.e(TAG, "host: "  + host); 
    //port部分
    int port = uri.getPort();
    Log.e(TAG, "host: "  + port); 
    // 访问路径
    String path = uri.getPath();
    Log.e(TAG, "path: "  + path);
    List<String> pathSegments = uri.getPathSegments(); 
    // Query部分
    String query = uri.getQuery();
    Log.e(TAG, "query: "  + query); 
    //获取指定参数值
    String pid = uri.getQueryParameter("pid");
    Log.e(TAG, "pid: "  + pid);
}
```

如何判断一个Schema是否有效 :

``` java
PackageManager packageManager = getPackageManager();
Intent intent = newIntent(Intent.ACTION_VIEW, Uri.parse("testapp://baidu.com:80/article?goodsId=10011002"));
List<ResolveInfo> activities = packageManager.queryIntentActivities(intent, 0);
booleanisValid = !activities.isEmpty();
if(isValid) {
    startActivity(intent);
}
```

这种方式也是我百度到的最多的方式，但是这样就带来了一个问题了，上面的需求说的是“在页面上有一个连接， 如果用户安装了APP，则点击打开对应的APP；如果用户没有安装，则点击打开对应的设置连接”，这明显就不符合需求了，这只能作为一些个别需求来使用了。

第二种方式：

既然通过在href配置schema协议不行，那就只能通过js代码来实现了，只有这样才能根据判断实现app有的时候就打开，没有的时候就跳转到下载链接下载。

我们知道，js是无法判断手机是否安装了某款app的，所以我们只能够曲线救国了，我们可以获取时间如果，长时间不能呼起app则默认为没有安装这款app，然后跳转到下载页。当然这不是我想出来的，是网上的各位大佬的想法。在这里又要细分为两种情况了。

1.直接唤醒

说明：通过h5可换醒app，如访问一个URL，点击按钮，打开应用，如果该应用APP没有安装，那么直接跳转到App Store的APP下载页面，通过点击的方式兼容性较好，如果安装了app，在手机各大浏览器(360浏览器、uc浏览器、搜狗浏览器、QQ浏览器、百度浏览器 )和QQ客户端中，能唤醒。微信、新浪微博客户端、腾讯微博客户端无法唤醒。

代码如下：

``` html
<html xmlns=http://www.w3.org/1999/xhtml>
<head>
<meta http-equiv=Content-Type content="text/html;charset=utf-8">
<head>
<script src="http://libs.baidu.com/jquery/1.9.0/jquery.js"></script>
<title>点击唤醒demo</title>
</head>
<body>
<style>#zjmobliestart{font-size:40px;}
</style>

<!--
说明：通过h5可换醒app，如访问一个URL，点击按钮，打开应用，如果该应用APP没有安装，那么直接跳转到App Store的APP下载页面，通过点击的方式。兼容性较好，如果安装了app，在手机各大浏览器(360浏览器 uc浏览器 搜狗浏览器 QQ浏览器 百度浏览器 )和
QQ客户端中，能唤醒。微信 新浪微博客户端 腾讯微博客户端无法唤醒。
-->

<a href="zjmobile://platformapi/startapp" id="zjmobliestart" target="_blank">唤醒浙江移动手机营业厅！</a>

<script type="text/javascript"> 
function applink(){  
    return function(){  
        var clickedAt = +new Date;  
         setTimeout(function(){
             !window.document.webkitHidden && setTimeout(function(){ 
                   if (+new Date - clickedAt < 2000){  
                       window.location = 'https://itunes.apple.com/us/app/zhe-jiang-yi-dong-shou-ji/id898243566#weixin.qq.com';  
                   }  
             }, 500);
         }, 500)   
    };  
}
applink();
</script>   
</body>
</html>
```

2.点击唤醒

说明：通过h5可换醒app，如访问一个URL就能直接打开应用，如果该应用APP没有安装，那么直接跳转到App Store的APP下载页面。兼容性一般：在手机各大浏览器(360浏览器、uc浏览器、搜狗浏览器 QQ浏览器、百度浏览器 )能唤醒。微信、QQ客户端、新浪微博客户端、 腾讯微博客户端无法唤醒。

代码如下：


``` html
<!Doctype html><html xmlns=http://www.w3.org/1999/xhtml>
<head>
<meta http-equiv=Content-Type content="text/html;charset=utf-8"><head>
<script src="http://libs.baidu.com/jquery/1.9.0/jquery.js"></script>
<title>直接唤醒demo</title>
</head>
<body>
<style>#zjmobliestart{font-size:40px;}
</style>

<!--
说明：通过h5可换醒app，如访问一个URL就能直接打开应用，如果该应用APP没有安装，那么直接跳转到App Store的APP下载页面
兼容性一般：在手机各大浏览器(360浏览器 uc浏览器 搜狗浏览器 QQ浏览器 百度浏览器 )能唤醒。微信 QQ客户端 新浪微博客户端 腾讯微博客户端无法唤醒。
-->

<p id="zjmobliestart">唤醒浙江移动手机营业厅！</p>

<script type="text/javascript"> function applink(){   
    window.location = 'zjmobile://platformapi/startapp';  
        var clickedAt = +new Date;  
         setTimeout(function(){
             !window.document.webkitHidden && setTimeout(function(){ 
                   if (+new Date - clickedAt < 2000){  
                       window.location = 'https://itunes.apple.com/us/app/zhe-jiang-yi-dong-shou-ji/id898243566#weixin.qq.com';  
                   }  
             }, 500);       
         }, 500)   

}
document.getElementById("zjmobliestart").onclick = applink();
</script>   
</body>
</html>
```

这样就完成了我们的需求了。

还要注意的是，如果是在微信中唤起本地app，手机的微信中，是利用微信内置的浏览器打开那个简单的HTML页面，注意：直接打开scheme://host/datastring是不可行的，微信不会把这串字符解析成网址，必须包装成网页才能借助微信的浏览器打开。进入后就是我们刚刚设计的页面。这个时候，直接点击“启动应用程序”是不会唤醒之前安装的APP的，因为微信做了屏蔽，你需要在右上角的菜单中选择“在浏览器中打开”。这个时候就可以唤醒你想唤醒的app了。