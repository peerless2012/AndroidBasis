# Only the original thread that created a view hierarchy can touch its views. 是怎么产生的
***
## 0x00 现象
我们都知道，在Android里面，只有主线程（`MainThread`）才可以更新ui，比如设置`TextView`的文本内容，`ImageView`的图片等等。

只要我们在非主线程去操作ui界面，就是抛出`"Only the original thread that created a view hierarchy can touch its views."`的异常。那么这个异常时怎么抛出来的呢？

## 0x01 重现
* 复现代码 1

	textView = (TextView) findViewById(R.id.tv);
		new Thread(new Runnable() {
			public void run() {
				SystemClock.sleep(3000);
				textView.setText("我是文本来自子线程");
			}
		}).start();

	异常信息

		1	05-10 21:40:50.664: E/AndroidRuntime(6817): FATAL EXCEPTION: Thread-3133
		2	05-10 21:40:50.664: E/AndroidRuntime(6817): Process: com.peerless2012.androidtest, PID: 6817
		3	05-10 21:40:50.664: E/AndroidRuntime(6817): android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
		4	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.view.ViewRootImpl.checkThread(ViewRootImpl.java:6363)
		5	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.view.ViewRootImpl.invalidateChildInParent(ViewRootImpl.java:909)
		6	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.view.ViewGroup.invalidateChild(ViewGroup.java:4691)
		7	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.view.View.invalidateInternal(View.java:11802)
		8	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.view.View.invalidate(View.java:11766)
		9	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.view.View.invalidate(View.java:11750)
		10	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.widget.TextView.checkForRelayout(TextView.java:6853)
		11	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.widget.TextView.setText(TextView.java:4060)
		12	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.widget.TextView.setText(TextView.java:3918)
		13	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at android.widget.TextView.setText(TextView.java:3893)
		14	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at compeerless2012.androidtest.MainActivity$1.run(MainActivity.java:41)
		15	05-10 21:40:50.664: E/AndroidRuntime(6817): 	at java.lang.Thread.run(Thread.java:818)

* 复现代码 2

		textView = (TextView) findViewById(R.id.tv);
		new Thread(new Runnable() {
			public void run() {
				SystemClock.sleep(3000);
				textView.invalidate();
			}
		}).start();

	异常信息

		1	05-10 21:47:30.869: E/AndroidRuntime(8658): FATAL EXCEPTION: Thread-3168
		2	05-10 21:47:30.869: E/AndroidRuntime(8658): Process: com.peerless2012.androidtest, PID: 8658
		3	05-10 21:47:30.869: E/AndroidRuntime(8658): android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
		4	05-10 21:47:30.869: E/AndroidRuntime(8658): 	at android.view.ViewRootImpl.checkThread(ViewRootImpl.java:6363)
		5	05-10 21:47:30.869: E/AndroidRuntime(8658): 	at android.view.ViewRootImpl.invalidateChildInParent(ViewRootImpl.java:909)
		6	05-10 21:47:30.869: E/AndroidRuntime(8658): 	at android.view.ViewGroup.invalidateChild(ViewGroup.java:4691)
		7	05-10 21:47:30.869: E/AndroidRuntime(8658): 	at android.view.View.invalidateInternal(View.java:11802)
		8	05-10 21:47:30.869: E/AndroidRuntime(8658): 	at android.view.View.invalidate(View.java:11766)
		9	05-10 21:47:30.869: E/AndroidRuntime(8658): 	at android.view.View.invalidate(View.java:11750)
		10	05-10 21:47:30.869: E/AndroidRuntime(8658): 	at com.peerless2012.androidtest.MainActivity$1.run(MainActivity.java:41)
		11	05-10 21:47:30.869: E/AndroidRuntime(8658): 	at java.lang.Thread.run(Thread.java:818)

## 0x02 异常分析
通过对比我们不难发现：

* 不管是在子线程给`TextView`设置文本还是调用`TextView`的`invalid()`方法都会导致上述异常。
* 并且通过对比异常日志，导致异常的有共有的函数，第一种情况异常信息 2-9 行和第二种情况异常   2-9 行是一模一样的。

通过上述情况不难看出，导致异常的原因并不在给`TextView`设置文本或者给`ImageView`设置图片，真正的原因是调用这些方法之后都会导致View的重绘，然后重绘的时候系统回去检测当前线程，如果是非主线程，则抛出异常，那么具体代码里面是怎么提现的呢？

## 0x03 代码分析

