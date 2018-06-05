---
title: Android混淆总结
date: 2017-12-23 19:27:42
tags: 优化
---
通过代码混淆可以将项目中的类、方法、变量等信息进行重命名，变成一些无意义的简短名字，同时也可以移除未被使用的类、方法、变量等。所以直观的看，通过混淆可以提高程序的安全性，增加逆向工程的难度，同时也有效缩减了apk的体积。

### 开启混淆

在基于Android Studio项目的app module的build.gradle中有如下默认代码片段：

``` gradle
buildTypes {
    release {
        minifyEnabled true
        shrinkResources true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

minifyEnabled true 代表要发布的release包的混淆配置，默认不开启混淆，设为true表示开启混淆。

shrinkResources true 代表开启资源文件压缩。

proguard-android.txt 代表系统默认的混淆规则配置文件，该文件在<Android SDK目录>/tools/proguard下，一般不要更改该配置文件。

proguard-rules.pro 代表当前module的混淆配置文件，可以通过修改该文件来添加当前项目的混淆规则。

### 编写混淆配置文件
以下是系统的proguard-android.txt

``` gradle
# This is a configuration file for ProGuard.
# http://proguard.sourceforge.net/index.html#manual/usage.html
# 混淆时不使用大小写混合类名
-dontusemixedcaseclassnames
# 不跳过library中的非public的类
-dontskipnonpubliclibraryclasses
# 打印混淆的详细信息
-verbose
# Optimization is turned off by default. Dex does not like code run
# through the ProGuard optimize and preverify steps (and performs some
# of these optimizations on its own).
# 关闭优化（原因见上边的原英文注释）
-dontoptimize
# 不进行预校验，可加快混淆速度
-dontpreverify
# Note that if you want to enable optimization, you cannot just
# include optimization flags in your own project configuration file;
# instead you will need to point to the
# "proguard-android-optimize.txt" file instead of this one from your
# project.properties file.
# 保留注解中的参数
-keepattributes *Annotation*
# 不混淆如下两个谷歌服务类
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService
# For native methods, see http://proguard.sourceforge.net/manual/examples.html#native
# 不混淆包含native方法的类的类名以及native方法名
-keepclasseswithmembernames class * {
    native <methods>;
}
# keep setters in Views so that animations can still work.
# see http://proguard.sourceforge.net/manual/examples.html#beans
# 不混淆View中的setXxx()和getXxx()方法，以保证属性动画正常工作
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}
# We want to keep methods in Activity that could be used in the XML attribute onClick
# 不混淆Activity中参数是View的方法，例如，一个控件通过android:onClick="clickMethodName"绑定点击事件，混淆后会导致点击事件失效
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}
# For enumeration classes, see http://proguard.sourceforge.net/manual/examples.html#enumerations
# 不混淆枚举类中的values()和valueOf()方法
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
# 不混淆Parcelable实现类中的CREATOR字段，以保证Parcelable机制正常工作
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}
# 不混淆R文件中的所有静态字段，以保证正确找到每个资源的id
-keepclassmembers class **.R$* {
    public static <fields>;
}
# The support library contains references to newer platform versions.
# Don't warn about those in case this app is linking against an older
# platform version.  We know about them, and they are safe.
# 不对android.support包下的代码警告（如果我们打包的版本低于support包下某些类的使用版本，会出现警告的问题）
-dontwarn android.support.**
# Understand the @Keep support annotation.
# 不混淆Keep类
-keep class android.support.annotation.Keep
# 不混淆使用了注解的类及类成员
-keep @android.support.annotation.Keep class * {*;}
# 如果类中有使用了注解的方法，则不混淆类和类成员
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}
# 如果类中有使用了注解的字段，则不混淆类和类成员
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}
# 如果类中有使用了注解的构造函数，则不混淆类和类成员
-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}
```
**keep关键字：**

关键字  | 含义
---|---
keep | 保留类和类成员，防止被混淆或移除
keepnames | 保留类和类成员，防止被混淆，但没有被引用的类成员会被移除
keepclassmembers | 只保留类成员，防止被混淆或移除
keepclassmembernames | 只保留类成员，防止被混淆，但没有被引用的成员会被移除
keepclasseswithmembers | 保留类和类成员，防止被混淆或移除，如果指定的类成员不存在还是会被混淆
keepclasseswithmembernames | 保留类和类成员，防止被混淆，如果指定的类成员不存在还是会被混淆，没有被引用的类成员会被移除
**相关通配符：**

通配符 | 含义
---|---
* | 匹配任意长度字符，但不含包名分隔符.。例如一个类的全包名路径是com.othershe.test.Person，使用com.othershe.test.*、com.othershe.test.*都是可以匹配的，但com.othershe.*就不能匹配
** | 匹配任意长度字符，并包含包名分隔符.。例如要匹配com.othershe.test.**包下的所有内容
*** | 匹配任意参数类型。例如*** getName(***)可匹配String getName(String)
... | 匹配任意长度的任意类型参数。例如void setName(...)可匹配void setName(String firstName, String secondName)
&lt;fileds&gt; | 匹配类、接口中所有字段
&lt;methods&gt; | 匹配类、接口中所有方法
&lt;init&gt; | 匹配类中所有构造函数

以上就是混淆的基本语法，系统的proguard-android.txt已经为我们完成了大部分基础的混淆配置工作，至于编写当前app module下的proguard-rules.pro，只需要针对当前项目添加一些特有的配置，避免某些重要的东西被混淆掉就可以了，我们主要考虑以下几点：
1. 在AndroidManifest.xml中注册的继承四大组件的子类的类名以及重写的方法名都不会被混淆。

比如，如果希望项目中android.support.v4.app.Fragment子类的类名和重写父类的方法名不被混淆可以添加如下配置
``` gradle
# 不混淆Fragment的子类类名以及onCreate()、onCreateView()方法名
-keep public class * extends android.support.v4.app.Fragment {
    public void onCreate(android.os.Bundle);
    public android.view.View onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle);
}
```
2. 不混淆某个特定的类和类中所有成员
``` gradle
-keep class com.test.utils.CommonUtil { *; }
```
3. 不混淆某个目录下的文件，例如使用Gson时，数据bean不能被混淆，需要如下配置：
``` gradle
# com.test.bean代表所有bean所在的目录
-keep class com.test.bean.** { *; }
```

4. 保留泛型
``` gradle
-keepattributes Signature
```

5. 保留用于调试堆栈跟踪的行号信息
``` gradle
-keepattributes SourceFile,LineNumberTable
```

6. WebView中使用了JS调用，需要添加如下配置：
``` gradle
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
   public *;
}
```
7. 项目中使用的第三方库的混淆规则，这里列举几个常用的：
``` gradle
# okhttp
-dontwarn okhttp3.**
-dontwarn okio.**
-dontwarn javax.annotation.**
-keepnames class okhttp3.internal.publicsuffix.PublicSuffixDatabase
# Retrofit
-dontwarn okio.**
-dontwarn javax.annotation.**
-dontnote retrofit2.Platform
-dontwarn retrofit2.Platform$Java8
-keepattributes Signature
-keepattributes Exceptions
# RxJava RxAndroid
-dontwarn sun.misc.**
-keepclassmembers class rx.internal.util.unsafe.*ArrayQueue*Field* {
    long producerIndex;
    long consumerIndex;
}
-keepclassmembers class rx.internal.util.unsafe.BaseLinkedQueueProducerNodeRef {
    rx.internal.util.atomic.LinkedQueueNode producerNode;
}
-keepclassmembers class rx.internal.util.unsafe.BaseLinkedQueueConsumerNodeRef {
    rx.internal.util.atomic.LinkedQueueNode consumerNode;
}
# Gson
-keep class com.google.gson.stream.** { *; }
-keepattributes EnclosingMethod
# xxx代表model类的全包名路径
-keep class xxx.** { *; }
# butterknie
-keep class butterknife.** { *; }
-dontwarn butterknife.internal.**
-keep class **$$ViewBinder { *; }
-keepclasseswithmembernames class * {
    @butterknife.* <fields>;
}
-keepclasseswithmembernames class * {
    @butterknife.* <methods>;
}
# eventbus
-keepattributes *Annotation*
-keepclassmembers class ** {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }
# Only required if you use AsyncExecutor
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
```
### 查看混淆结果
混淆后打包，会在app module/build/outputs/mapping/release目录下生成如下文件

![](1.jpg)

- dump.txt：描述apk文件中所有类的内部结构
- mapping.txt：混淆前后的类、类成员、方法的对照关系（重要，追溯Crash堆栈信息要用到）
- resources.txt：资源文件的压缩信息
- seeds.txt：未被混淆的类和成员
- usage.txt：被移除的代码

最后还是有必要看一下混淆后的代码结构，验证混淆是否成功。一个简单的办法，Android Studio的Build菜单下有一个Analyze APK选项，只需要先选择要分析的apk包，在之后的界面点击classes.dex即可看到混淆后的代码结构：

![](2.jpg)

如果看到a b c d e 这样的包名，说明已经混淆成功了。

### 追溯Crash信息

代码混淆后，也会导致Crash堆栈信息被混淆，难以阅读，增加定位问题位置的难度，一个混淆后的Crash堆栈信息类似这样，核心的信息都没了：

![](3.jpg)

为了解决这个问题，可以使用<SDK目录>\tools\proguard\bin下的proguardgui.bat脚本将Crash堆栈信息还原到混淆前的状态。步骤如下：
- 双击打开脚本，选择左边的ReTrace选项
- 选择Mapping file文件，也就是混淆后打包后在app module/build/outputs/mapping/release下生成的mapping.txt
- 拷贝混淆后的堆栈信息
- 点击右下角的ReTrace!按钮，完成Crash堆栈信息的追溯

如下图中间部分就是追溯到的原Crash堆栈信息：

![](4.jpg)

代码混淆基本的内容就这些了。