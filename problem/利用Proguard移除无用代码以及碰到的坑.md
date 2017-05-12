## 利用Proguard移除无用代码以及碰到的坑
## 0x00 起因
今天用反编译工具反编译了下App，如图：

![截图](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/proguard_pre.png)
可以看到，里面用来打印生命周期的代买还在里面，虽然说我们一般会封装成log的工具类，然后利用`BuildConfig.DEBUG`来判断是否打印日志，但是这种也太low了是吧？毕竟对于有洁癖的人来说还是多了个空方法，看着不爽。
## 0x01 解决过程
由于之前没看到过相关资料，因此只能搜索，历经切换了多次关键字后终于发现了Proguard中`-assumenosideeffects`这个参数，通过配置可以在`release`中移除某些代码。

具体配置是：
* 必须是Release模式并且`minifyEnabled`置为`true`，这样才会使用`Proguard`混淆
* 在本Module的混淆配置文件`proguard-rules.pro`中增加下面的代码：
	
		-assumenosideeffects class android.util.Log {     
		public static *** d(...);     
		public static *** e(...);     
		public static *** i(...);     
		public static *** v(...);     
		public static *** println(...);     
		public static *** w(...);     
		public static *** wtf(...);
		}
* 确保混淆配置文件中没有关闭压缩，也就是说不要有`-dontoptimize`这个配置

你以为到这里就结束了？呵呵，TYTN！
我按照这个打包了好多次，就是没有效果，三个关键点比对了好多次，就是没效果。后来，终于找到了原因，原来`buildTypes`中的`release`中是这样写的：
	
	release {
            minifyEnabled true
            zipAlignEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
    }

__注意__,问题就出在`getDefaultProguardFile('proguard-android.txt')`这里，这里的`proguard-android.txt`指向的是`SDK`中`sdk_dir/tools/proguard/proguard-android.txt`,这里面有一句：
![截图](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/proguard_default_config.png)

问题就在这里，去掉这里或者直接在`buildTypes`移除缺省的配置（前提是保证自定义的配置是完整的满足混淆的）即可。

## 0x02 最后
关于`-assumenosideeffects`这个参数如何能移除代码、能移除哪些类型的可参考下面的第一篇资料。
最后奉上处理后的代码：
![截图](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/proguard_after.png)
是不是干净多了？

## 0x03 补充
上面最终结果的图中，圈出来了两个地方：
* 第一个，由于开启代码优化，因此`Proguard`就把代码中的一个静态方法直接内联到调用的地方了（类似C++的内联函数）
* 有些log里面的变量是没法去掉的，因此会多一个无用的字符串，具体参考第一个参考资料。

祝大家愉快！！！

## 0x04 参考资料
* [黑科技:用Proguard的-assumenosideeffects清除log](http://mp.weixin.qq.com/s/DE4gr8cTRQp2jQq3c6wGHQ)
* [how to use -assumenosideeffects class android.util.Log in my app
](http://stackoverflow.com/questions/6408574/how-to-use-assumenosideeffects-class-android-util-log-in-my-app)

## 0x05 关于
Author peerless2012

Email  [peerless2012@126.con](mailto:peerless2012@126.con)

Blog   [https://peerless2012.github.io](https://peerless2012.github.io)
