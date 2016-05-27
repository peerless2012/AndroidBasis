__我的DIalogFragment血泪史：__

__不要跟我提DIalogFragment，乍一看，屏幕旋转时能保存状态，好叼啊！！！实际开发中发现这就是个坑货。__

__一般情况下在Activity或者Fragment的onCreate()或者onResume()中去显示完全没问题__

__但是。。。。。。。。。。。__

__我们其实会在很多地方调用，比如异步任务请求结束以后用对话框的方式提示用户，这种需求是实实在在存在的。但是一旦在请求还没返回，手机就锁屏或者跳转到其他页面，然后数据返回弹出对话框，嘣。。。崩溃了，跟Fragment类似，如下：__

	java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState

__如果是Fragment还好办，commit()方法换成commitAllowStateLoss()，但是DialogFragment碰到就没地方哭去了__

__当然，也不是没有解决方法，比如：__

__异步任务返回的时候判断当前Activity或者Fragment是否在前台并且没有失去焦点再弹出，否则在全局中用变量记录，然后再Activity或者Fragment的onResume()中再弹出。__

__确实可以解决，但是。。。。我懒得用__ 