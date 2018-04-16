---
title: Android中的各种Span
date: 2017-04-16 20:38:02
tags: 控件
---
### 简介
在android.text.style包下,有一些Span类,可以提供我们完成一些在TextView中的特殊内容。(比如:部分内容颜色、字体、大小不同等等)。本篇博客主要讲解一些span的用法，以及结合Android动画机制制作出十分酷炫的动画Span。

### 给字符添加边框
FrameSpan实现给相应的字符序列添加边框的效果，整体思路其实比较简单。

1. 计算字符序列的宽度；
2. 根据计算的宽度、上下坐标、起始坐标绘制矩形；
3. 绘制文字

展现效果如下所示：

![](1.jpg)

再来看一下代码，其实代码十分简单。
``` java
public class FrameSpan extends ReplacementSpan {
    private final Paint mPaint;
    private int mWidth;
    public FrameSpan() {
        mPaint = new Paint();
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setColor(Color.BLUE);
        mPaint.setAntiAlias(true);
    }
    @Override
    public int getSize(Paint paint, CharSequence text, int start, int end, Paint.FontMetricsInt fm) {
        //return text with relative to the Paint
        mWidth = (int) paint.measureText(text, start, end);
        return mWidth;
    }
    @Override
    public void draw(Canvas canvas, CharSequence text, int start, int end, float x, int top, int y, int bottom, Paint paint) {
        //draw the frame with custom Paint
        canvas.drawRect(x, top, x + mWidth, bottom, mPaint);
        canvas.drawText(text, start, end, x, y, paint);
    }
}
```
用法示例：
``` java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    TextView tv = findViewById(R.id.id_tv_framespan);
    final SpannableString spannableString = new SpannableString(
            "helloworld,helloworld!helloworld,helloworld!helloworld," +
                    "helloworld!helloworld,helloworld!");
    spannableString.setSpan(new FrameSpan(), 0, 30, Spanned.SPAN_INCLUSIVE_INCLUSIVE);
    tv.setText(spannableString);
}
```
### 图文垂直居中
系统提供的ImageSpan和DynamicDrawableSpan只能实现图片和文字底部对齐或者是baseline对齐，现在VerticalImageSpan可以实现图片和文字居中对齐。

![](2.jpg)

图中的图片保持了和文字居中对齐，现在来看看VerticalImageSpan的源码。
``` java 
public class VerticalImageSpan extends ImageSpan {
    private Drawable drawable;
    public VerticalImageSpan(Drawable drawable) {
        super(drawable);
        this.drawable=drawable;
    }
    @Override
    public int getSize(Paint paint, CharSequence text, int start, int end, Paint.FontMetricsInt fontMetricsInt) {
        Drawable drawable = getDrawable();
        if(drawable==null){
            drawable= this.drawable;
        }
        Rect rect = drawable.getBounds();
        if (fontMetricsInt != null) {
            Paint.FontMetricsInt fmPaint = paint.getFontMetricsInt();
            int fontHeight = fmPaint.bottom - fmPaint.top;
            int drHeight = rect.bottom - rect.top;
            int top = drHeight / 2 - fontHeight / 4;
            int bottom = drHeight / 2 + fontHeight / 4;
            fontMetricsInt.ascent = -bottom;
            fontMetricsInt.top = -bottom;
            fontMetricsInt.bottom = top;
            fontMetricsInt.descent = top;
        }
        return rect.right;
    }
    @Override
    public void draw(Canvas canvas, CharSequence text, int start, int end, float x, int top, int y, int bottom, Paint paint) {
        Drawable drawable = getDrawable();
        canvas.save();
        int transY = ((bottom - top) - drawable.getBounds().bottom) / 2 + top;
        canvas.translate(x, transY);
        drawable.draw(canvas);
        canvas.restore();
    }
}
```
在geSize方法中通过fontMetricsInt设置从而实现图片和文字居中对齐，其实计算的根本为计算baseline的位置，因为TextView是按照baseline对齐的。

分析getSize方法可以知道这个图片的baseline为图片中央往下fontHeight / 2，这样也就实现了图片和文字的居中对齐。

