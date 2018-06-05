---
title: notebook
date: 2018-05-02 10:59:57
---
#### popupwindow动画
``` java
mTypePopupWindow.setAnimationStyle(R.style.popupwindow_anim);
```
``` xml
<style name="popupwindow_anim">  
    <item name="android:windowEnterAnimation">@anim/popupwindow_anim_in</item>  
    <item name="android:windowExitAnimation">@anim/popupwindow_anim_out</item>  
</style>
```

#### 修改DrawerLayout的阴影的颜色
``` java
mDrawer.setScrimColor(Color.parseColor("#20000000"));
```

#### LinearLayout自带的分割线
``` xml
android:divider="@drawable/divider"
android:showDividers="middle"
```

#### 插值器Interpolator
AccelerateDecelerateInterpolator 在动画开始与介绍的地方速率改变比较慢，在中间的时候加速
AccelerateInterpolator 在动画开始的地方速率改变比较慢，然后开始加速
AnticipateInterpolator 开始的时候向后然后向前甩
AnticipateOvershootInterpolator 开始的时候向后然后向前甩一定值后返回最后的值
BounceInterpolator 动画结束的时候弹起
CycleInterpolator 动画循环播放特定的次数，速率改变沿着正弦曲线
DecelerateInterpolator 在动画开始的地方快然后慢
LinearInterpolator 以常量速率改变
OvershootInterpolator 向前甩一定值后再回到原来位置



