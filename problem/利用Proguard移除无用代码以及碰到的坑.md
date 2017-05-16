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

## 0x04 更新
通过测试发现，在混淆配置文件中加上下面的代码更保险一点。

	-assumenosideeffects class (your package).LogUtils {
	public static *** d(...);
	public static *** e(...);
	public static *** i(...);
	public static *** v(...);
	public static *** println(...);
	public static *** w(...);
	public static *** wtf(...);
	}

另外附上测试结果：
	
工具类：

	import android.util.Log;

	/**
	 * @Author peerless2012
	 * @Email peerless2012@126.com
	 * @DateTime 2016/11/24 17:10
	 * @Version V1.0
	 * @Description: log工具类
	 * <h1>使用注意</h1>
	 * 1 尽量使用本工具类（系统工具类只支持String类型）
	 * 2 msg字段能不拼接就不拼接，避免自动装箱和拆箱。
	 * 3 TAG最好以常量的方式存在，而不是动态获取。
	 * 4 不用手动调用toString方法。
	 */
	public class LogUtils {
	
	    private static boolean DEBUG = BuildConfig.DEBUG;
	
	    private static String TAG;
	
	    static {
	        int index = BuildConfig.APPLICATION_ID.lastIndexOf(".");
	        TAG = BuildConfig.APPLICATION_ID.substring(index);
	    }
	
	    /*-----------------------  VERBOSE  -------------------------*/
	
	    public static void v(Object msg) {
	        o(Log.VERBOSE, TAG, msg);
	    }
	
	    public static void v(long msg) {
	        l(Log.VERBOSE, TAG, msg);
	    }
	
	    public static void v(float msg) {
	        f(Log.VERBOSE, TAG, msg);
	    }
	
	    public static void v(double msg) {
	        d(Log.VERBOSE, TAG, msg);
	    }
	
	    public static void v(String tag, Object msg) {
	        o(Log.VERBOSE, tag, msg);
	    }
	
	    public static void v(String tag, long msg) {
	        l(Log.VERBOSE, tag, msg);
	    }
	
	    public static void v(String tag, float msg) {
	        f(Log.VERBOSE, tag, msg);
	    }
	
	    public static void v(String tag, double msg) {
	        d(Log.VERBOSE, tag, msg);
	    }
	
	    /*-----------------------  DEBUG  -------------------------*/
	
	    public static void d(Object msg) {
	        o(Log.DEBUG, TAG, msg);
	    }
	
	    public static void d(long msg) {
	        l(Log.DEBUG, TAG, msg);
	    }
	
	    public static void d(float msg) {
	        f(Log.DEBUG, TAG, msg);
	    }
	
	    public static void d(double msg) {
	        d(Log.DEBUG, TAG, msg);
	    }
	
	    public static void d(String tag, Object msg) {
	        o(Log.DEBUG, tag, msg);
	    }
	
	    public static void d(String tag, long msg) {
	        l(Log.DEBUG, tag, msg);
	    }
	
	    public static void d(String tag, float msg) {
	        f(Log.DEBUG, tag, msg);
	    }
	
	    public static void d(String tag, double msg) {
	        d(Log.DEBUG, tag, msg);
	    }
	
	    /*-----------------------  INFO  -------------------------*/
	    public static void i(Object msg) {
	        o(Log.INFO, TAG, msg);
	    }
	
	    public static void i(long msg) {
	        l(Log.INFO, TAG, msg);
	    }
	
	    public static void i(float msg) {
	        f(Log.INFO, TAG, msg);
	    }
	
	    public static void i(double msg) {
	        d(Log.INFO, TAG, msg);
	    }
	
	    public static void i(String tag, Object msg) {
	        o(Log.INFO, tag, msg);
	    }
	
	    public static void i(String tag, long msg) {
	        l(Log.INFO, tag, msg);
	    }
	
	    public static void i(String tag, float msg) {
	        f(Log.INFO, tag, msg);
	    }
	
	    public static void i(String tag, double msg) {
	        d(Log.INFO, tag, msg);
	    }
	
	    /*-----------------------  WARN  -------------------------*/
	
	    public static void w(Object msg) {
	        o(Log.WARN, TAG, msg);
	    }
	
	    public static void w(long msg) {
	        l(Log.WARN, TAG, msg);
	    }
	
	    public static void w(float msg) {
	        f(Log.WARN, TAG, msg);
	    }
	
	    public static void w(double msg) {
	        d(Log.WARN, TAG, msg);
	    }
	
	    public static void w(String tag, Object msg) {
	        o(Log.WARN, tag, msg);
	    }
	
	    public static void w(String tag, long msg) {
	        l(Log.WARN, tag, msg);
	    }
	
	    public static void w(String tag, float msg) {
	        f(Log.WARN, tag, msg);
	    }
	
	    public static void w(String tag, double msg) {
	        d(Log.WARN, tag, msg);
	    }
	
	    /*-----------------------  ERROR  -------------------------*/
	
	    public static void e(Object msg) {
	        o(Log.ERROR, TAG, msg);
	    }
	
	    public static void e(Throwable throwable) {
	        o(Log.ERROR, TAG, Log.getStackTraceString(throwable));
	    }
	
	    public static void e(long msg) {
	        l(Log.ERROR, TAG, msg);
	    }
	
	    public static void e(float msg) {
	        f(Log.ERROR, TAG, msg);
	    }
	
	    public static void e(double msg) {
	        d(Log.ERROR, TAG, msg);
	    }
	
	    public static void e(String tag, Object msg) {
	        o(Log.ERROR, tag, msg);
	    }
	
	    public static void e(String tag, Throwable throwable) {
	        o(Log.ERROR, tag, Log.getStackTraceString(throwable));
	    }
	
	    public static void e(String tag, long msg) {
	        l(Log.ERROR, tag, msg);
	    }
	
	    public static void e(String tag, float msg) {
	        f(Log.ERROR, tag, msg);
	    }
	
	    public static void e(String tag, double msg) {
	        d(Log.ERROR, tag, msg);
	    }
	
	
	
	    private static void o(int type, String tag, Object msg) {
	        if (DEBUG) {
	            if (type == Log.VERBOSE) {
	                Log.v(tag, msg.toString());
	            }else if (type == Log.DEBUG) {
	                Log.d(tag, msg.toString());
	
	            } else if (type == Log.INFO) {
	                Log.i(tag, msg.toString());
	
	            } else if (type == Log.WARN) {
	                Log.w(tag, msg.toString());
	
	            } else if (type == Log.ERROR) {
	                Log.e(tag, msg.toString());
	
	            } else if (type == Log.ASSERT) {
	                Log.wtf(tag, msg.toString());
	            }
	        }
	    }
	    private static void l(int type, String tag, long msg) {
	        if (DEBUG) {
	            if (type == Log.VERBOSE) {
	                Log.v(tag, String.valueOf(msg));
	            }else if (type == Log.DEBUG) {
	                Log.d(tag, String.valueOf(msg));
	
	            } else if (type == Log.INFO) {
	                Log.i(tag, String.valueOf(msg));
	
	            } else if (type == Log.WARN) {
	                Log.w(tag, String.valueOf(msg));
	
	            } else if (type == Log.ERROR) {
	                Log.e(tag, String.valueOf(msg));
	
	            } else if (type == Log.ASSERT) {
	                Log.wtf(tag, String.valueOf(msg));
	            }
	        }
	    }
	    private static void f(int type, String tag, float msg) {
	        if (DEBUG) {
	            if (type == Log.VERBOSE) {
	                Log.v(tag, String.valueOf(msg));
	            }else if (type == Log.DEBUG) {
	                Log.d(tag, String.valueOf(msg));
	
	            } else if (type == Log.INFO) {
	                Log.i(tag, String.valueOf(msg));
	
	            } else if (type == Log.WARN) {
	                Log.w(tag, String.valueOf(msg));
	
	            } else if (type == Log.ERROR) {
	                Log.e(tag, String.valueOf(msg));
	
	            } else if (type == Log.ASSERT) {
	                Log.wtf(tag, String.valueOf(msg));
	            }
	        }
	    }
	    private static void d(int type, String tag, double msg) {
	        if (DEBUG) {
	            if (type == Log.VERBOSE) {
	                Log.v(tag, String.valueOf(msg));
	            }else if (type == Log.DEBUG) {
	                Log.d(tag, String.valueOf(msg));
	
	            } else if (type == Log.INFO) {
	                Log.i(tag, String.valueOf(msg));
	
	            } else if (type == Log.WARN) {
	                Log.w(tag, String.valueOf(msg));
	
	            } else if (type == Log.ERROR) {
	                Log.e(tag, String.valueOf(msg));
	
	            } else if (type == Log.ASSERT) {
	                Log.wtf(tag, String.valueOf(msg));
	            }
	        }
	    }
	}

