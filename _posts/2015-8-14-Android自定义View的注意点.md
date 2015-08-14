---
layout: post
title: Android自定义View的注意点
---

*参考android developer，文章适用于用过一点点自定义view的新手。*

首先讲两个方法：

- invalidate();
- requestLayout();

invalidate()方法用于指示View的绘画已经失效，需要刷新，使用这个方法会调用View内部的draw方法。

> 已经失效一般来说就是你在ondraw里面需要的参数变了，就要使用这个方法

requestLayout()方法用于告诉View，布局已经变了，需要重新measure和layout，使用这个方法会调用View内部的measure和layout方法。
一旦view的相关属性变化一定要调用对应的方法，否则会发生意想不到的错误。
自定义view一定会有ondraw，每个view都会调用这个方法，用到这个方法的时候必然用到一个canvas，一个paint

**画什么由canvas决定**

**view怎么画由paint决定**

ondraw方法的canvas参数有很多draw开头的api，具体不介绍了，下面有段代码：

```java
protectedvoid onDraw(Canvas canvas){
    super.onDraw(canvas);// Draw the shadow
    canvas.drawOval(mShadowBounds,mShadowPaint);// Draw the label text
    canvas.drawText(mData.get(mCurrentItem).mLabel, mTextX, mTextY, mTextPaint);// Draw the pie slices
    for(int i =0; i < mData.size();++i){
        Item it = mData.get(i);
        mPiePaint.setShader(it.mShader);
        canvas.drawArc(mBounds,360- it.mEndAngle,it.mEndAngle - it.mStartAngle,true, mPiePaint);
    }// Draw the pointer
    canvas.drawLine(mTextX, mPointerY, mPointerX, mPointerY, mTextPaint);
    canvas.drawCircle(mPointerX, mPointerY, mPointerSize, mTextPaint);
}
```

有一个很重要的问题，这个view即使画出来了，放哪儿？放多大？
这就需要重载几个方法
如果是想简单处理的，用onsizechanged这个方法就可以
想要更好的控制view的布局，就重载 onMeasure()，如下：

```java
@Override
protectedvoid onMeasure(int widthMeasureSpec,int heightMeasureSpec){
    // Try for a width based on our minimum
    int minw = getPaddingLeft()+ getPaddingRight()+ getSuggestedMinimumWidth();
    // Whatever the width ends up being, ask for a height that would let the pie
    int w = resolveSizeAndState(minw, widthMeasureSpec,1);
    // get as big as it can
    int minh =MeasureSpec.getSize(w)-(int)mTextWidth + getPaddingBottom()+ getPaddingTop();
    int h = resolveSizeAndState(MeasureSpec.getSize(w)-(int)mTextWidth, heightMeasureSpec,0);
    setMeasuredDimension(w, h);
}
```

有几点注意的：

1. padding是属于这个view的，所以要view自己处理
2. resolveSizeAndState()这个方法用来比较这个view计算出来的想要的宽高和onmesure传进来的宽高，进而得出这个view最终的宽高。
3. setMeasuredDimension(w, h);这个方法是必须要调用的，不调用会报运行时错误