用法示例：
``` java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    TextView tv = findViewById(R.id.id_tv_framespan);
    Drawable drawable = getResources().getDrawable(R.drawable.ic_launcher);
    drawable.setBounds(0, 0, 50, 50);
    final SpannableString spannableString = new SpannableString(
            "helloworld,helloworld!helloworld,helloworld!helloworld," +
                    "helloworld!helloworld,helloworld!");
    spannableString.setSpan(new VerticalImageSpan(drawable), 0, 1, Spanned.SPAN_INCLUSIVE_INCLUSIVE);
    tv.setText(spannableString);
}

```
### 字体多色渐变
彩虹样的Span，其实实现起来也是很简单的，主要是用到了Paint的Shader技术，效果如下所示：

![](3.jpg)

源代码如下所示：
``` java 
private static class RainbowSpan extends CharacterStyle implements UpdateAppearance {
    private final int[] colors;
    public RainbowSpan(Context context) {
      colors = context.getResources().getIntArray(R.array.rainbow);
    }
    @Override
    public void updateDrawState(TextPaint paint) {
      paint.setStyle(Paint.Style.FILL);
      Shader shader = new LinearGradient(0, 0, 0, paint.getTextSize() * colors.length, colors, null, Shader.TileMode.MIRROR);
      Matrix matrix = new Matrix();
      matrix.setRotate(90);
      shader.setLocalMatrix(matrix);
      paint.setShader(shader);
    }
}
```
由于paint使用shader是从上到下进行绘制，因此这里需要用到矩阵，然后将矩阵旋转90度。

用法示例：
```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    TextView tv = findViewById(R.id.id_tv_framespan);
    final SpannableString spannableString = new SpannableString(
            "helloworld,helloworld!helloworld,helloworld!helloworld," +
                    "helloworld!helloworld,helloworld!");
    spannableString.setSpan(new RainbowSpan(this), 0, 40, Spanned.SPAN_INCLUSIVE_INCLUSIVE);
    tv.setText(spannableString);
}
```

### 字体多色渐变动画效果

![](5.gif)

如果要实现一个动画的彩虹样式，那么该如何实现呢？

其实结合上面的RainbowSpan和AnimateForegroundColorSpan的例子便可以实现AnimatedRainbowSpan。

实现思路：通过ObjectAnimator动画调整RainbowSpan中矩阵的平移，从而实现动画彩虹的效果。

代码如下所示：

首先是AnimatedColorSpan
``` java
private static class AnimatedColorSpan extends CharacterStyle implements UpdateAppearance {
    private final int[] colors;
    private Shader shader = null;
    private Matrix matrix = new Matrix();
    private float translateXPercentage = 0;
    public AnimatedColorSpan(Context context) {
      colors = context.getResources().getIntArray(R.array.rainbow);
    }
    public void setTranslateXPercentage(float percentage) {
      translateXPercentage = percentage;
    }
    public float getTranslateXPercentage() {
      return translateXPercentage;
    }
    @Override
    public void updateDrawState(TextPaint paint) {
      paint.setStyle(Paint.Style.FILL);
      float width = paint.getTextSize() * colors.length;
      if (shader == null) {
        shader = new LinearGradient(0, 0, 0, width, colors, null, Shader.TileMode.MIRROR);
      }
      matrix.reset();
      matrix.setRotate(90);
      matrix.postTranslate(width * translateXPercentage, 0);
      shader.setLocalMatrix(matrix);
      paint.setShader(shader);
    }
  }
}
```
配合属性动画：
``` java 
public class AnimatedRainbowSpanActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_animated_rainbow_span);
        final TextView textView = (TextView) findViewById(R.id.text);
        String text = textView.getText().toString();
        AnimatedColorSpan span = new AnimatedColorSpan(this);
        final SpannableString spannableString = new SpannableString(text);
        String substring = getString(R.string.animated_rainbow_span).toLowerCase();
        int start = text.toLowerCase().indexOf(substring);
        int end = start + substring.length();
        spannableString.setSpan(span, start, end, 0);
        ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(
            span, ANIMATED_COLOR_SPAN_FLOAT_PROPERTY, 0, 100);
        objectAnimator.setEvaluator(new FloatEvaluator());
        objectAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
              textView.setText(spannableString);
            }
        });
        objectAnimator.setInterpolator(new LinearInterpolator());
        objectAnimator.setDuration(DateUtils.MINUTE_IN_MILLIS * 3);
        objectAnimator.setRepeatCount(ValueAnimator.INFINITE);
        objectAnimator.start();
    }
private static final Property<AnimatedColorSpan, Float> ANIMATED_COLOR_SPAN_FLOAT_PROPERTY
      = new Property<AnimatedColorSpan, Float>(Float.class, "ANIMATED_COLOR_SPAN_FLOAT_PROPERTY") {
    @Override
    public void set(AnimatedColorSpan span, Float value) {
        span.setTranslateXPercentage(value);
    }
    @Override
    public Float get(AnimatedColorSpan span) {
        return span.getTranslateXPercentage();
    }
};
```
### 打字效果