我们先来看`TextView`设置文本的关键代码：

	private Layout mLayout;

	@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //a lot of code
        if (mLayout == null) {
            makeNewLayout(want, hintWant, boring, hintBoring,width - getCompoundPaddingLeft() - getCompoundPaddingRight(), false);
        } else {
            //some code
        }
    }
	
	protected void makeNewLayout(int wantWidth, int hintWidth,
                                 BoringLayout.Metrics boring,
                                 BoringLayout.Metrics hintBoring,
                                 int ellipsisWidth, boolean bringIntoView) {
        //a lot of code
        mLayout = makeSingleLayout(wantWidth, boring, ellipsisWidth, alignment, shouldEllipsize,
                effectiveEllipsize, effectiveEllipsize == mEllipsize);
        //a lot of code
    }
	private void setText(CharSequence text, BufferType type,boolean notifyBefore, int oldlen) {
        // a lot of code
        if (mLayout != null) {
            checkForRelayout();
        }
        //some code
    }
	
	private void checkForRelayout() {
        if ((mLayoutParams.width != LayoutParams.WRAP_CONTENT ||
                (mMaxWidthMode == mMinWidthMode && mMaxWidth == mMinWidth)) &&
                (mHint == null || mHintLayout != null) &&
                (mRight - mLeft - getCompoundPaddingLeft() - getCompoundPaddingRight() > 0)) {
            
            makeNewLayout(want, hintWant, UNKNOWN_BORING, UNKNOWN_BORING,
                          mRight - mLeft - getCompoundPaddingLeft() - getCompoundPaddingRight(),
                          false);

            if (mEllipsize != TextUtils.TruncateAt.MARQUEE) {
                if (mLayoutParams.height != LayoutParams.WRAP_CONTENT &&
                    mLayoutParams.height != LayoutParams.MATCH_PARENT) {
                    invalidate();
                    return;
                }

                if (mLayout.getHeight() == oldht &&
                    (mHintLayout == null || mHintLayout.getHeight() == oldht)) {
                    invalidate();
                    return;
                }
            }
            requestLayout();
            invalidate();
        } else {
            nullLayouts();
            requestLayout();
            invalidate();
        }
    }

我们跟着代码看一下逻辑

* `TextView`里面有一个变量`mLayout`,当TextView需要被显示出来的时候会调用`onMeasure(int widthMeasureSpec, int heightMeasureSpec)`方法，这时候`mLayout`肯定是空，所以会调用`makeNewLayout()`方法给`mLayout`赋值，这以后`mLayout`一般就不会为空了。
* 当我们给TextView设置文本内容的时候，会判断`mLayout`是否为空，如果不会空，则会调用`checkForRelayout()`方法。
* 大致看一下`checkForRelayout()`这个方法我们会发现，`invalidate()`这个方法无路如何都会执行。

那么我们的第一个问题已经得到证实，也就是说虽然非主线程操作ui的方式不尽相同，但是最终导致异常的原因都是一样的，那就是`invalidate()`这个方法。


我们再来看一下`View`和`ViewGroup`里面关于`invalidate()`都做了那些操作。

View.java

	void assignParent(ViewParent parent) {
        if (mParent == null) {
            mParent = parent;
        } else if (parent == null) {
            mParent = null;
        } else {
            throw new RuntimeException("view " + this + " being added, but"
                    + " it already has a parent");
        }
    }

	public void invalidate() {
        invalidate(true);
    }

    void invalidate(boolean invalidateCache) {
        invalidateInternal(0, 0, mRight - mLeft, mBottom - mTop, invalidateCache, true);
    }

    void invalidateInternal(int l, int t, int r, int b, boolean invalidateCache,
            boolean fullInvalidate) {
        //...
        if (...) {
            //...
			final AttachInfo ai = mAttachInfo;
            final ViewParent p = mParent;
            if (p != null && ai != null && l < r && t < b) {
                //...
                p.invalidateChild(this, damage);
            }
            //...
        }
    }

* 当`View`被`add`到`ViewGroup`之后，`ViewGroup`会把自身传递给子`View`，然后赋值给子`View`的`mParent`.
* 当`View`或者`ViewGroup`调用`invalidate()`方法以后，实际上最终调用的是`invalidateInternal()`这个方法。
* 在`invalidateInternal()`里面会调用父`View`的`invalidateChild(View child, final Rect dirty)`方法。

