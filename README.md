# AndroidBasis
> Android笔记

## [利用Proguard移除无用代码以及碰到的坑](https://github.com/peerless2012/AndroidBasis/blob/master/%e5%88%a9%e7%94%a8Proguard%e7%a7%bb%e9%99%a4%e6%97%a0%e7%94%a8%e4%bb%a3%e7%a0%81%e4%bb%a5%e5%8f%8a%e7%a2%b0%e5%88%b0%e7%9a%84%e5%9d%91.md)
利用Proguard移除代码中无用code，例如log。

## [Api21以下的LayoutInflater中setFactory2的bug是怎么产生的](https://github.com/peerless2012/AndroidBasis/blob/master/problem/Api21%E4%BB%A5%E4%B8%8B%E7%9A%84LayoutInflater%E4%B8%ADsetFactory2%E7%9A%84bug%E6%98%AF%E6%80%8E%E4%B9%88%E4%BA%A7%E7%94%9F%E7%9A%84.md)
对于Api >=11 并且 < 21,在FrameWork层有一个阻止已经设置Factory2的LayoutInflater的clone对象的Factory2的没有正确合并的bug。

那么这个bug是怎么产生的？又是怎么解决的？

## [背景透明的Dialog](https://github.com/peerless2012/AndroidBasis/blob/master/UI/%E9%80%8F%E6%98%8E%E8%83%8C%E6%99%AFDialog.md)
Android中用style的方式实现背景透明的对话框。

## [初识Realm](https://github.com/peerless2012/AndroidBasis/blob/master/Study/%E5%88%9D%E8%AF%86Realm.md)
__Realm__ 让你能够高效地编写 app 的模型层代码，保证你的数据被安全、快速地存储。它具有 __跨平台__ 、 __简单易用__ ， __可视化__ 的特点。

所以，赶快开始吧！

## [Only the original thread that created a view hierarchy can touch its views. 是怎么产生的](https://github.com/peerless2012/AndroidBasis/blob/master/problem/Only%20the%20original%20thread%20that%20created%20a%20view%20hierarchy%20can%20touch%20its%20views.%20%E6%98%AF%E6%80%8E%E4%B9%88%E4%BA%A7%E7%94%9F%E7%9A%84.md)

我们都知道，在Android里面，只有主线程（`MainThread`）才可以更新ui，比如设置`TextView`的文本内容，`ImageView`的图片等等。

只要我们在非主线程去操作ui界面，就是抛出`"Only the original thread that created a view hierarchy can touch its views."`的异常。那么这个异常时怎么抛出来的呢？

## [带有极光推送的项目导出apk的时候出现 ClassCastException](https://github.com/peerless2012/AndroidBasis/blob/master/problem/%E5%B8%A6%E6%9C%89%E6%9E%81%E5%85%89%E6%8E%A8%E9%80%81%E7%9A%84%E9%A1%B9%E7%9B%AE%E5%AF%BC%E5%87%BAapk%E7%9A%84%E6%97%B6%E5%80%99%E5%87%BA%E7%8E%B0%20ClassCastException.md)
项目中集成的有极光推送，debug模式下没有问题，当要导出成apk的时候碰到两种类型的错误:

* `Warning: android.support.v4.app.AppOpsManagerCompat23: can't find referenced method 'java.lang.Object getSystemService(java.lang.Class)' in class android.content.Context`

* `java.lang.ClassCastException: java.lang.Object cannot be cast to java.lang.String`

## [服务或广播中无法启动设备管理员激活界面的问题](https://github.com/peerless2012/AndroidBasis/blob/master/problem/%E6%9C%8D%E5%8A%A1%E6%88%96%E5%B9%BF%E6%92%AD%E4%B8%AD%E6%97%A0%E6%B3%95%E5%90%AF%E5%8A%A8%E8%AE%BE%E5%A4%87%E7%AE%A1%E7%90%86%E5%91%98%E6%BF%80%E6%B4%BB%E7%95%8C%E9%9D%A2%E7%9A%84%E9%97%AE%E9%A2%98.md)

在`Service`或者`BroadCastReceiver`中启动`Activity`的话需要加上一个`Intent.FLAG_ACTIVITY_NEW_TASK`标记,一般情况下都是好用的，但是有一个需求是在开机广播中启动设置中的设备管理员激活界面,但是发现无法弹出该页面。

## [RadioGroup的onCheckedChanged回调两次的原因](https://github.com/peerless2012/AndroidBasis/blob/master/UI/RadioGroup%E7%9A%84onCheckedChanged%E5%9B%9E%E8%B0%83%E4%B8%A4%E6%AC%A1%E7%9A%84%E5%8E%9F%E5%9B%A0.md)

有的时候我们会使用RadioGroup + RadioButton + Fragment来切换主界面的Fragment,但是我们需要在进入的时候默认选择一个,这个时候就会手动调用RadioGroup的 `public void check(int id)`方法,但是这个时候我们就会发现`public void onCheckedChanged(RadioGroup group, int checkedId);`方法会调用两次.当然这肯定不是我们想要的.



