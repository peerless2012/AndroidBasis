# 初识Realm
> Realm Java 让你能够高效地编写 app 的模型层代码，保证你的数据被安全、快速地存储。

## 0x00 简介
Realm 是一个跨平台的移动数据库引擎，于 2014 年 7 月发布，准确来说，它是专门为移动应用所设计的数据持久化解决方案之一。

Realm 可以轻松地移植到您的项目当中，并且绝大部分常用的功能（比如说插入、查询等等）都可以用一行简单的代码轻松完成！

Realm 并不是对 Core Data 的简单封装，相反地， Realm 并不是基于 Core Data ，也不是基于 SQLite 所构建的。它拥有自己的数据库存储引擎，可以高效且快速地完成数据库的构建操作。

之前我们提到过，由于 Realm 使用的是自己的引擎，因此， Realm 就可以在 iOS 和 Android 平台上共同使用（完全无缝），并且支持 Swift 、 Objective-C 以及 Java 语言来编写（ Android 平台和 iOS 平台使用不同的 SDK ）。

数以万计的使用 Realm 的开发者都会发现，使用 Realm 比使用 SQLite 以及 Core Data 要快很多。

特点：

* __跨平台__ ：现在绝大多数的应用开发并不仅仅只在 iOS 平台上进行开发，还要兼顾到 Android 平台的开发。为两个平台设计不同的数据库是愚蠢的，而使用 Realm 数据库， iOS 和 Android 无需考虑内部数据的架构，调用 Realm 提供的 API 就可以完成数据的交换，实现 “ 一个数据库，两个平台无缝衔接 ” 。
* __简单易用__ ： Core Data 和 SQLite 冗余、繁杂的知识和代码足以吓退绝大多数刚入门的开发者，而换用 Realm ，则可以极大地减少学习代价和学习时间，让应用及早用上数据存储功能。
* __可视化__ ： Realm 还提供了一个轻量级的数据库查看工具，借助这个工具，开发者可以查看数据库当中的内容，执行简单的插入和删除数据的操作。毕竟，很多时候，开发者使用数据库的理由是因为要提供一些所谓的 “ 知识库 ” 。

## 0x01 集成
### 0 前提：

1. 目前我们还不支持 Android 以外的 Java 环境；
2. Android Studio >= 1.5.1 ；
3. 较新的 Android SDK 版本；
4. JDK 版本 >=7；
5. 我们支持 Android API 9 以上的所有版本（Android 2.3 Gingerbread 及以上）。

### 1 集成步骤

1. 添加下面的class path依赖到你工程级别的build.gradle文件中
	
		buildscript {
		    repositories {
		        jcenter()
		    }
		    dependencies {
		        classpath "io.realm:realm-gradle-plugin:1.0.0"
		    }
		}

	工程级别build.gradle文件在这个位置：

	![工程级别build.gradle的路径](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/project-level-build-gradle.png)

2. 应用 `realm-android` 插件到应用程序（也就是Module）级别 build.gradle 文件中。
	
		apply plugin: 'realm-android'
	
	应用程序（Module）级别 build.gradle 文件在这个位置：
	
	![应用级别build.gradle的路径](https://raw.githubusercontent.com/peerless2012/AndroidBasis/master/Imgs/application-level-build-gradle.png)

### 2 使用步骤
1. 在Module增加依赖依赖：
	
		compile 'io.realm:android-adapters:1.2.1'

2. 在工程中配置 Realm （一般在Application中配置即可）
	
		RealmConfiguration realmConfiguration = new RealmConfiguration.Builder(this)
                .name("test.realm")// 存储文件名称，类似db文件名
                .migration(new RealmMigration() { // 当本地已经存在的数据版本跟当前运行的不一致会调用此方法
                    @Override
                    public void migrate(DynamicRealm realm, long oldVersion, long newVersion) {

                    }
                })
                .build();
        Realm mRealm = Realm.getInstance(realmConfiguration);// 设置配置

3. 定义model类

	__注意__  __这个类需要是单独的类，而不能是内部类。__

		@RealmClass
		public class People extends RealmObject {

    		@PrimaryKey
		    private int id;
		
		    private String name;
		
		    private int age;
		
		    public int getId() {
		        return id;
		    }
		
		    public void setId(int id) {
		        this.id = id;
		    }
		
		    public String getName() {
		        return name;
		    }
		
		    public void setName(String name) {
		        this.name = name;
		    }
		
		    public int getAge() {
		        return age;
		    }
		
		    public void setAge(int age) {
		        this.age = age;
		    }
		
		    @Override
		    public String toString() {
		        return "People{" +
		                "id=" + id +
		                ", name='" + name + '\'' +
		                ", age=" + age +
		                '}';
		    }
		}
	
4. 使用

		People people = new People();
        people.setName("名字 ： "+(0 + offset));
        people.setAge(0+offset);
        mRealm.beginTransaction();
        mRealm.copyToRealm(people);
        mRealm.commitTransaction();

详情参照官方文档

## 0x02 可能碰到的问题
1. the xxx RealmMigration must be provided
	
	出现这个问题的时候是因为，我刚开始的时候没有在设置Realm的class path和应用 realm-android 插件的情况下使用了Realm的功能，增加上述以后的版本跟之前的可能不一样导致的。
	
	解决方法是卸载app，重新安装。

## 0x03 参考资料
[官方文档](https://realm.io/docs/java/latest/#installation)

[基本使用](http://www.tuicool.com/articles/V7ZFvuB)

[Realm数据库基础教程](http://www.cocoachina.com/ios/20150505/11756.html)