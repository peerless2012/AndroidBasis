# 带有极光推送的项目导出apk的时候出现 ClassCastException
> 项目中集成的有极光推送，debug模式下没有问题，当要导出成apk的时候碰到两种类型的错误

# 0x00 错误日志

错误日志如下：
	
	Warning: android.support.v4.app.AppOpsManagerCompat23: can't find referenced method 'java.lang.Object getSystemService(java.lang.Class)' in class android.content.Context
	Warning: com.google.gson.jpush.internal.ab: can't find referenced class com.google.gson.jpush.internal.w$com.google.gson.jpush.internal.ac
	Warning: com.google.gson.jpush.internal.w: can't find referenced class com.google.gson.jpush.internal.w$com.google.gson.jpush.internal.y
	Warning: com.google.gson.jpush.internal.w: can't find referenced class com.google.gson.jpush.internal.w$com.google.gson.jpush.internal.aa
	Warning: com.google.gson.jpush.internal.z: can't find referenced class com.google.gson.jpush.internal.w$com.google.gson.jpush.internal.ac
	Warning: library class android.webkit.WebView depends on program class android.net.http.SslCertificate
	Warning: library class android.webkit.WebView depends on program class android.net.http.SslCertificate
	Warning: library class android.webkit.WebViewClient depends on program class android.net.http.SslError
	Warning: library class org.apache.http.conn.ssl.SSLSocketFactory depends on program class org.apache.http.conn.scheme.HostNameResolver
	Warning: library class org.apache.http.conn.ssl.SSLSocketFactory depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams
	Warning: library class org.apache.http.params.HttpConnectionParams depends on program class org.apache.http.params.HttpParams


	java.lang.ClassCastException: java.lang.Object cannot be cast to java.lang.String
	at proguard.obfuscate.MemberObfuscator.newMemberName(MemberObfuscator.java:198)
	at proguard.obfuscate.MemberNameCollector.visitAnyMember(MemberNameCollector.java:74)
	at proguard.classfile.util.SimplifiedVisitor.visitProgramMember(SimplifiedVisitor.java:79)
	at proguard.classfile.util.SimplifiedVisitor.visitProgramMethod(SimplifiedVisitor.java:91)
	at proguard.classfile.visitor.MemberAccessFilter.visitProgramMethod(MemberAccessFilter.java:90)
	at proguard.classfile.ProgramMethod.accept(ProgramMethod.java:71)
	at proguard.classfile.ProgramClass.methodsAccept(ProgramClass.java:504)
	at proguard.classfile.visitor.AllMemberVisitor.visitProgramClass(AllMemberVisitor.java:48)
	at proguard.classfile.ProgramClass.accept(ProgramClass.java:346)
	at proguard.classfile.ProgramClass.hierarchyAccept(ProgramClass.java:359)
	at proguard.classfile.LibraryClass.hierarchyAccept(LibraryClass.java:371)
	at proguard.classfile.LibraryClass.hierarchyAccept(LibraryClass.java:371)
	at proguard.classfile.ProgramClass.hierarchyAccept(ProgramClass.java:416)
	at proguard.classfile.visitor.ClassHierarchyTraveler.visitProgramClass(ClassHierarchyTraveler.java:75)
	at proguard.classfile.visitor.MultiClassVisitor.visitProgramClass(MultiClassVisitor.java:85)
	at proguard.classfile.ProgramClass.accept(ProgramClass.java:346)
	at proguard.classfile.ClassPool.classesAccept(ClassPool.java:116)
	at proguard.obfuscate.Obfuscator.execute(Obfuscator.java:217)
	at proguard.ProGuard.obfuscate(ProGuard.java:333)
	at proguard.ProGuard.execute(ProGuard.java:135)
	at proguard.ProGuard.main(ProGuard.java:492) 

 
## 0x01错误分析
 * 类名: can't find referenced method '方法名' in class 类名
 
 	> 导出apk的时候会对代码进行混淆，有些第三方的jar包已经自带混淆或者不能混淆，导致了此类型错误
 * java.lang.ClassCastException: java.lang.Object cannot be cast to  类型转换错误
 
	> Progard 版本太低导致兼容心问题

## 0x02解决方案

 * 对于第一种错误，需要在混淆的配置文件中添加如下代码：
 		
		# 不警告   
		-dontwarn com.google.**
		# 混淆的时候保留，不进行混淆
		-keep class com.google.gson.** {*;} 
 
 * 对于第二个问题需要使用较高版本（4.x以上proguard，目前最新版是5.2.1）的proguard进行打包，这个在[极光推送的开发者平台](http://docs.jpush.io/guideline/faq/#android)也有介绍。


## 0x03 参考资料
[极光推送的开发者平台#Android集成常见问题](http://docs.jpush.io/guideline/faq/#android)