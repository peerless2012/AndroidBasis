## 现象
有的时候我们会使用RadioGroup + RadioButton + Fragment来切换主界面的Fragment,但是我们需要在进入的时候默认选择一个,这个时候就会手动调用RadioGroup的 `public void check(int id)`方法,但是这个时候我们就会发现`public void onCheckedChanged(RadioGroup group, int checkedId);`方法会调用两次.当然这肯定不是我们想要的.

## 代码分析
我们先来看一下<code>public void check(int id)</code>里面做了什么操作:

	public void check(int id) {
        // don't even bother
        if (id != -1 && (id == mCheckedId)) {
            return;
        }

        if (mCheckedId != -1) {
            setCheckedStateForView(mCheckedId, false);
        }

        if (id != -1) {
            setCheckedStateForView(id, true);
        }

        setCheckedId(id);
    }

这里面主要有两个方法:

	private void setCheckedId(int id) {
        mCheckedId = id;
        if (mOnCheckedChangeListener != null) {
            mOnCheckedChangeListener.onCheckedChanged(this, mCheckedId);
        }
    }

	private void setCheckedStateForView(int viewId, boolean checked) {
        View checkedView = findViewById(viewId);
        if (checkedView != null && checkedView instanceof RadioButton) {
            ((RadioButton) checkedView).setChecked(checked);
        }
    }

首次调用的时候`private int mCheckedId = -1;`所以`setCheckedId(int id)`和`setCheckedStateForView(int viewId, boolean checked)`这个方法都会调用.

第一个方法很好理解,回调的就是我们监听的方法,这里会产生一次`onCheckedChanged(RadioGroup group, int checkedId)`.

我们重点来看第二个方法,在这里是找到了要选中的RadioButton,然后调用RadioButton身上的`public void setChecked(boolean checked)`方法,这个方法的源代码见下方:

	public void setChecked(boolean checked) {
        if (mChecked != checked) {
            mChecked = checked;
            refreshDrawableState();
            notifyViewAccessibilityStateChangedIfNeeded(
                    AccessibilityEvent.CONTENT_CHANGE_TYPE_UNDEFINED);

            // Avoid infinite recursions if setChecked() is called from a listener
            if (mBroadcasting) {
                return;
            }

            mBroadcasting = true;
            if (mOnCheckedChangeListener != null) {
                mOnCheckedChangeListener.onCheckedChanged(this, mChecked);
            }
            if (mOnCheckedChangeWidgetListener != null) {
                mOnCheckedChangeWidgetListener.onCheckedChanged(this, mChecked);
            }

            mBroadcasting = false;            
        }
    }

这里面需要关注两个监听,一个是`mOnCheckedChangeListener` 一个是 `mOnCheckedChangeWidgetListener`,`mOnCheckedChangeListener`.

1. 如果用户调用了RadioButton的 `public void setOnCheckedChangeListener(OnCheckedChangeListener listener)` 方法,`mOnCheckedChangeWidgetListener`就不会为空,会回调到用户设置的监听.
2. `mOnCheckedChangeWidgetListener`这个一般不是由用户设置的,是由RadioGroup设置的.
	

		private int mCheckedId = -1;
	    // tracks children radio buttons checked state
	    private CompoundButton.OnCheckedChangeListener mChildOnCheckedChangeListener;		
		private class CheckedStateTracker implements CompoundButton.OnCheckedChangeListener {
	    public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
	            // prevents from infinite recursion
	            if (mProtectFromCheckedChange) {
	                return;
	            }
	
	            mProtectFromCheckedChange = true;
	            if (mCheckedId != -1) {
	                setCheckedStateForView(mCheckedId, false);
	            }
	            mProtectFromCheckedChange = false;
	
	            int id = buttonView.getId();
				//RadioButton回调的
	            setCheckedId(id);
	        }
    	}

		public void onChildViewAdded(View parent, View child) {
            if (parent == RadioGroup.this && child instanceof RadioButton) {
                int id = child.getId();
                // generates an id if it's missing
                if (id == View.NO_ID) {
                    id = View.generateViewId();
                    child.setId(id);
                }
				//给RadioButton设置监听回调
                ((RadioButton) child).setOnCheckedChangeWidgetListener(
                        mChildOnCheckedChangeListener);
            }

            if (mOnHierarchyChangeListener != null) {
                mOnHierarchyChangeListener.onChildViewAdded(parent, child);
            }
        }

        /**
         * {@inheritDoc}
         */
        public void onChildViewRemoved(View parent, View child) {
            if (parent == RadioGroup.this && child instanceof RadioButton) {
                ((RadioButton) child).setOnCheckedChangeWidgetListener(null);
            }

            if (mOnHierarchyChangeListener != null) {
                mOnHierarchyChangeListener.onChildViewRemoved(parent, child);
            }
        }

	在RadioGroup中调用RadioButton的`setChecked(true)`方法,RadioButton的`setChecked(true)`方法又回调了RadioGroup的`CheckedStateTracker`的`public void onCheckedChanged(CompoundButton buttonView, boolean isChecked)`,这个里面又会调用 `setCheckedId(id);` 在这里面就回调到我们给RadioGroup设置的`setOnCheckedChangeListener(this)`,至此产生了一次回调.

分析到这里,`public void check(int id)`这个方法里面 `setCheckedStateForView(id, true);` 和 `setCheckedId(id);` 各产生了一个回调,因此共计产生两次回调.

## 解决
如果想选中某个RadioButton,直接找到这个RadioButton,这是它的 `setChecked(true)` 即可