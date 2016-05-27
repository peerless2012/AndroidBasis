# 透明背景Dialog

## 背景
Android中对话框主要用到Dialog和AlertDialog两种。

__Dialog：__

![普通Dialog](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/dialog_normal.png)

__AlertDialog：__

![普通AlertDialog](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/alertdialog_normal.png)

Dialog和Alert可以实现类似的效果，不过从给出的接口来看：

* Dialog的title可以直接设置，但是内容区域的话需要传入Dialog要显示的View或者View的布局。
* AlertDialog则封装的更全面一些，在不需要我们手动设置View的情况下就可以展示列表，文本等多种类型（通过查看源码中它的布局结构可以发现，它其实就是在布局中定义了很多种类型布局，然后动态显示和隐藏）

总得来说，Dialog的功能比较简单，功能少。而AlertDialog是对Dialog的扩充，功能比较完善。

#### 现在先来说一下需求：

有的时候产品需要在进入界面的时候在内容区域上展示一个悬浮的View，然后点击按钮关闭（注： 背景不会变灰，且能看到下面的内容），这个时候我们可以：

* 用布局，相对布局或者帧布局，下面放内容，上面放突出显示的内容。然后点击隐藏。

	>优点是简单。
	>
	>缺点是通用性差，要增加或者取消这个操作起来比较麻烦。

* 用Dialog，在Dialog中显示内容。
	> 优点是只要实现了，在任何地方都可以方便使用。
	> 
	> 缺点是需要自己去定义。

## 尝试解决
我们在这里就不用布局这么low的方式去做了，我们用Dialog的方式去实现，这样就可以在很多地方灵活应用。

### AlertDialog
首先想到的就是AlertDialog。因为他封装的比较好，提供的方法也比较多，如果用他实现了，我们会方便很多。

自定义Widget的属性，我们一般从style入手：

	<style name="TransAlertDialog" parent="@android:style/Theme.Holo.Dialog">
        <item name="android:windowBackground">@android:color/transparent</item><!-- 内容区域背景透明 -->
         <item name="android:backgroundDimEnabled">false</item><!-- 背景不模糊 -->
    </style>

然后在创建AlertDialog的时候传入这个style，然后结果却令人失望。

![透明背景AlertDialog](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/alertdialog_trans_dark.png)

那么换成Holo.Light主题呢？

	<style name="TransAlertDialog" parent="@android:style/Theme.Holo.Light.Dialog">
        <item name="android:windowBackground">@android:color/transparent</item><!-- 内容区域背景透明 -->
         <item name="android:backgroundDimEnabled">false</item><!-- 背景不模糊 -->
    </style>

![透明背景AlertDialog](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/alertdialog_trans_light.png)


黑色或白色的背景有木有，即使自己写个透明布局然后设置进去也无济于事，因为：

* 黑色或者白色是AlertDialog内部ContentLayout去控制的，外部没有方法去修改。
* 内容区域的背景还是一个带边缘阴影效果，自己的布局是无法完全遮挡住的。

翻了一段时间源码，真没发现哪里可以设置，只能依依不舍的放弃了。

### Dialog
既然AlertDialog无法实现，那就试试Dialog吧。主题样式如下：

	<style name="TransDialog" parent="@android:style/Theme.Dialog">
        <item name="android:windowBackground">@android:color/transparent</item><!-- 内容区域背景透明 -->
         <item name="android:backgroundDimEnabled">false</item><!--背景不模糊-->
    </style>

在创建Dialog的时候传入这个样式，结果是：

![](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/dialog_trans.png)

解决了！！！2333

## 结果
根据上述尝试，如果想要一个窗口背景和内容区域背景透明的Dialog，只能用Dialog + 透明style来实现了。

## 写在最后的

Android显示对话框还有一个DialogFragment，对于它我有段血泪史，稍后再说。

## 关于
Author peerless2012

Email  [peerless2012@126.con](mailto:peerless2012@126.con)

Blog   [https://peerless2012.github.io](https://peerless2012.github.io)