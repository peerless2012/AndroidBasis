# RoundImageView
最近看一些文章，介绍一个项目的引用库的时候，还会用到[CircleImageView](https://github.com/hdodenhof/CircleImageView),那么如果你只是想要一个RoundImageView（不加其他特效）的话，其实自己完全可以写一个（当然是站在巨人的肩膀上了）。

## 0x00 基础
怎么绘制圆形，这个我就不多说了，网上一大堆，如果你详细看过`Android Support V4`里面都有什么的时候你就会发现它`RoundedBitmapDrawableFactory`,这个就是我们今天的主角。

`RoundedBitmapDrawableFactory`是一个工厂类，有三个方法创建一个`RoundedBitmapDrawable`。
1. `RoundedBitmapDrawableFactory.create(Resources res, Bitmap bitmap)` // 通过`Bitmap`
2. `RoundedBitmapDrawableFactory.create(Resources res, String filepath)` // 通过文件路径
3. `RoundedBitmapDrawableFactory.create(Resources res, java.io.InputStream is)` // 通过输入流

我们一般使用第一种就够了。

`RoundedBitmapDrawable`可以很方便的实现`Circle`或`Round Rect`的`BitmapDrawable`。

里面有两个方法需要注意：
1. `setCircular(boolean circular)` // 设置是否是圆形
2. `setCornerRadius(float cornerRadius)` // 设置圆角矩形圆角大小

__注意:__ 以上两个是互斥的。

## 0x01 实现
我就直接粘代码了：

```java
public class RoundedImageView extends AppCompatImageView{

    private boolean mIsCircular = true;

    private int mCornerRadius = 0;

    public RoundedImageView(Context context) {
        super(context);
    }

    public RoundedImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context, attrs);
    }

    public RoundedImageView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs);
    }

    private void init(Context context, AttributeSet attributeSet) {
        TypedArray typedArray = context.obtainStyledAttributes(attributeSet, R.styleable.RoundedImageView);
        mIsCircular = typedArray.getBoolean(R.styleable.RoundedImageView_isCircular, true);
        mCornerRadius = typedArray.getDimensionPixelSize(R.styleable.RoundedImageView_cornerRadius, 0);
        typedArray.recycle();
    }

    @Override
    public void setImageBitmap(Bitmap bitmap) {
        if (bitmap != null) {
            RoundedBitmapDrawable roundedBitmapDrawable = RoundedBitmapDrawableFactory.create(getResources(), bitmap);
            if (mIsCircular) {
                // 圆形
                roundedBitmapDrawable.setCircular(true);
            } else if (mCornerRadius > 0) {
                // 圆角半径
                roundedBitmapDrawable.setCornerRadius(mCornerRadius);
            }
            super.setImageDrawable(roundedBitmapDrawable);
        } else {
            super.setImageBitmap(bitmap);
        }
    }

    @Override
    public void setImageDrawable(@Nullable Drawable drawable) {
        if (drawable instanceof BitmapDrawable) {
            Bitmap bitmap = ((BitmapDrawable) drawable).getBitmap();
            this.setImageBitmap(bitmap);
        } else {
            super.setImageDrawable(drawable);
        }
    }
}
```

style在这里

```java
<declare-styleable name="RoundedImageView">
    <attr name="isCircular" format="boolean"/>
    <attr name="cornerRadius" format="dimension|reference"/>
</declare-styleable>
```

## 0x02 总结
其实只要重写`setImageDrawable(@Nullable Drawable drawable)`和`setImageBitmap(Bitmap bitmap)`，把`BitmapDrawable`或者`Bitmap`利用`RoundedBitmapDrawableFactory`转换成`RoundedBitmapDrawable`，然后设置给ImageView即可。

__ps:__ 如果你没有引用`AppCompat`，那么直接继承`ImageView`即可。

## 0x03 最后
至此，我们自己用的`RoundedImageView`就完成了，不需要再引用什么库了。

## 0x04 关于
Author peerless2012

Email  [peerless2012@126.con](mailto:peerless2012@126.con)

Blog   [https://peerless2012.github.io](https://peerless2012.github.io)