ViewGroup.java

	//addView方法
	public void addView(View child) {
        addView(child, -1);
    }

    public void addView(View child, int index) {
        //...
        addView(child, index, params);
    }

    public void addView(View child, int width, int height) {
        //...
        addView(child, -1, params);
    }

    public void addView(View child, LayoutParams params) {
        addView(child, -1, params);
    }

    public void addView(View child, int index, LayoutParams params) {
        //...
        requestLayout();
        invalidate(true);
        addViewInner(child, index, params, false);
    }

	private void addViewInner(View child, int index, LayoutParams params,
            boolean preventRequestLayout) {
		//...
        if (preventRequestLayout) {
			//把当前ViewGroup当作parent赋给子View
            child.assignParent(this);
        } else {
			//把当前ViewGroup当作parent赋给子View
            child.mParent = this;
        }
		//...
        AttachInfo ai = mAttachInfo;
        if (...) {
            //...
            child.dispatchAttachedToWindow(mAttachInfo, (mViewFlags&VISIBILITY_MASK));
            //...
        }
		//...
    }

	
	//invalidate方法
	public final void invalidateChild(View child, final Rect dirty) {
        ViewParent parent = this;
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
			//a lot of code
            do {
                View view = null;
                if (parent instanceof View) {
                    view = (View) parent;
                }
                //...
                parent = parent.invalidateChildInParent(location, dirty);
                //...
            } while (parent != null);
        }
    }

	public ViewParent invalidateChildInParent(final int[] location, final Rect dirty) {
        if ((mPrivateFlags & PFLAG_DRAWN) == PFLAG_DRAWN ||
                (mPrivateFlags & PFLAG_DRAWING_CACHE_VALID) == PFLAG_DRAWING_CACHE_VALID) {
            if ((mGroupFlags & (FLAG_OPTIMIZE_INVALIDATE | FLAG_ANIMATION_DONE)) !=
                        FLAG_OPTIMIZE_INVALIDATE) {
                //...
                return mParent;
            } else {
                //...
                return mParent;
            }
        }

        return null;
    }

* 当需要添加子View的时候会调用方法`addView(View v)`方法，该方法最终调用的是`				addViewInner(View child, int index, LayoutParams params,
            boolean preventRequestLayout)`，在这个方法里会把当前`ViewGroup`赋值给子`View`的`mParent`，并且会把`AttachInfo`也赋值给子`View`。
* 当View（或者是ViewGroup）调用`invalidate()`方法的时候，会调用父View的`invalidateChild(View child, final Rect dirty)`，这里面有一个`do while`循环，每循环一次会调用`parent = parent.invalidateChildInParent(location, dirty)`,这个方法的意思就是调用父`View`的`invalidateChildInParent`方法，然后把父`View`中的`mParent`返回，下一次就调用当前`View`父`View`的父`View`的`invalidateChildInParent`方法，直到到最外面的`View`节点。
* 最终会遍历到根节点，也就是ViewRoot，调用ViewRoot的`invalidateChildInParent`方法，实际是调用`ViewRoot`的实现类`ViewRootImpl`身上的`invalidateChildInParent`方法。

ViewRootImpl.java

	final Thread mThread;
	
	public ViewRootImpl(Context context, Display display) {
        mThread = Thread.currentThread();
        //...
    }
	
	@Override
    public void invalidateChild(View child, Rect dirty) {
        invalidateChildInParent(null, dirty);
    }

    @Override
    public ViewParent invalidateChildInParent(int[] location, Rect dirty) {
        checkThread();
        //...
    }
	
	void checkThread() {
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }


* `ViewRootImpl`创建的时候，创建它的`Thread`会赋值给`mThread`，当然肯定是主线程创建的。
* 当`ViewGroup`中的`invalidateChild(View child, final Rect dirty)`方法循环到最外层的时候，这个`mParent`就是`ViewRootImpl`。
* 当调用到`ViewRootImpl`的`invalidateChildInParent(int[] location, Rect dirty)`方法的时候会去检测线程，也就是`checkThread()`。
* `checkThread()`里面会判断当前线程是不是主线程，如果不是的就抛出异常。

至此，分析结束。

## 0x04 一个有趣的现象
如果在TextView被初始化的时候，开启子线程去操作ui界面，代码如下：

	textView = (TextView) findViewById(R.id.tv);
		new Thread(new Runnable() {
			public void run() {
				textView.setText("我是文本来自子线程");
			}
		}).start();

是没有任何问题的，为什么呢？

其实原理也很简单，在这个时候，整个界面还没显示，那么`TextView`的`onMeasure`方法还没有执行，那么`mLayout`也就是空，在`setText`方法里已经有判断了，如果`mLayout`为空，则不会执行`checkForRelayout()`，当然也就不会崩溃了。

这时候你可能会发现，设置文本被`mLayout==null`绕过了，那么我直接调用`invalidate()`方法呢？代码如下：

	textView = (TextView) findViewById(R.id.tv);
		new Thread(new Runnable() {
			public void run() {
				textView.invalidate();
			}
		}).start();

这个也是不会崩溃的，不过目前还没找到代码作为依据，如果有知道的小伙伴，可以[@](mailto:peerless2012@126.com)我。

时间比较紧张，如有遗漏，还请不吝指教。