![](6.gif)

有了上面的例子，写TypeWriterSpan就变得十分简单了。

先创建TypeWriterSpanGroup
``` java
public class TypeWriterSpanGroup {
    private final float mAlpha;
    private final ArrayList<MutableForegroundColorSpan> mSpans;
    public TypeWriterSpanGroup(float alpha) {
        mAlpha = alpha;
        mSpans = new ArrayList<MutableForegroundColorSpan>();
    }
    public void addSpan(MutableForegroundColorSpan span) {
        span.setAlpha((int) (mAlpha * 255));
        mSpans.add(span);
    }
    public void setAlpha(float alpha) {
        int size = mSpans.size();
        float total = 1.0f * size * alpha;
        for(int index = 0 ; index < size; index++) {
            MutableForegroundColorSpan span = mSpans.get(index);
            if(total >= 1.0f) {
                span.setAlpha(255);
                total -= 1.0f;
            } else {
                span.setAlpha((int) (total * 255));
                total = 0.0f;
            }
        }
    }
    public float getAlpha() {
        return mAlpha;
    }
}
```
然后添加Span与添加动画，整体使用示例如下：
``` java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    final TextView tv = findViewById(R.id.id_tv_framespan);
    String val = "helloworld,helloworld!helloworld,helloworld!helloworld";
    final SpannableString spannableString = new SpannableString(val);
    // 添加Span
    final TypeWriterSpanGroup group = new TypeWriterSpanGroup(0);
    for(int index = 0 ; index <= val.length()-1 ; index++) {
        MutableForegroundColorSpan span = new MutableForegroundColorSpan();
        group.addSpan(span);
        spannableString.setSpan(span, index, index + 1, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
    }
    // 添加动画
    ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(group, TYPE_WRITER_GROUP_ALPHA_PROPERTY, 0.0f, 1.0f);
    objectAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
        @Override
        public void onAnimationUpdate(ValueAnimator animation) {
            //refresh
            tv.setText(spannableString);
        }
    });
    objectAnimator.setDuration(5000);
    objectAnimator.start();
}
```
动画属性变化器代码如下：
``` java
private static final Property<TypeWriterSpanGroup, Float> TYPE_WRITER_GROUP_ALPHA_PROPERTY =
    new Property<TypeWriterSpanGroup, Float>(Float.class, "TYPE_WRITER_GROUP_ALPHA_PROPERTY") {
        @Override
        public void set(TypeWriterSpanGroup spanGroup, Float value) {
            spanGroup.setAlpha(value);
        }
        @Override
        public Float get(TypeWriterSpanGroup spanGroup) {
            return spanGroup.getAlpha();
        }
    };
```
涉及到的类：
``` java
public class MutableForegroundColorSpan extends CharacterStyle implements UpdateAppearance {
    public static final String TAG = "MutableForegroundColorSpan";
    private int mColor = Color.BLACK;
    private int mAlpha = 0 ;
    @Override
    public void updateDrawState(TextPaint tp) {
        tp.setColor(mColor);
        tp.setAlpha(mAlpha);
    }
    public int getColor() {
        return mColor;
    }
    public void setColor(int color) {
        this.mColor = color;
    }
    public void setAlpha(int alpha) {
        mAlpha = alpha;
    }
}
```