之所以重载这么多方法是为了避免入参的时候出现类型转换，比如如果只有一个object类型的参数，那么实际上编译后的代码是带有类型转换的，
比如：`public static void v(String tag, Object msg)`，如果调用的时候是`v(TAG,90)`,那么实际上编译后的代码是`v(TAG, String.valueOf(90))`，因此是无法完全去掉日志相关代码的。

#### 测试代码：

	public class App extends Application{
	
	    private static final String TAG = "App";
	
	    @Override
	    public void onCreate() {
	        super.onCreate();
	    }
	
	    @Override
	    protected void attachBaseContext(Context base) {
	        super.attachBaseContext(base);
			LogUtils.i(TAG, base); // 支持
	        LogUtils.i(TAG, "attachBaseContext = " + base); // 不支持
	        LogUtils.i(TAG, "attachBaseContext = " + base.toString()); // 不支持
	    }
	
	    @Override
	    public void onTrimMemory(int level) {
	        super.onTrimMemory(level); // 支持
	        LogUtils.i(TAG, " onTrimMemory "); // 支持
	        LogUtils.i(TAG, level); // 支持
	        LogUtils.i(TAG, level * 1.0f); // 支持
	        LogUtils.i(TAG, level * 1.5); // 支持
	        LogUtils.i(TAG, (byte)level); // 支持
	        try {
	            if (level == ComponentCallbacks2.TRIM_MEMORY_UI_HIDDEN) {
	                throw new RuntimeException();
	            }
	        } catch (Exception e) {
	            LogUtils.e(TAG, e); // 支持
	        }
	        LogUtils.i(TAG, "onTrimMemory" + level); // 支持
	    }
	}

#### 测试结果：

	protected void attachBaseContext(Context context) {
        super.attachBaseContext(context);
        new StringBuilder("attachBaseContext = ").append(context);
        new StringBuilder("attachBaseContext = ").append(context.toString());
    }

	
	public void onTrimMemory(int i) {
        super.onTrimMemory(i);
        if (i == 20) {
            try {
                throw new RuntimeException();
            } catch (Exception e) {
            }
        }
    }

## 0x05 参考资料
* [黑科技:用Proguard的-assumenosideeffects清除log](http://mp.weixin.qq.com/s/DE4gr8cTRQp2jQq3c6wGHQ)
* [how to use -assumenosideeffects class android.util.Log in my app
](http://stackoverflow.com/questions/6408574/how-to-use-assumenosideeffects-class-android-util-log-in-my-app)

## 0x06 关于
Author peerless2012

Email  [peerless2012@126.con](mailto:peerless2012@126.con)

Blog   [https://peerless2012.github.io](https://peerless2012.github.io)
