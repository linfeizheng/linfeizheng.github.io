---
title: apk瘦身
date: 2018-05-02 11:24:49
tags: 优化
---
1. 代码混淆     
主流开源项目的混淆规则列表
https://github.com/krschultz/android-proguard-snippets
2. Android Studio → Analyze → Inspect Code
删掉无用的代码和图片
3. 使用tinypng、智图等图片压缩工具对图片进行压缩  
https://tinypng.com/
http://zhitu.isux.us/
4. 资源文件混淆
介绍：http://www.tuicool.com/articles/qMVbIbi
github：https://github.com/shwenzhang/AndResGuard
5. 字体文件压缩
github：https://github.com/forJrking/FontZip
6. 减少第三库的使用
 
7. 使用9.png图  
上跟左，表示拉伸；右跟下，表示内容范围
8. 用代码代替图片
shape代替圆形
9. 用RotateDrawable代替仅仅是方向不同的“内容相同”的图片
![](1.jpg)
这里两个图片是两个按钮箭头，但是仅仅方向不同而已，其实可以只用其中一个图片即可，而另一个用RotateDrawable来让其“调转”180度
``` xml
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/ic_arrow_left"
    android:fromDegrees="180"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toDegrees="180" />
```
10. 用layer-list来制作多层图片从而达到复用  
![](2.jpg)    
有些需求中需要一种图片，但是明显这个图片是其他几个图片简单叠加而已，那么可以使用layer-list来达到目的
``` xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <!-- 最底层的图片，以x，y轴坐标为中心进行旋转-->
        <rotate android:pivotX="0" android:pivotY="0"
            android:fromDegrees="-10" android:toDegrees="-10">
            <bitmap android:src="@mipmap/ic_launcher"/>
        </rotate>
    </item>
    <!-- 第二层的图片，以x，y轴坐标为中心进行旋转-->
    <item>
        <rotate android:pivotX="0" android:pivotY="0"
            android:fromDegrees="15" android:toDegrees="15">
            <bitmap android:src="@mipmap/ic_launcher"/>
        </rotate>
    </item>
    <!-- 最上层的图片，以x，y轴坐标为中心进行旋转-->
    <item>
        <rotate android:pivotX="0" android:pivotY="0"
            android:fromDegrees="35" android:toDegrees="55">
            <bitmap android:src="@mipmap/ic_launcher"/>
        </rotate>
    </item>
</layer-list>
```
11. 通过配置resConfig可以选择只打包哪几种语言，如果只保留默认英语和中文语言，配置如下
``` gradle
android{
    defaultConfig{
        resConfigs "zh", "en"
    }
}
```
12. 谷歌redex对字节码进行优化
https://github.com/facebook/redex.git
13. 避免枚举
一个枚举可以为您的应用程序的classes.dex文件添加大约1.0到1.4 KB的大小。