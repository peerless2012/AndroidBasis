# 带有极光推送的项目导出apk的时候出现 ClassCastException
> 项目中集成的有极光推送，debug模式下没有问题，当要导出成apk的时候碰到两种类型的错误

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

 
