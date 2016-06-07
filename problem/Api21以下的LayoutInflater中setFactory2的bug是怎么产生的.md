# LayoutInflater在Api 21以下的bug原理分析

## 0x00 背景

最近在做[LayoutInflater&LayoutInflaterCompat的源码分析](https://github.com/peerless2012/SourceAnalysis/blob/master/Android/FrameWork/LayoutInflater&LayoutInflaterCompat%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md) (建议分析本篇文章前先去看LayoutInflater源码分析) ，查看LayoutInflaterCompat的源码的时候发现在LayoutInflaterCompatHC的forceSetFactory2的时候看到这样一个注释：

	/**
     * For APIs >= 11 && < 21, there was a framework bug that prevented a LayoutInflater's
     * Factory2 from being merged properly if set after a cloneInContext from a LayoutInflater
     * that already had a Factory2 registered. We work around that bug here. If we can't we
     * log an error.
     */
    static void forceSetFactory2(LayoutInflater inflater, LayoutInflater.Factory2 factory) {
        //some code ...
    }

注释翻译过来大致是：

对于Api >=11 并且 < 21,在FrameWork层有一个阻止已经设置Factory2的LayoutInflater的clone对象的Factory2的没有正确合并的bug。

说起来比较绕，原谅我英语比较烂。用直白的话来说就是：

对于Api >=11 并且 < 21的Android FrameWork层来说存在着一个bug导致了Factory2不能被正确的合并。这个bug是如果从LayoutInflater clone 一个LayoutInflater，并且被clone的LayoutInflater已经设置了Factory2，那么在给clone出来的LayoutInflater设置Factory2是没有效果的。

那么问题来了，是什么原因导致了Factory2的设置不起作用呢？

## 0x01 原因
通过查看源代码，问题就出现在setFactory2这个方法上。

API 19中的setFactory2方法：

	public void setFactory2(Factory2 factory) {
        if (mFactorySet) {
            throw new IllegalStateException("A factory has already been set on this LayoutInflater");
        }
        if (factory == null) {
            throw new NullPointerException("Given factory can not be null");
        }
        mFactorySet = true;
        if (mFactory == null) {
            mFactory = mFactory2 = factory;
        } else {
            mFactory = new FactoryMerger(factory, factory, mFactory, mFactory2);
        }
	}

API 21中的setFactory2方法：

	public void setFactory2(Factory2 factory) {
        if (mFactorySet) {
            throw new IllegalStateException("A factory has already been set on this LayoutInflater");
        }
        if (factory == null) {
            throw new NullPointerException("Given factory can not be null");
        }
        mFactorySet = true;
        if (mFactory == null) {
            mFactory = mFactory2 = factory;
        } else {
            mFactory = mFactory2 = new FactoryMerger(factory, factory, mFactory, mFactory2);
        }
    }

区别就在最后一行：
	
	// api 19
	mFactory = new FactoryMerger(factory, factory, mFactory, mFactory2);

	// api 21
 	mFactory = mFactory2 = new FactoryMerger(factory, factory, mFactory, mFactory2);

## 0x02 重现流程
1. 对于一个LayoutInflater，先调用它的setFactory方法（其实AppCompatActivity已经这么做了），这时候mFactorySet == false，mFactory == null，所以走mFactory = mFactory2 = factory;这一行，ok，没问题。
2. 调用LayoutInflater的clone对象的时候，调用public abstract LayoutInflater cloneInContext(Context newContext);方法，返回一个当前LayoutInflater的clone对象。这个方法内部是调用了：
	
		/**
	     * Create a new LayoutInflater instance that is a copy of an existing
	     * LayoutInflater, optionally with its Context changed.  For use in
	     * implementing {@link #cloneInContext}.
	     * 
	     * @param original The original LayoutInflater to copy.
	     * @param newContext The new Context to use.
	     */
	    protected LayoutInflater(LayoutInflater original, Context newContext) {
	        mContext = newContext;
	        mFactory = original.mFactory;
	        mFactory2 = original.mFactory2;
	        mPrivateFactory = original.mPrivateFactory;
	        setFilter(original.mFilter);
	    }
	去获取clone对象，这时候我们发现mFactory 和mFactory2 都已经被赋值为被clone的LayoutInflater的Factory，所以clone以后二者都不为空了。
3. 这时候调用clone好的LayoutInflater的setFactory2的方法，mFactorySet == false，mFactory ！= null，导致代码走else的逻辑，也即：

		mFactory = new FactoryMerger(factory, factory, mFactory, mFactory2);

	只有mFactory赋值了，mFactory2没有赋值，又由于mFactorySet已经变为true，所以mFactory2就不能修改了，它一直就是被clone的LayoutInflater中的Factory2。

4. 在LayoutInflater的createViewFromTag方法中：

		View createViewFromTag(View parent, String name, Context context, AttributeSet attrs,
	            boolean ignoreThemeAttr) {
	        // some code
	
	        try {
	            View view;
	            if (mFactory2 != null) {
	                view = mFactory2.onCreateView(parent, name, context, attrs);
	            } else if (mFactory != null) {
	                view = mFactory.onCreateView(name, context, attrs);
	            } else {
	                view = null;
	            }
	
	           // some code
	
	            return view;
	        }
			// some code 
	    }
	
	由于在上述情况下，mFactory2不为空，那么实例化View会一直走第一个判断，而用户想通过clone一个LayoutInflater，并设置Factory2来拦截View的创建一直没有机会执行，clone也白clone。

## 0x03 解决方法

解决方法其实v4中的兼容包已经帮我们做了：

	static void setFactory(LayoutInflater inflater, LayoutInflaterFactory factory) {
        final LayoutInflater.Factory2 factory2 = factory != null
                ? new FactoryWrapperHC(factory) : null;
        inflater.setFactory2(factory2);

        final LayoutInflater.Factory f = inflater.getFactory();
        if (f instanceof LayoutInflater.Factory2) {
            // The merged factory is now set to getFactory(), but not getFactory2() (pre-v21).
            // We will now try and force set the merged factory to mFactory2
            forceSetFactory2(inflater, (LayoutInflater.Factory2) f);
        } else {
            // Else, we will force set the original wrapped Factory2
            forceSetFactory2(inflater, factory2);
        }
    }

    /**
     * For APIs >= 11 && < 21, there was a framework bug that prevented a LayoutInflater's
     * Factory2 from being merged properly if set after a cloneInContext from a LayoutInflater
     * that already had a Factory2 registered. We work around that bug here. If we can't we
     * log an error.
     */
    static void forceSetFactory2(LayoutInflater inflater, LayoutInflater.Factory2 factory) {
        if (!sCheckedField) {
            try {
                sLayoutInflaterFactory2Field = LayoutInflater.class.getDeclaredField("mFactory2");
                sLayoutInflaterFactory2Field.setAccessible(true);
            } catch (NoSuchFieldException e) {
                Log.e(TAG, "forceSetFactory2 Could not find field 'mFactory2' on class "
                        + LayoutInflater.class.getName()
                        + "; inflation may have unexpected results.", e);
            }
            sCheckedField = true;
        }
        if (sLayoutInflaterFactory2Field != null) {
            try {
                sLayoutInflaterFactory2Field.set(inflater, factory);
            } catch (IllegalAccessException e) {
                Log.e(TAG, "forceSetFactory2 could not set the Factory2 on LayoutInflater "
                        + inflater + "; inflation may have unexpected results.", e);
            }
        }
    }

先设置Factory2，然后获取Factory

* 如果f是Factory2类型，则把mFactory2也强制赋值为f。通clone的LayoutInflater获取的LayoutInflater，setFactory2方法只把Factory2设置给了mFactory，这个操作就是把Factory2也设置给mFactory2.
* 如果不是Factory2类型，则把新的mFactory2设置给LayoutInflater。

## 0x04 小尾巴

<font color="#FF0000" size=5>如果你觉得对你有用，还请不<b>吝戳一下右上角</b>，你的鼓励就是我的动力！666</font>

## 0x05 关于

Author peerless2012

Email  [peerless2012@126.con](mailto:peerless2012@126.con)

Blog   [peerless2012.github.io](https://peerless2012.github.io)