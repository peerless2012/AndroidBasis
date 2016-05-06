# AndroidBasis
> Android基础总结

## [服务或广播中无法启动设备管理员激活界面的问题](https://github.com/peerless2012/AndroidBasis/blob/master/problem/%E6%9C%8D%E5%8A%A1%E6%88%96%E5%B9%BF%E6%92%AD%E4%B8%AD%E6%97%A0%E6%B3%95%E5%90%AF%E5%8A%A8%E8%AE%BE%E5%A4%87%E7%AE%A1%E7%90%86%E5%91%98%E6%BF%80%E6%B4%BB%E7%95%8C%E9%9D%A2%E7%9A%84%E9%97%AE%E9%A2%98.md)

在`Service`或者`BroadCastReceiver`中启动`Activity`的话需要加上一个`Intent.FLAG_ACTIVITY_NEW_TASK`标记,一般情况下都是好用的，但是有一个需求是在开机广播中启动设置中的设备管理员激活界面,但是发现无法弹出该页面。

## [RadioGroup的onCheckedChanged回调两次的原因](https://github.com/peerless2012/AndroidBasis/blob/master/UI/RadioGroup%E7%9A%84onCheckedChanged%E5%9B%9E%E8%B0%83%E4%B8%A4%E6%AC%A1%E7%9A%84%E5%8E%9F%E5%9B%A0.md)

有的时候我们会使用RadioGroup + RadioButton + Fragment来切换主界面的Fragment,但是我们需要在进入的时候默认选择一个,这个时候就会手动调用RadioGroup的 `public void check(int id)`方法,但是这个时候我们就会发现`public void onCheckedChanged(RadioGroup group, int checkedId);`方法会调用两次.当然这肯定不是我们想要的.



