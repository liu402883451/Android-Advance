#android进阶之路精简版

###由于之前用macdown写的，简书的目录并没卵用，再研究研究，有知道的童鞋可以帮忙留言下，或者下载用macdown浏览效果好点哈~
[简书地址](https://www.jianshu.com/u/6fdcd457906a)
##目录

[1. Android触摸事件传递机制](#part1)<br>
[2. Android View的绘制流程](#part2)<br>
[3. Android动画机制](#part3)<br>
[4. Support Annotation Library使用详解](#part4)<br>
[5. Percent Support Library使用详解](#part5)<br>
[6. Design Support Library使用详解](#part6)<br>
[7. Android Studio中的NDK开发](#part7)<br>
[8. Gradle必知必会](#part8)<br>
[9.Builder模式详解](#part9)<br>
[10. 注解在Android中的应用](#part10)<br>
[11. ANR产生的原因及其定位分析](#part11)<br>
[12. Android异步处理技术](#part12)<br>
[13. Android数据序列化方案研究](#part13)<br>
[14.Android WebView Java和Javascript交互详解](#part14)<br>
[15.MVP模式及其在Android中的实践](#part15)<br>
[16.MVVM模式及其在Android DataBinding实战](#part16)<br>
[17. 观察者模式的拓展:事件总线](#part17)<br>
[18. 64K方法数限制原理与解决方案](#part18)<br>
[19.Android插件框架机制研究与实践](#part19)<br>
[20.推送机制实现原理详解](#part20)<br>
[21. APP瘦身经验总结](#part21)<br>
[22. Android Crash日志收集原理与实践](#part22)<br>
[23. 函数式编程思想及其在Android中的应用](#part23)<br>
[24. 依赖注入及其在Android中的应用](#part24)<br>
[25.Android世界的Swift:Kotlin在Android中的应用](#part25)<br>
[26.Android在线热修复方案研究](#part26)<br>
[27.面向切面编程及其在Android中的应用](#part27)<br>
[28.改造Android构建系统](#part28)<br>
[29. 代码优化](#part29)<br>
[30.图片优化](#part30)<br>
[31.布局优化](#part31)<br>
[32.Android混淆机制](#part32)<br>
[33.Android反编译机制详解](#part33)<br>
[34.Android加固技术研究](#part34)<br>
[35.Android安全编码](#part35)<br>
[36.Android调试工具Facebook Stetho](#part36)<br>
[37.内存泄漏检查函数库LeakCanary](#part37)<br>
[38.基于Facebook Redex实现Android APK的压缩和优化](#part38)<br>
[39.Android Studio你所需要知道的功能](#part39)

---

<span id = "part1"/>

##1. Android触摸事件传递机制

###1.1 	触摸事件的类型
		ACTION_DOWN:按下 	
		ACTION_MOVE:移动
		ACTION_UP:抬起

###1.2 	事件传递的三个阶段
* 分发(Dispatch):&nbsp; public boolean dispatchTouchEvent(MotionEvent ev)
	
* 拦截(Intercept):&nbsp; public boolean onInterceptTouchEvent(MotionEvent ev)

* 消费(Consume):&nbsp; public boolean onTouchEvent(MotionEvent ev)

		拥有事件传递处理能力的类:
		1. Activity:&nbsp;dispatchTouchEvent、onTouchEvent

		2. ViewGroup:&nbsp;dispatchTouchEvent、onInterceptTouchEvent、
		onTouchEvent

		3. View:&nbsp;dispatchTouchEvent、onTouchEvent

###1.3	View的事件传递机制
		拥有dispatchTouchEvent、onTouchEvent两个方法，其返回值:
		1.true
		2.false
		3.super.dispatchTouchEvent()
####总结
		1. 触摸事件的传递流程是从dispatchTouchEvent开始，默认情况（返回父类的同名函
		数），则事件从外层向内层传递，到达最内层的View时，就由它的onTouchEvent处理，该方
		法如果消费事件，返回true，处理不了返回false，这时事件重新向外层传递，并由外层
		View的onTouchEvent处理，依次类推。
		2. 如果事件在向内层传递过程中由于人为干预，事件处理函数返回true，则会导致事件提前
		被消费，内层View将不会收到这个事件
		3. View控件的事件触发顺序是先执行onTouch方法，在最后执行onClick方法。如果
		onTouch返回true，则事件不会继续传递，最后也不会调用onClick方法；如果onTouch返
		回false，则事件继续传递。


###1.4	ViewGroup的事件传递机制
		拥有dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent三个方法
####总结
		1.触摸事件的传递顺序是有Activity到ViewGroup，再由ViewGroup传递给它的子View
		2.ViewGroup通过onInterceptTouchEvent方法对事件进行拦截，如果方法返回true，
		则事件不会继续传递给子View，如果方法返回false或者
		super.onInterceptTouchEvent,则事件会继续传递给子View。
		3.在子View中对事件进行消费后，ViewGroup将接收不要任何事件。

---
<span id = "part2"/>
##2. Android View的绘制流程

####Android UI管理系统层级关系：
		由外到内: Activity --> PhoneWindow --> DecorView --> TitleView/
		ContentView
		PhoneWindow是Activity和View系统交互的接口。DecorView继承自FrameLayout，是
		Activity中所有View的祖先
###2.1 	绘制的整体流程
		Activity绘制从根视图ViewRootImpl的performTraversals()方法开始，从上到下遍历
		视图树，每个View控件负责绘制自己，而ViewGroup还要通知子View进行绘制操作。视图绘
		制三个步骤:测量（Measure）、布局（Layout）、绘制（Draw）。
###2.2 	MeasureSpec
		MeasureSpec表示一个32位的整型数，高两位表示测量模式SpecMode，低30位表示某种测
		量模式下的规格大小SpecSize。MeasureSpec是View类的形态内部类，用来说明应该如何
		测量这个View。
####三种测量模式：
		1.UNSPECIFIED: 不指定测量模式，通常用于系统内部，应用开发很少使用。
		2.EXACTLY: 精确测量模式，宽高指定为match_parent和具体数值时生效，这种模式下
		View的测量值就是SpecSize的值。
		3.AT_MOST: 最大值测量模式，宽高指定为wrap_content时生效，子视图尺寸可以是不超
		过父视图允许的最大尺寸的任何尺寸。
###2.3	Measure
		作用： 计算View的实际大小。
		1. 测量流程从ViewRootImpl的performMeasure方法开始，具体测量操作分发给
		ViewGroup，ViewGroup在它的measureChild方法传递给子View。
		2. View（ViewGroup）的measure方法，最终回调onMeasure方法实现，这个方法通常由
		View的特定子类实现，开发者也可以通过重写该方法实现自定义View。
		3. 如果没有重写onMeasure方法，默认调用getDefaultSize方法获得View的宽高。
###2.4	Layout
		作用： 确定View在父容器中的布局位置
		1. 布局流程从ViewRootImpl的performLayout方法开始，Layout过程由父容器获取子
		View的位置参数后，调用子View的layout方法并将位置参数传入实现。
		2. View中的layout方法中的onLayout方法是空方法，子类如果是ViewGroup，则重写
		onLayout，实现ViewGroup中所有View控件的布局流程。
		
###2.5	Draw
		作用： 将控件绘制出来。
		1. 绘制流程从performDraw方法开始，draw --> drawSoftware -->view.draw(canvas)
		2. 最终调用到View的draw方法绘制每个具体的View，绘制基本分为6个步骤：
			1）绘制View背景
			2）保存canvas图层，为fading做准备
			3）绘制View的内容
			4）绘制View的子View
			5）绘制View的fading边缘并恢复图层
			6）绘制View的装饰（例如滚动条）

---
<span id = "part3"/>			
##3. Android动画机制


		3.0之前逐帧动画和补间动画， 3.0属性动画， 4.4过渡动画
###3.1 逐帧动画（Frame Animation）
		两种方式：
		1.XML资源文件方式：res/drawable目录，xml使用<animation-list>标签，<item>为
		每一帧。
		2.代码方式：AnimationDrawable类
###3.2 补间动画（Tween Animation）
		两种方式1.res/anim目录、2.代码

		四种基本效果，可组合：
		1.透明度AlphaAnimation: 
			初始透明度 fromAlpha、结束透明度 toAlpha
			
		2.大小变化ScaleAnimation: 
			开始坐标伸缩尺寸 fromX、fromY
			坐标伸缩尺寸 toX、toY
			缩放中心点坐标 pivotX、pivotY
			
		3.位移变化TranslateAnimation
			位移开始坐标 fromXDelta、fromYDelta
			位移结束坐标 toXDelta、toYDelta
			
		4.旋转RotateAnimation
			起始终止角度 fromDegree、toDegree
			旋转中心点坐标 pivotX、pivotY
			
		插值器:Interpolator,渐变值，使动画匀速、加速、减速、抛物线等多种速度变化
		
###3.3 属性动画（Property Animation）
		两种方式:	 1.res/animator目录 2.代码
		与补间动画区别：直接改变了View对象的位置
		
		AnimatorSet用来组合多个Animator，指定这些Animator顺序播放还是同时播放。
		
		ValueAnimator是属性动画最重要的类：
			1.计算动画各个帧的相关属性值
			2.将这些属性值设置给指定的对象
			
		ObjectAnimator该类作为ValueAnimator的子类不仅继承了ValueAnimator的所有方法
		和特性，并且还封装很多实用的方法，方便开发人员快速实现动画。同时，由于属性值会自动
		更新，使用ObjectAnimator实现动画不需要像ValueAnimator那样必须实现 
		ValueAnimator.AnimatorUpdateListener ，因此实现任意对象的动画显示就更加容易
		了。我们在大部分的开发工作中，都会使用ObjectAnimator而非ValueAnimator实现我们
		所需的动画效果。
		
###3.4 过渡动画（Transition Animation）
		本质是属性动画，与属性动画区别是需要为动画前后准备不同的布局，并通过API实现两个布
		局的过渡动画。
		两种方式： 1.res/transition目录，2.代码
		
---
<span id = "part4"/>
##4. Support Annotation Library使用详解


		Android19.1引入的函数包，它包含一系列有用的元注解，用来帮助开发者在编译期间发现可
		能存在的bug。
###4.1 Nullness注解
		@Nullable作用于函数参数或者返回值，标记参数或者返回值可以为空。
		@NonNull作用于函数参数或者返回值，标记参数或者返回值不可以为空。

###4.2 资源类型注解
		规定传入的资源的类型，类型不对在编译期就报错。如@LayoutRes:表明需要传入
		android.R.layout类型的资源文件。
###4.3 类型定义注解
		用来规定方法可以传入的参数的值的指定值。
		如鲁班压缩中的压缩模式：
		@IntDef({FIRST_GEAR, THIRD_GEAR, CUSTOM_GEAR})
		public static final int FIRST_GEAR = 1;
	    public static final int THIRD_GEAR = 3;
    	public static final int CUSTOM_GEAR = 4;
    	这样调用方法只能使用这三个值
###4.4 线程注解
		@UiThread: 标记运行在UI线程，一个UI线程是Activity运行的主窗口
		@MainThread: 标记运行在主线程，一个应用只有一个主线程，主线程也是@UiThread
		@WorkerThread: 标记运行在后台线程
		@BinderThread: 标记运行在Binder线程
###4.5 RGB颜色值注解
		@ColorRes是资源类型注解
		@ColorInt标记参数类型需要传入RGB或者ARGB颜色整型值
###4.6 值范围注解
		函数参数的取值范围，防止传入错误的参数
		1.	@Size(min=1) 
			@Size(max=23) 
			@Size(2) 
			@Size(multiple=2)
		2. 	@IntRange：int或者long
		3.  @FloatRange：float或者double
###4.7 权限注解
		@RequiresPermission
###4.8 重写函数注解
		@CallSuper
###4.9 返回值注解
		@CheckResult
###4.10 @VisibleForTesting
		用于单元测试，使其对测试可见
###4.11 @Keep
		用于混淆过程中不需要混淆的类或者方法
---
<span id = "part5"/>
##5. Percent Support Library使用详解


		三个类 ： PercentFrameLayout、PercentRelativeLayout、PercentLayoutHelper
		属性:	layout_widthPercent: 用百分比表示宽度
				layout_heightPercent: 用百分比表示高度
				layout_aspectRatio: 用百分比表示View的宽高比(宽/高)
		
		鸿洋_第三方库:	
[博客1](https://blog.csdn.net/lmj623565791/article/details/49990941)
[博客2](https://blog.csdn.net/lmj623565791/article/details/45460089)
		
[jar包下载git地址](https://github.com/hongyangAndroid/Android_Blog_Demos/tree/master/blogcodes/src/main/java/com/zhy/blogcodes/genvalues)
		
[主流屏幕分辨率](http://screensiz.es/)

---
<span id = "part6"/>
##6. Design Support Library使用详解

###6.1. Snakbar: 
	带动画效果的快速提示栏，显示在屏幕底部，可替代Toast，不同的是可以带按钮。
###6.2. TextInputLayout: 
	作为EditText的容器，为EditText生成一个浮动的Label。
###6.3. TabLaout: 	
	固定Tabs: 对应xml配置app:tabMode="fixed",
	可滑动的Tabs: 对应xml配置app:tabMode="scrollable"
	可配合ViewPager使用: tabLayout.setupWithViewPager(mViewPager);
###6.4. NavigationView: 
	抽屉效果的导航视图,和DrawerLayout配合使用
###6.5. FloatingActionButton: 
	浮动操作按钮
	git地址: https://blog.csdn.net/qq_22770457/article/details/71249181?
	utm_source=gold_browser_extension
###6.6. CoordinatorLayout: 
	使不同视图组件直接相互作用，并协调动画效果。如： Snakbar出现FloatingActionButton
	自动向上移动，消失时FAB自动往下移动。将CoordinatorLayout作为FAB的父容器。
###6.7. CollapsingToolbarLayout: 
	作用是提供了一个可以折叠的Toolbar，通常和AppBarLayout配合使用。它继承至
	FrameLayout，给它设置layout_scrollFlags，它可以控制包含在
	CollapsingToolbarLayout中的控件
###6.8. BottomSheetBehavior: 
	实现底部动作条功能，在布局中添加app:layout_behavior属性，将这个布局作为
	CoordinatorLayout的子View。

---
<span id = "part7"/>
##7. Android Studio中的NDK开发

###7.1 ABI的基本概念
* Android目前支持7种CPU架构:ARMv5、ARMv7、x86、MIPS、ARMv8、MIPS6、x86_64
* 每一种架构关联着一种ABI(Application Binary Interface),它定义二进制文件(.so文件)如何运行在相应的系统平台上
	
		目录:src --> main --> jniLibs下
	
###7.2 在Android Studio中使用C++代码的两种方式
		1.引入预编译的二进制库(so包方式)
		2.在Android Studio中直接从C/C++源码编译
###7.3 在Gradle中添加原生.so文件依赖的方式
		1、配置ndk.dir
		local.properties文件
		ndk.dir=/Users/work/Library/Android/sdk/ndk-bundle
		
		2、在Gradle中配置NDK模块
		module下的build.gradle文件
		defaultConfig{
			ndk {
            	moduleName “xxx”
            	...
        	}
		}
        
		3、添加C/C++文件到指定的目录:src/main/xxx
			android {
				...
				sourceSets.main {
					jni.srcDirs 'src/main/xxx'
				}
			}
###7.4 使用.so文件时一些注意事项
* 使用高平台版本编译的.so文件运行在低版本设备上
* 混合使用不同的C++运行时编译的.so文件
* 没有为每个支持的CPU架构提供对应的.so文件
* 将.so文件放在错误的地方
* 只提供armeabi架构的.so文件而忽略其他ABIs的

---
<span id = "part8"/>
##8. Gradle必知必会

###8.1 共享变量的定义
	1.在工程根目录创建一个common_config.gradle的文件
	project.ext {
		compileSdkVersion = 23
		...
	}
	2.在各个module的build.gradle文件引用:
	apply from: "${project.rootDir}/common_config.gradle"
	android {
		compileSdkVersion project.ext.compileSdkVersion
		...
	}
###8.2 通用配置
	module中的 apply from: "${project.rootDir}/common_config.gradle"替换成以下方式:
	在工程的build.gradle文件中配置subprojects
	subprojects {
		apply from: "${project.rootDir}/common_config.gradle"
		...
	}
###8.3 aar函数库的引用
	module1中引用了aar文件中的类时，其他module依赖module1时，会提示aar文件找不到错
	误。
	解决方案: 在根目录的build.gradle文件中添加配置
	allprojects {
		repositories {
			...
			flatDir {
				dirs '../module1/libs'
			}
		}
	}
###8.4 签名和混淆的配置
	签名：signingConfigs{...}
	https://www.cnblogs.com/gao-chun/p/4891275.html
	混淆：
	buildTypes { 
		release {
            minifyEnabled true//开启混淆
            signingConfig signingConfigs.release
        }
	}
---- 

<span id = "part9"/>
## 9.Builder模式详解

### 9.1 经典的Builder模式
	#### 经典的Builder模式四个参与者:
	1.Product: 被构造的复杂对象，ConcreteBuilder用来创建该对象的内部表示，并定义它的
	装配过程。(JavaBean)
	2.Builder: 抽象接口，定义创建Product对象的各个组成部件的操作。
	3.ConcreteBuilder: Builder接口的具体实现，可定义多个，是实际构建Product的地方，
	提供一个返回Product的接口。
    4.Director: Builder接口的构造者和使用者。 
	
	@Data
	public class Product {
		//JavaBean
		... XX1;
		... XX2;
	}
	
	public interface Builder {
		public void buildXX1();
		public void buildXX2();
		public Product getProduct();
	}
	
	public class ConcreteBuilderA implements Builder {
		private Product product;
		//复写Builder接口的抽象方法
		...
	}
	
	public class ConcreteBuilderB implements Builder {
		private Product product;
		//复写Builder接口的抽象方法
		...
	}
	
	public class Director {
		private Builder builder;
		public Director(Builder builder){
			this.builder = builder;
		}
		public void buildProduct(){
			this.builder.buildXX1();
			this.builder.buildXX2();
		}
		public Product getProduct(){
			return this.builder.getProduct();
		}
	}
####PS: @Data为Lombok插件，JavaBean注解，可省略getters和setters方法
### 9.2 Builder模式的变种
	对于JavaBean中有必选属性和可选属性这种情况:
	将必选属性声明成final的，必须在构造函数中对必选属性进行赋值，否则编译不通过。
	1. 一种方案是定义多个重载的构造函数，但只适用于少量属性的情况，不然随着实体类属性个数
	增加，会难以阅读和维护。
	2. 另一种方案是遵循JavaBeans规范，定义一个默认无参的构造函数，为每个属性提供getters
	和setters函数。
		但是存在缺点:实体类是可变的，实体类的实例状态不连续，创建对象一直调用set方法很不方
	便。
	此时变种的Builder模式产生:
	@Data
	public calss User {
		private final String XX1;//必选属性
		...//多个final属性
		private User(UserBuilder builder){
			XX1 = builder.XX1;
			...
		}
		public static class UserBuilder	 {
			private final String XX1;//必选属性
			private String XX2;//非必选属性
			...
			public UserBuilder(String XX1){
				this.XX1 = XX1;
			}
			public UserBuilder setXX2(String XX2){
				this.XX2 = XX2;
				return this;
			}
			...//多个类似setXX2的非必选属性的方法
			public User builder() {
				return new User(this);
			}
		}
	}
	
	调用: new User.UserBuilder("xx1")
				.setXX2("xx2")
				.build();
### 9.3 变种Builder模式的自动化生成
	Android Studio插件InnerBuilder
	写完属性名，鼠标右键Generate --> Builder
	自动生成Builder相关代码
### 9.4 开源函数库的例子
	1. AlertDialog
	2. 图片缓存函数库Android-Universal-Image-Loader的全局配置
	3. 网络请求框架OkHttp的请求封装

----

<span id = "part10"/>	
## 10. 注解在Android中的应用

		REST网络请求函数库Retrofit使用运行时注解，依赖注入函数库Dagger2使用编译时注解。
### 10.1 注解的定义
	注解通过 @interface 声明，接口的方法对应着注解的元素
### 10.2 标准注解
	java API中默认定义的注解，它定义在java.lang、java.lang.annotation和
	javax.annotation包中。
	1. 编译相关注解	@Override、@Deprecated、@SuppressWarnings、@Generated、
	@FunctionalInterface
	2. 资源相关注解，一般用在JavaEE领域，Android开发应该不会使用到。
	3. 元注解	用来定义和实现注解的注解
		五种:	@Target： 用来指定注解所适用的对象范围 *
				@Retention： 用来指明注解的访问范围 *
					源码级注解: @Retention(RetentionPolicy.SOURCE)
					编译时注解: @Retention(RetentionPolicy.CLASS)//默认
					运行时注解: @Retention(RetentionPolicy.RUNTIME)
				@Documented： 表示被修饰的注解应该被包含在被注解项的文档中
				@Inherited： 表示注解可以被子类继承
				@Repeatable： 表示注解可以在同一个项上应用多次，Java8引入，其他四个
				Java5引入
### 10.3 运行时注解
	一般配合反射机制使用，相比编译时注解性能较低，灵活性好。
### 10.4 编译时注解
	1、定义注解处理器
	2、注册注解处理器
	
		@AutoService(Processor.class)	//生成 META-INF 信息
		@SupportedAnnotationTypes({"com.xxx.BindView"}) //声明 Processor 处理
		的注解，注意这是一个数组，表示可以处理多个注解；
		@SupportedSourceVersion(SourceVersion.RELEASE_7) //声明支持的源码版本
		public class ViewInjectProcessor extends AbstractProcessor { 
			@Override public boolean process(Set<? extends TypeElement> set, 
			RoundEnvironment roundEnvironment) {
			 //... 具体实现
			 } 
		}
	
	3、android-apt插件
----

<span id = "part11"/>
## 11. ANR产生的原因及其定位分析

### 11.1 ANR产生的原因
	只有UI线程相应超时才会引起ANR
	1.当前的事件没有得到处理
	2.当前的事件处理耗时太长
	Activity/View: keyDispatchTimeout
	BroadcastReceiver: BroadcastTimeout
	Service: ServiceTimeout
### 11.2 典型的ANR问题场景
	1.UI线程存在耗时操作
	2.UI线程等待子线程释放某个锁，从而无法处理用户的输入
	3.耗时的动画需要大量的计算工作
### 11.3 ANR的定位和分析
	1.Logcat日志信息 
	2.traces.txt日志信息,可定位到代码行
	mac安装adb命令: brew cask install android-platform-tools
	从手机内部存储拷贝的电脑命令: adb pull /data/anr/traces.txt ~/Desktop/
### 11.4 ANR的避免和检测
	除了切记不要在主线程中做耗时操作，也可以借助于工具来进行检测
	1.StrictMode:严苛模式

	StrictMode.setThreadPolicy(
          	new StrictMode.ThreadPolicy.Builder()
            	.detectNetwork()
             	.detectCustomSlowCalls().penaltyDeath().build());
    StrictMode.setVmPolicy(
    		new StrictMode.VmPolicy.Builder()
        		.detectLeakedSqlLiteObjects()
     			.detectLeakedClosableObjects()
              	.detectLeakedRegistrationObjects()
               	.detectActivityLeaks()
              	.penaltyDeath().build());
                       
	2.BlockCanary:监控主线程的卡顿
	// 仅在debug包启用BlockCanary进行卡顿监控和提示的话，可以这么用
    debugCompile 'com.github.markzhai:blockcanary-android:1.5.0'
    releaseCompile 'com.github.markzhai:blockcanary-no-op:1.5.0'

----
<span id = "part12"/>
## 12. Android异步处理技术

###12.1 Thread
	两种方式:1.继承Thread类 	2.实现Runnable接口
	应用层可分为三种类型的线程: 
		主线程：UI线程，更新UI，非主线程更新UI会抛出CalledFromWrongThreadException
		Binder线程：用于不同进程之间的通信，典型例子AIDL
		后台线程：应用中显式创建的线程都是后台线程，需要手动添加任务
###12.2 HandlerThread
	集成了Looper和MessageQueue的线程，启动时同时生成Looper和MessageQueue，然后等待
	消息进行处理。
	不需要开发者去创建和维护Looper，用法和普通线程一样:
	mHandlerThread = new HandlerThread("HandlerThread");
        mHandlerThread.start();

        mHandlerInNewThread = new Handler(mHandlerThread.getLooper()){
            @Override
            public void handleMessage(Message msg) {
                switch(msg.what){
                	//处理接收到的消息    
                }
            }
        };
	HandlerThread只有一个消息队列，消息顺序执行，线程安全，吞吐量收到一定影响，队列中任
	务可能被前面没执行完的任务阻塞。
	如果需要在HandlerThread接收消息之前进行初始化操作，可以重写HandlerThread的
	onLooperPrepared函数
###12.3 AsyncQueryHandler
	用于ContentProvider执行异步的CRUD操作的工具类，CRUD操作被放到单独的一个子线程执
	行，操作结束获取到结果后，将通过消息的方式传递给调用的AsyncQueryHandler的线程，通常
	就是主线程。
	
	AsyncQueryHandler封装了四个方法用来操作ContentProvider的CRUD操作:
	1. public final void startInsert(int token, Object cookie, Uri uri, 
	ContentValues initialValues)
	
	2. public final void startDelete(int token, Object cookie, Uri uri, 
	String selection, String[] selectionArgs)
    
    3. public final void startUpdate(int token, Object cookie, Uri uri, 
    ContentValues values, String selection, String[] selectionArgs)        
   
    4. public void startQuery(int token, Object cookie, Uri uri, String[] 
    projection, String selection, String[] selectionArgs, String orderBy)
    
    AsyncQueryHandler的子类实现回调函数，得到CRUD操作返回的结果:
    1. protected void onInsertComplete(int token, Object cookie, Uri uri) {
        // Empty
    } 
    2. protected void onDeleteComplete(int token, Object cookie, int 
    result) {
        // Empty
    }
    3. protected void onUpdateComplete(int token, Object cookie, int 
    result) {
        // Empty
    }
    4. protected void onQueryComplete(int token, Object cookie, Cursor 
    cursor) {
        // Empty
    }
###12.4 IntentService
	Service的各个生命周期函数运行在主线程，为了Service中实现在子线程中处理耗时任务，引入
	IntentService
	与HandlerThread类似，在一个后台线程顺序执行所有任务，通过Context.startService传
	递一个Intent类型参数启动IntentService的异步执行。后台线程队列所有任务处理完成后，
	IntentService会结束生命周期，不需要开发者手动结束。
	IntentService是抽象类，使用需要继承它并实现onHandleIntent方法，同时在子类的构造方
	法中需要调用super(String className)传入子类的名字。
###12.5 Executor Framework
	创建和销毁对象存在开销，影响性能。使用Java Executor框架可以通过线程池等机制解决这个
	问题。
		1.创建工作线程池，通过队列控制线程执行的任务个数。
		2.检测导致线程意外终止的错误。
		3.等待线程执行完成并获取执行结果。
		4.批量执行线程，通过固定顺序获取执行结果。
		5.在合适的时机启动后台线程，保证线程执行结果很快反馈给用户。
		
	Executor框架预定义的线程池实现:
		1.固定大小的线程池:Executors.newFixedThreadPool(int n);//n为线程个数
		2.可变大小的线程池:Executors.newCachedThreadPool();
		3.单个线程的线程池:Executors.newSingleThreadPool();
		
	预定义线程池基于ThreadPoolExecutor，通过ThreadPoolExecutor可自定义线程池行为。
		其中一个构造函数：
	 	public ThreadPoolExecutor(int corePoolSize, 
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue)
        corePoolSize：核心线程数，一直存在于线程池中，即使当前没有任务需要处理。
        maximumPoolSize：最大线程数
        keepAliveTime：线程的空闲存活时间
        unit：keepAliveTime的单位（NANOSECONDS/MICROSECONDS/MILLSECONDS/
        SECONDS）
        workQueue：线程池所使用的任务缓存队列
        
###12.6 AsyncTask
	AsyncTask是Executor框架基础上的封装，它实现耗时任务在工作线程执行，提供接口实现工作
	线程和主线程的通信。
	子类继承AsyncTask，可重写方法：
    @Override
    protected Object doInBackground(Object[] objects) {
        ...//工作线程
    }

    @Override
    protected void onPostExecute(Object o) {
        ...//主线程
    }

    @Override
    protected void onPreExecute() {
        ...//主线程
    }

    @Override
    protected void onProgressUpdate(Object[] values) {
        ...//主线程
    }

    @Override
    protected void onCancelled(Object o) {
        ...//主线程
    }
    AsyncTask执行并行任务，在API大于13建议使用excuteOnExecutor(串行或并行)代替
    execute(串行执行)
    
    一个应用中使用的所有AsyncTask实例共享全局的属性，
    1.如果AsyncTask中的任务是串行执行，其他的AsyncTask都会排队
    2.如果AsyncTask中的任务是异步执行，那么四核CPU系统上，最多五个任务可以同时进行。
    AsyncTask的ThreadPoolExecutor指定核心线程数是系统CPU核数+1。
###12.7 Loader
	Android 3.0引入的异步数据加载框架，在数据源发送变化时，能够及时发出消息通知。涉及API:
		1.Loader:加载器框架的基类，封装了实现异步数据加载的接口。
		2.AsyncTaskLoader:Loader的子类,基于AsyncTask实现的异步数据加载，抽象类，子类
		必须实现loadInBackground方法。
		3.CursorLoader:AsyncTaskLoader的子类，封装了对ContentResolver的query操作
		4.LoaderManager:抽象类，Activity和Fragment默认关联一个LoaderManager的对
		象，通过getLoaderManager()获取
		5.LoaderManager.LoaderCallbacks:LoaderManager的回调接口，主要三个方法：
			public Loader<D> onCreateLoader(int id, Bundle args);//初始化并返回
			一个新的Loader实例
			public void onLoadFinished(Loader<D> loader, D data);//当加载器完成
			加载过程回调
			public void onLoaderReset(Loader<D> loader);//当加载器被重置并且数据
			无效时回调

----
<span id = "part13"/>
## 13. Android数据序列化方案研究

### 13.1 Serializable
		Serializable接口是标识接口，无需实现方法。缺点是使用反射机制，效率较低。(JDK提供
		的接口)
		序列化过程会使用名为serialVersionUID的版本号和序列化的类相关联，如果接收者和发送
		者的serialVersionUID取值不同，出现InvalidClassException异常。最佳实践是显式
		指定serialVersionUID的值(IDE随机生成)。
		1. 序列化一个对象，首先创建某种类型的OutputStream，接着将OutputStream封装到
		ObjectOutputStream对象中，然后调用ObjectOutputStream的writeObject方法进行
		序列化。PS:序列化基于字节，不能使用Reader和Writer基于字符的方式。
		2. 反序列化过程和序列化过程是相对的，创建InputStream，接着封装在
		ObjectInputStream中，调用readObject。
### 13.2 Parcelable
		Parcelable是Android SDK提供的接口，基于内存，读写速度高于磁盘，通常用于跨进程
		对象的传递。
		Android studio中安装Android Parcelable code generator插件可自动生成代码模
		板。
		插件地址网站:http://plugins.jetbrains.com/
		实现Parcelable接口需要实现几个方法:
			1. describeContents : 接口内容的描述，一般返回0即可。
			2. writeToParcel : 序列化的方法，将类的数据写入到Parcel容器中。
			静态的Parcelable.Creator接口:包含两个方法：
				3. createFromParcel : 反序列化的方法，将Parcel还原成Java对象。
				4. newArray : 提供给外部类反序列化这个数组使用。
### 13.3 SQLiteDatabase
		轻量级关系型数据库，占用内存小，运算速度极快。但是其API不友好，可使用ORM框架。安全
		性问题:默认目录是/data/data/PACKAGE_NAEM/database,在手机Root后可以直接访
		问，敏感信息需要加密，可使用开源框架sqlcipher。
[SQLiteDatabase博客地址](http://www.cnblogs.com/whoislcj/p/5511522.html)

### 13.4 SharedPreferences
		Android提供的轻量级存储API，一般保存应用的一些常用配置信息。SharedPreferences
		信息保存在/data/data/PACKAGE_NAEM/shared_prfs目录，Root后可直接访问，敏感信
		息需要加密。可使用开源框架conceal或者SharedPreferences的封装类secure-
		preferences(无须开发者手动编写加解密代码)。
### 13.5 JSON
		JavaScript Object Notation,轻量级的数据交换格式。Android原生JSON解析API性
		能很差，可使用GSON、fastJson
		
----
<span id = "part14"/>
## 14.Android WebView Java和Javascript交互详解

### 14.1. Java调用Javascript:
	mWebView.loadUrl("javascript:toast()");//其中toast()方法是HTML5页面中的
	Javascript函数
### 14.2. Javascript调用Java:
	WebView提供一个名为WebSettings的工具类实现让WebView中的Javascript脚本调用
	Android应用的Java方法。
	使用步骤:
		1.调用WebView关联的WebSettings实例的setJavaScriptEnabled
		2.调用WebView的addJavascriptInterface方法将应用中的Java对象暴露给
		Javascript
		3.在Javascript脚本中调用步骤二暴露出来的Java对象的方法
	
		在Android4.2之前可能存在安全隐患，解决方案:不再使用addJavascriptInterface方
		式，使用onJsPrompt方法，在Javascript中将字符串信息(对应onJsPrompt入参中的
		message)传递给Java，而Java执行后把返回结果的字符串形式(对应onJsPrompt入参中的
		mStringResult)传递给Javascript。麻烦！！！可使用开源函数库:safe-java-js-
		webview-bridge
[safe-java-js-webview-bridge博客地址](https://www.jianshu.com/p/f09dd86f0fb4)

----
<span id = "part15"/>
## 15.MVP模式及其在Android中的实践


	View层: 视图层，包含界面相关的功能。一般持有Presenter层的引用，或者通过Dagger依赖
	注入的方式获得Presenter的实例，并将非UI的逻辑操作委托给Presenter。
	Presenter层: 逻辑控制层，用来隔离View层和Model层。接收View层的网络请求，分发给对应
	的Model处理，监听Model层的处理结果，最终将其反馈给View层，实现界面的刷新。
	Model层: 封装各种数据来源，对Presenter层提供简单易用的接口。
	
	MVP的开源实现:
		1.Android Architecture Components
		2.TODO-MVP
		3.TODO-MVP-Loaders
		4.TODO-MVP-Clean
		5.TODO-Databinding

----		
<span id = "part16"/>
## 16.MVVM模式及其在Android DataBinding实战


	Model-View-ViewModel，将Presenter改为了ViewModel，实现View和ViewModel的双向
	绑定。View层的变化自动导致ViewModel发生变化，ViewModel的数据变化也会自动实现View
	的刷新。引入方式:
	build.gradle文件加入:
	android {
		...
		dataBinding {
			enabled = true
		}
	}
### 16.1 Data Binding表达式
	布局文件以<layout>作为根标签，里面包含data和view标签，其中data实现数据绑定，view标
	签是布局。
	<layout>
		<data>
			<variable name = "user" type = "com.example.User"/>
		</data>
		<LinearLayout ...>
			<TextView ...android:text = "@{user.name}"/>
		</LinearLayout>
	</layout>
### 16.2 数据对象
	以上代码的user变量成为数据对象，它可以是POJO类，也可以是JavaBean对象。
	@Data
	public class User {
		private final String name;
	}
### 16.3 数据绑定
	默认情况下，Binding类的命名以布局文件名加上Binding作为后缀组成。
	main_activity.xml --> MainActivityBinding
	
	MainActivityBinding binding = 
	DataBindingUtil.setContentView(this,R.layout.main_activity);
	User user = new User("xxx");
	binding.setUser(user);
	
	也可以通过inflate方式:
	MainActivityBinding binding = DataBindingUtil.inflate(getLayoutInflater());
	
	RecyclerView的adapter中:
	ListItemBinding binding = ListItemBinding.inflate(layoutInflater, 
	R.layout.list_item, false);
	ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, 
	R.layout.list_item, viewGroup, false);
### 16.4 事件绑定
	public class MyHandlers {
		public void onClickName(View view){...}
	}
	
	<layout>
		<data>
			<variable name = "handlers" type = "com.example.MyHandlers"/>
			<variable name = "user" type = "com.example.User"/>
		</data>
		<LinearLayout ...>
			<TextView ...android:text = "@{user.name}"
				android:onclick = "@{handlers.onClickName}"
			/>
		</LinearLayout>
	</layout>

----
<span id = "part17"/>
## 17. 观察者模式的拓展:事件总线


	继承自观察者模式，事件总线也是基于发布订阅的机制来实现事件的发送和接收的。
### 17.1 为何要使用
	不同组件或者模块间的消息传递，传统模式耦合度很高。事件总线可以实现组件和模块间的解耦。
### 17.2 原理
	事件总线涉及:
	事件Event: 一个普通的POJO类，只包含数据，不包含对数据的操作。
	订阅者Subscriber: 订阅某种类型事件的对象，有一个回调函数对接收到的事件进行处理，可以
	订阅事件，也可以取消订阅事件，订阅可以有优先级。
	发布者Pbulisher: 事件的源头，发布某种类型事件的对象。
	总线EventBus: 负责订阅者、事件等信息的存储，同时处理事件的流动和分发。
### 17.3 开源实现
EvntBus: 主要适合组件和模块间通信使用。
[https://www.jianshu.com/p/f9ae5691e1bb](https://www.jianshu.com/p/f9ae5691e1bb)	
RxJava: 用于异步数据流的处理，基于函数响应式编程方式。[https://www.jianshu.com/p/
a93c79e9f689](https://www.jianshu.com/p/a93c79e9f689)

----

<span id = "part18"/>
## 18. 64K方法数限制原理与解决方案 


 	Android Dalvik可执行文件.dex中的Java方法数引用超过65536，将出现编译错误:
 	旧版本构建系统:	
 		Conversion to Dalvik format failed:
 		Unable to excute dex:method ID not in [0,0xffff]:65536
	新版本构建系统:	
		trouble writing output:
		Too many field references: 131000; max is 65536.
		You may try using --multi-dex option.	
 	
### 18.1 64K限制的原因
	Android APK文件本质上是一个压缩文件，包含classes.dex文件是可执行的Dalvik字节码文
	件，这个.dex文件中存放的是所有编译后的Java代码。Dalvik可执行文件规范限制了单个.dex
	文件最多能引用方法数65536个，其中包含Android Framework，第三方库函数以及APP自身的
	方法。
### 18.2 使用MultiDex解决64K限制的问题
	1. Android 5.0之前的版本:
	Dalvik为每个APK只生成一个classes.dex文件，我们需要拆分单一的classes.dex文件，类
	似于classes.dex、classes2.dex、classes3.dex，启动应用时会先加载classes.dex文
	件(主文件)，然后加载其他文件。Google推出MultiDex Support Library函数库。
	2. Android 5.0之后的版本:
	使用ART虚拟机替代Dalvik虚拟机，天然支持从APK文件中加载多个.dex文件，应用安装期间会执
	行预编译操作，扫描APK中的classes(..N).dex文件并编译成单一的.oat文件，在应用运行时
	去加载这个.oat文件。
	
### 18.3 如何避免出现64K限制
	使用MultiDex Support Library是下下策，会降低应用的性能。应该避免出现64K限制，方法
	如下：
	1.检查应用的直接和间接第三方依赖，避免引入很大的函数库，仅仅只用到了其中很少的功能。
	2.使用Proguard移除无用的代码:它的压缩功能通过分析字节码，能够检测并移除没用使用到的
	类、字段、方法和属性。
	
### 18.4 配置MultiDex
	1. build.gradle文件:
	android {
		...
		defaultConfig {
			...
			multiDexEnabled true
		}
	}
	dependencies {
		compile 'com.android.support:multidex:1.0.1'
	}
	2. 	没用自定义Application类:
			在清单文件中使用MultiDexApplication替换Application。
		自定义了Application类:
			可以让它继承MultiDexApplication
		自定义了Application类，不想修改它的父类，可以复写attachBaseContext方法
		@Override
    	protected void attachBaseContext(Context base) {
        	super.attachBaseContext(base);
        	MultiDex.install(base);
    }
        
### 18.5 MultiDex Support Library的局限性
	1. 首次启动时Dalvik加载所有.dex文件，如果从dex文件太大会导致ANR。
	2. Android 4.0之前系统上使用MultiDex可能启动失败。(还好4.0之前已经很少)
	3. 由于Dalvik线性内存分配器linearAlloc的限制，使用MultiDex的应用在出现很大的内存
	分配时，可能会导致应用崩溃
	4. 当引入MultiDex机制时，必然会存在主从dex文件，应哟好难过启动所需要的类都必须放到主
	dex中，否则出现NoClassDefFoundError。
	
### 18.6 在开发阶段优化MultiDex的构建
	然并卵~~~

---
<span id = "part19"/>
## 19.Android插件框架机制研究与实践


	MultiDex方案并不完美，而且APK启动速度会受影响，所以引入插件框架的概念。
	插件框架基本形式是将一个APK中的不同模块(插件)进行拆分，并打入到不同的dex文件或者APK文
	件中，主工程(宿主)只是一个空壳，提供了用来加载dex文件或者APK文件的框架。好处: 
		1.提升Android Studio工程构建速度
		2.提高应用的启动速度
		3.支持多团队并行开发
		4.在线动态加载或者更新模块
		5.按需加载不同的模块，实现灵活的功能配置。
### 19.1 基本概念
	1. 宿主(主工程)和插件(功能模块)
	2. ClassLoader机制:
		PathClassLoader只能加载已经安装的APK文件，不符合需求。
		DexClassLoader支持加载外部的APK/Jar/dex文件，所有插件化方案都是使用
		DexClassLoader。
### 19.2 开源框架
	1. DynamicAPK(携程方案，可热更新)	
		缺点: 插件工程不支持Native代码，例如不支持so库，
			 不支持对Library工程、arr、maven远程仓库的依赖。

	2. DroidPlugin(360手机助手方案)			
		缺点: 插件APK不支持自定义资源的Notification，
			 插件APK无法注册特殊IntentFilter的四大组件，
			 对某些带有Native代码的插件APK支持不好，可能无法正常运行，
			 插件与插件、插件与宿主之间代码完全隔离，只能通过Android系统级别的通信方式。
		https://github.com/Qihoo360/DroidPlugin/blob/master/readme_cn.md

	3. Small(轻量级跨平台插件化框架)
		优点: 所有插件支持内置于宿主包中，
			 所有编码和资源文件的使用与开发普通应用没有差别，
			 通过设定URI，宿主以及Native应用插件，web插件，在线网页等能够方便的进行通
			 信，
			 支持Android、IOS、HTML5，三者可以通过同一套JavaScript接口实现通信。
		缺点: Small目前看来唯一的不足是暂不支持Service动态注册，不同可以通过将Service
		预先注册在宿主的清单文件中进行规避，因为Service的更新频率通常非常低。
[Small git](https://github.com/liu402883451/Small)<br/>
[Small 博客](https://www.jianshu.com/p/7990714d10cb)<br/>
[Small 官方文档](http://code.wequick.net/Small/cn/quickstart)<br/>

----		
<span id = "part20"/>
## 20.推送机制实现原理详解


	移动端主动向服务器端发起Request请求叫Pull模式-->短连接，反之，服务器端主动发送消息给
	移动端的通信模式叫Push模式，即推送机制-->长连接
	移动端通过定时器轮询的方式向服务端发起Pull请求--伪推送，耗电耗流量。真正的推送机制是基
	于TCP长连接实现的。
### 20.1 推送的开源实现方案
	1. 基于XMPP协议
		XMPP协议方式实现推送简单、相对稳定，但是它基于XML协议实现消息通信，耗流量，不推
		荐。
	2. 基于MQTT协议
		IBM开发的轻量级的消息传输协议，和XMPP协议相似，基于发布订阅模式实现即时消息通信，
		传输格式非常精小，最小数据包只有2个比特。推荐。
### 20.2 推送的第三方平台
	常见的:友盟推送、小米推送、百度推送、极光推送等。
### 20.3 自己实现推送功能
	Android中想要建立TCP长连接，不能使用HttpUrlConnection或者HttpClient等网络请求
	API，它们是更上层的，属于HTTP协议。Java提供了网络套接字Socket，封装了很多TCP的操
	作。
	推送的基本框架需要包含:
		1.和服务端建立连接的功能
		2.发送数据给服务端的功能
		3.从服务端接收推送数据的功能
		4.心跳包的实现
	代码实现中，每个功能会封装成独立的线程，通过一个管理器统筹连接的建立和管理。
#### 20.3.1 长连接的建立(TCPConnectThread)
	核心代码: TCP_URL(服务端的URL地址)、TCP_PORT(端口号)、
	SOCKECT_CONNECT_TIMEOUT(连接超时时间)
	mSocket = new Socket();
	mSocket.connect(new InetSocketAddress(TCP_URL,TCP_PORT),
	 SOCKECT_CONNECT_TIMEOUT);
	mSocket.setKeepAlive(true);//长连接
#### 20.3.2 数据的发送(TCPSendThread)
	长连接建立后，需要保存返回的Socket实例，这个实例代表这个长连接的通道。Socket通信发送
	的是字节数据，一个完整的消息至少包含协议头+协议主体内容+校验码。
#### 20.3.3 数据的接收(TCPReceiveThread)
	与数据的发送类似，使用DataInputStream读取数据
#### 20.3.4 心跳包的实现(TCPHeartBeatThread)
	为了保持长连接的可用性，移动端需要每隔一段时间往Scoket通道中发送一个心跳包。心跳包也可
	以附带一个其他信息，如GPS定位信息等。
	Android中心跳包的间隔性发送可以通过AlaemManager实现，也可以在线程中通过while循环
	+Thread.sleep()方式实现。
---

<span id = "part21"/>
## 21. APP瘦身经验总结

### 21.1 优化图片资源占用的空间
	1. PNG/JPEG转换为WebP
	2. 尽量使用NinePatch格式的PNG图
### 21.2 使用Lint删除无用资源
	
### 21.3 利用Android Gradle配置
	minifyEnable:标识是否开启混淆
	shrinkResources:标识是否去除无用的resource文件
	ndk.abiFilters
### 21.4 重构和代码优化
### 21.5 资源混淆
	微信方案:	AndResGuard 
[AndResGuard](https://github.com/liu402883451/AndResGuard)
	
### 21.6 插件化--如:Small、RePlugin(360方案)

----
<span id = "part22"/>
## 22. Android Crash日志收集原理与实践

### Java层Crash捕获机制
	异常分两类: 
		CheckedException: 编译时异常，编译阶段必须处理，否则编译失败。
		UnCheckedException: 运行时异常，程序运行时某些条件触发，也可以用try-catch捕
		获，但实际上不知道在哪些地方捕获。
	Java API提供一个全局异常捕获处理器，Thread.UncaughtExceptionHandler接口，可以实
	现该接口，重写uncaughtException方法，在该方法中获取Crash的堆栈信息。
	使用自定义的UncaughtExceptionHandler，需要在Application类的onCreate方法中进行
	注册:
	Thread.setDefaultUncaughtExceptionHandler(new 
	MyUncaughtExceptionHandler());
	
### Crash上报
	包含信息:应用版本号、系统类型及版本号、手机型号及版本号、手机唯一的设备ID、渠道号、
	Crash发生的时间、应用的包名
	第三方:bugly、友盟...
----

<span id = "part23"/>
## 23. 函数式编程思想及其在Android中的应用


	函数式编程是一种编程范式，它的基础是lambda演算。lambda演算的函数可以接受函数作为输入
	和输出。
	面向对象编程: 数据和数据的操作是耦合在一起的，核心的抽象模型是数据本身，核心的活动是组
	合新对象以及拓展存在的对象。
	函数式编程: 数据与函数是松耦合的，函数隐藏了自身的实现，核心抽象模型是函数，而不是数据
	结构，核心活动是编写新的函数。可以极大简化项目，特别是处理嵌套回调的异步事件、复杂的列
	表过滤和变换或时间相关问题。

	Android开发中的框架: 
	1. RxJava和RxAndroid
	RxJava: 一个Java虚拟机上实现的函数响应式扩展库，它扩展自观察者模式，提供了基于
	observable序列实现的异步调用及基于事件的响应式编程。
	RxAndroid: RxJava在Android平台的拓展，能够简化Android上面基于RxJava的开发。
		
	2. Google推出的Agera框架，有望成为官方标准
	Android SDK9以上。类似于RxJava，也是基于观察者模式实现的。
	
### 23.1 以RxJava为例
	响应式代码的基本组成部分是Observables和Subscribers，Observable用来发送消息，
	Subscriber用来接收消息。Observable可以发送任意数量的消息，当消息被成功处理或者出错
	时，流程结束。Observables会调用它的每个Subscriber的onNext()方法，并最终以
	Subscriber.onComplete()或者Subscriber.onError结束。
	Observables一般只有等到Subscriber订阅它，才会开始发送消息。
	
	实例代码:
	1.Observable
	Observable<String> myObservable = Observable.create(
		new Observable.onSubscribe<String>(){
			@Override
			public void call(Subscriber<? super String> sub){
				sub.onNext("xxx");
				sub.onComplete();
			}
		}
	);
	
	2.Subscriber
	Subscriber<String> mySubscriber = new Subscriber<>() {
		@Override
		public void onNext(String s) { Log.d(s); }
		
		@Override
		public void onComplete() {  }
		
		@Override
		public void onError(Throwable e) {  }
	};
	3.订阅
	myObservable.subcribe(mySubscriber);//输出xxx
	当订阅完成后，myObservable将调用mySubscriber的onNext()和onComplete()方法
	
### 23.2 代码的简化
	RxJava为常见的操作提供很多内建的Observable创建函数，例如可以用Observable.just()
	方法简化上面的Observable代码。
	1.Observable
	Observable<String> myObservable = Observable.just("xxx");
	
	2.Subscriber
	Action1<String> onNextAction = new Action1<>() {
		@Override
		public void call(String s) { 
			Log.d(s); 
		}
	};
	Actions可以用来定义Subscriber的每一个部分。Observable.subcribe()方法能接收一
	个、两个、三个参数，分别表示onNext()、onComplete()、onError()方法。
	
	3.订阅
	myObservable.subcribe(onNextAction);//输出xxx
	
	链式调用:
	Observable.just("xxx")
		.subcribe(new Action1<>() {
			@Override
			public void call(String s) { 
				Log.d(s); 
			}
		});
		
	Java8的lambda表达式:
	Observable.just("xxx")
		.subscribe(s -> Log.d(s));
		

### 23.3 Operators简介
	Operators在消息发送者Observable和消息消费者Subscriber之间起到操纵消息的作用。例如
	map() operator,它可以用来将已被发送的消息转换成另一种形式。
	实例代码:
	Observable.just("xxx")
		.map(new Func1(String, String) {
			@Override
			public String call(String s) { 
				return s + "111";
			}
		})
		.subscribe(s -> Log.d(s));
		
	Java8的lambda表达式:	
	Observable.just("xxx")
		.map(s -> s + "111")
		.subscribe(s -> Log.d(s));
		
	Observable和Subscriber能完成任何事情: Observable可以是一个数据库查询，
	Subscriber获取数据结果并显示在页面上；Observable从网络上获取数据，Subscriber将其
	保存到本地；Observable可以是屏幕上某个控件的点击，Subscriber用来监听并响应这个点击
	事件。
	
	Observable和Subscriber与它们之间的一系列转换步骤是相互独立的:我们可以在
	Observable和Subscriber之间加入任意多个类似map()的operator方法，我们可以得到一个
	无穷尽的调用链。
	
----
<span id = "part24"/>
## 24. 依赖注入及其在Android中的应用


	作用: 降低代码耦合度
	Android平台函数库: ButterKnife、RoboGuice、Dagger和Dagger2
### 24.1 基本概念
	控制反转(IoC),面向对象编程的重要法则之一，旨在减小程序中不同部分的耦合问题。
	IoC分为两种类型:
		1.依赖注入(Dependency Injection) DI --使用广泛
		2.依赖查找(Dependency Lookp)
	例子:
	public class Car {
		private Engine mEngine;
		private Wheel mWheel;
		
		public Car() {
			mEngine = new PetrolEngine();
			mWheel = new MichelinWheel();
		}
	}
	在Car的构造函数中，需要自己创建Engine等实例。同时需要知道如何实现Engine。一旦
	Engine类型由Petrol改为Diesel，需要修改Car的构造函数。(耦合严重)
	
#### 24.1.1 构造函数注入
	public Car(Engine engine, Wheel wheel) {
		mEngine = engine;
		mWheel = wheel;
	}
#### 24.1.2 Setter函数注入
	public Car() {	}
	public void setEngine(Engine engine) {
		mEngine = engine;
	}
	public void setWheel(Wheel wheel) {
		mWheel = wheel;
	}
#### 24.1.3 接口注入
	public interface ICarInject {
		void bindEngine(Engine engine);
		void bindWheel(Wheel wheel);
	}
	public class Car implements ICarInject {
		public Car() {	}
		public void bindEngine(Engine engine) {
			mEngine = engine;
		}
		public void bindWheel(Wheel wheel) {
			mWheel = wheel;
		}
	}

### 24.2 开源框架的选择
#### 24.2.1 ButterKnife
	纯粹的View注入框架
[ButterKnife git地址](https://github.com/liu402883451/butterknife)<br/>
[Android Butter Knife 框架——最好用的View注入](https://www.jianshu.com/p/9ad21e548b69)

#### 24.2.2 RoboGuice
	反射机制，性能较差，代码入侵性大，包比较大。
[Android RoboGuice 使用指南](http://wiki.jikexueyuan.com/project/android-roboguice/)<br/>

#### 24.2.3 Dagger
	虽然实现编译时注入，还是使用到反射机制，会损耗一部分性能，调试困难。
	
#### 24.2.4 Dagger2
	与Dagger最大区别是对象图的验证，配置以及预先设置等方面抛弃了反射机制的使用，转而是在编
	译阶段完成，性能大大提升，由于没有使用反射，没有实现动态机制，缺乏灵活性。
[Dagger2 最清晰的使用教程](https://www.jianshu.com/p/24af4c102f62)<br/>

----	
<span id = "part25"/>
## 25.Android世界的Swift:Kotlin在Android中的应用

[kotlin-for-android-developers中文翻译在线阅读](https://github.com/wangjiegulu/kotlin-for-android-developers-zh/blob/master/SUMMARY.md)<br/>

[Kotlin 教程 | 菜鸟教程](http://www.runoob.com/kotlin/kotlin-tutorial.html)<br/>

[Kotlin 语言中文站](https://www.kotlincn.net/)<br/>

---

<span id = "part26"/>
## 26.Android在线热修复方案研究

	Robust(美团)是新一代热更新系统，无差别兼容Android2.3-8.0版本；无需重启补丁实时生
	效，快速修复线上问题，补丁修补成功率高达99.9%。(亲测可行)
[Robust git地址](https://github.com/liu402883451/Robust)<br/>

----

<span id = "part27"/>
## 27.面向切面编程及其在Android中的应用


	Hugo AOP框架
	在进行Android性能调优、减少应用卡顿时，寻找可优化的code是一个必要的过程。如何发现应用
	中的耗时任务甚至是耗时函数呢，如果可以在log中打印每个方法的执行时间，甚至把执行方法时
	的输入输出同时打印，绝对是非常棒的功能。
	幸运的是jake Wharton大神已经做出了这样的工具：Hugo。
[Hugo git地址](https://github.com/liu402883451/hugo)<br/>

----
<span id = "part28"/>
## 28.改造Android构建系统


	Freeline 由蚂蚁聚宝 Android 团队开发，它可以充分利用缓存文件，在几秒钟内迅速地对代
	码的改动进行编译并部署到设备上，有效地减少了日常开发中的大量重新编译与安装的耗时。
[Freeline git地址](https://github.com/liu402883451/freeline)<br/>

----

## 五、性能优化篇
<span id = "part29"/>
## 29. 代码优化

### 29.1 数据结构的选择
	例如: Android平台特有的稀疏数组的实现SparseArray，它是Integer到Object的一个映
	射，在特定场合可用于代替HashMap<Integer,<E>>,提高性能。它的核心实现是二分查找算法。
	new HashMap<Integer,Boolean>() ==> new SparseBooleanArray()
	new HashMap<Integer,Integer>() ==> new SparseIntArray()
	new HashMap<Integer,Long>() ==> new SparseLongArray()
	new HashMap<Integer,String>() ==> new SparseArray()
	在Android工程中运行Lint分析，会有一个名为AndroidLintUseSparseArrays的检查项，
	如果违规，他会提示。
	
### 29.2 Handler和内部类的正确用法
#### 29.2.1 错误用法及原因分析
	错误用法: 可能引入内存泄漏
	public class HandlerActivity extends Activity {
		@override
		public void handleMessage(Message msg){
			//...
		}
	}
	原因:Handler是和Looper以及MessageQueue一起工作，一个应用启动后，系统默认创建一个 
	为主线程服务的Looper对象，该Looper对象用于处理主线程所有的Message对象，它的生命周期
	贯穿于整个应用的生命周期。在主线程中使用的Handler默认都会绑定到这个Looper对象。这
	时发送到MessageQueue中的Message对象都会持有这个Handler对象的引用，这样在Looper
	处理消息时才能回调到Handler的handleMessage方法。因此，如果Message还没被处理完成
	。那么Handler对象不会被回收。
	
	Handler实例声明为Activity类的内部类，在java中，非静态匿名内部类会持有外
	部类的一个隐式的引用，这样会导致Activity无法被回收。
	
#### 29.2.2 解决方案
	1.在子线程中使用Handler
	2.将Handler声明为静态的内部类
	正确姿势 上代码:
	
	public class HandlerActivity extends Activity {
	
		private static class InnerHandler extends Handler {
			private final WeakReference<HandlerActivity> mActivity;
			public InnerHandler(HandlerActivity activity) {
				mActivity = new WeakReference<>(activity);
			}
			
			@override
			public void handleMessage(Message msg) {
				HandlerActivity activity = mActivity.get();
				if(activity != null) {
					//...
				}
			}
		}
		
		private final InnerHandler mHandler = new InnerHandler(this);
		
		private static final Runnable sRunnable = new Runnable() {
			@override
			public void run() {
				//...
			}
		}
		
		@override
		protected void onCreate(Bundle saveInstanceState) {
			super.onCreate(saveInstanceState);
			mHandler.postDelayed(sRunnable, 1000 * 60 * 5);
		}
	}

### 29.3 正确地使用Context
	错误使用Context导致的内存泄漏:
	典型例子:
	在单例模式中，在getInstance方法中传入Context，会产生内存泄漏
	原因:
	调用getInstance时传入的Context是一个Activity或者Service的实例，在应用退出之前，这个单例一直存在，会导致对应的Activity或者Service被单例引用，从而不会被垃圾回收。
	正确姿势:
	使用Application Context，因为它是应用唯一的，而且生命周期和应用一致。

### 29.4 掌握Java的四种引用方式
[Java强引用、软引用、弱引用、虚引用详解](https://blog.csdn.net/xiaofengcanyuexj/article/details/45271195?from=timeline)
	
### 29.5 其他代码微优化
	1. 避免创建非必要的对象--最好重用对象而不是每次需要的时候创建
	2. 对常量使用static final修饰
	3. 避免内部的Getters/Setters
	4. 代码的重构--<<代码整洁之道>>、<<重构-改善既有代码的设计>>
----
<span id = "part30"/>
## 30.图片优化


	1. 首选WebP格式，Android Stuido自带转换工具，mipmap文件夹右键 --> Convert to WebP
	2. 需要拉伸的图片使用NinePatch格式的PNG图

----
<span id = "part31"/>
## 31.布局优化

### 31.1 include标签共享布局
	将通用的布局抽取出来，独立成一个XML文件，然后在需要用到的页面中使用include标签引入进
	来。如:页面的标题栏
	<include layout="@layout/..."/>
### 31.2 ViewStub标签实现延迟加载
	使用场景:通常用于网络请求页面失败的显示。一般情况下若要实现一个网络请求失败的页面，我们
	是不是使用两个View呢，一个隐藏，一个显示。试想一下，如果网络状况良好，并不需要加载失败
	页面，但是此页面确确实实已经加载完了，无非只是隐藏看不见而已。如果使用ViewStub，在需
	要的时候才进行加载，不就达到节约内存提高性能的目的了吗？
[使用 ViewStub 提高加载性能](https://juejin.im/entry/5849184a128fe1005894d7b9)
### 31.3 merge标签减少布局层次
	merge标签使用方式:
	1. 代替最外层为<FrameLayout>标签
	2. 当前布局为另一个布局的子布局，使用<include>标签引入时，可以把当前布局的根布局替换
	成<merge>
### 31.4 尽量使用CompoundDrawable
	文本需要带一个小图标情况:使用一个TextView
	图片用drawableLeft/drawableRight/drawableTop/drawableBottom/
	drawablePadding属性代替
### 31.5 使用Lint
[Android-Lint工具使用](https://www.jianshu.com/p/010f895fe2cc)

----
<span id = "part32"/>
## 32.Android混淆机制


	包含: Java代码的混淆、Native代码的混淆、资源混淆
	
[Android Studio混淆模板及常用第三方混淆](https://blankj.com/2016/06/28/android-proguard-templet/?from=timeline)

---

<span id = "part33"/>
## 33.Android反编译机制详解

### 33.1 资源文件的反编译
	ApkTool
	命令: java -jar apktool.jar d xxx.apk
[ApkTool](https://bitbucket.org/iBotPeaches/apktool/downloads/)
### 33.2 Java代码的反编译
	dex2jar
	命令: sh d2j-dex2jar.sh -f xxx.apk 
	生成classes-dex2jar.jar文件
	jd-gui
	可视化界面，打开jd-gui工具，然后将生成的classes-dex2jar.jar文件拖进去，即可看见反
	编译的文件
[jd-gui下载地址](http://jd.benow.ca/)

[dex2jar git地址](https://github.com/liu402883451/dex2jar)

----	
<span id = "part34"/>
## 34.Android加固技术研究

[360加固下载](https://jiagu.360.cn/#/global/help/89)

[360加固如何使用](http://jiagu.360.cn/qcms/help.html#!id=88)

----
<span id = "part35"/>
## 35.Android安全编码

### 35.1 WebView远程代码执行
	Android4.2之前，使用WebView.addJavascriptInterface方法实现通过Javascript调用
	本地Java接口时，攻击者可以通过反射执行任意Java对象的方法。
	解决方案:
	API Level > 16,被调用的本地Java函数必须使用@JavascriptInterface注解进行标记
	API Level < 16,不使用addJavascriptInterface接口，转而使用safe-java-webview-
	bridge
[git地址: safe-java-webview-bridge](https://github.com/liu402883451/safe-java-js-webview-bridge)
### 35.2 WebView密码明文保存
	mWebView.getSettings().setSavePassword(false);//已废弃，API 18(4.3)之前使用，之后谷歌已经修复该漏洞。
### 35.3 Android本地拒绝服务
	Android提供了Intent机制实现四大组件间的交互和通信，本地拒绝服务的根本原因在于
	Intent.getXXXExtra()系列函数。
	攻击者可以通过构造自定义序列化对象，并通过Intent.putXXXExtra()函数设置给Intent，使用该Intent调起对应的组件时，发生异常导致APP发生Crash。
	1. 非法序列化对象导致的ClassNotFuoundException
		漏洞代码:
		Intent intent = getIntent();
		intent.getSerializableExtra("xxx");
		攻击代码:
		Intent intent = new Intent();
		intent.putComponent(new ComponentName(yourPackageName,yourClassName));
		intent.putExtra("xxx",new ThridPartyObject());
		startActivity(intent);
		解决方案:
		在intent.getSerializableExtra("xxx");外try-catch
	
	2. 空Action导致的NullPointException
		漏洞代码:
		Intent intent = getIntent();
		if(intent.getAction().equals("android.intent.action.xxx")){
			//TODO
		}
		攻击代码:
		Intent intent = new Intent();
		intent.putComponent(new ComponentName(yourPackageName,yourClassName));
		startActivity(intent);
		解决方案:
		除了对getAction返回值进行判空外，养成比较时将常量写在前面的好习惯，其实
		不判空也不会出现问题。
	
	3. 强制类型转换导致的ClassCastException
		漏洞代码:
		User user = (User) intent.getSerializableExtra("xxx");
		攻击代码:
		Intent intent = new Intent();
		intent.putComponent(new ComponentName(yourPackageName,yourClassName));
		intent.putExtra("xxx",10);
		解决方案:
		不啰嗦，try-catch
	4. 数组越界导致的IndexOutOfBoundsException
		Intent.getXXXArrayExtra系列时没有对数组边界做判断。
	
### 35.4 SharedPreference全局任意读写
		SharedPreference指定模式:
		Context.MODE_PRIVATE:私有读写，只有自己的APP可以访问
		Context.MODE_WORLD_READBLE:全局可读，其他APP也可以读取
		Context.MODE_WORLD_WRITEBLE:全局可写，其他APP也可以写入
		正确做法是使用Context.MODE_PRIVATE
		
### 35.5 密钥硬编码
		用于敏感数据传输和本地数据加密的密钥，不能硬编码写在代码中
### 35.6 AES/DES/RSA弱加密
		对称加密:
		DES是一种弱对称加密算法，一般不建议在商业产品中使用
		AES更安全
		AES使用不安全的加密模式:AES的ECB模式是不安全的。
		非对称加密:
		RSA加密时没有使用padding的话，no padding方式的RAS更容易受到攻击
		RSA加密，密钥长度小于512bit，是不安全的。
		Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding")//安全
		Cipher cipher = Cipher.getInstance("RSA/ECB/NoPadding")//不安全
		
[加密算法对比](https://blog.csdn.net/u013718120/article/details/56486408)

[八种加密的工具类git地址](https://github.com/songxiaoliang/EncryptionLib)

### 35.7 WebView忽略SSL证书
	漏洞代码:
	mWebView.setWebViewClient(new WebViewClient(){
            @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
                handler.cancel();
            }
        });
    正确姿势:使用handler.proceed();
### 35.8 HTTPS证书弱校验
	1.自定义X509TrustManager未实现安全校验
	2.自定义HostnameVerifier默认接受所有域名
	3.SSLSocketFactory信任所有证书
[retrofit支持https](https://www.jianshu.com/p/bd3477f6f5f9)
### 35.9 PendingIntent使用不当
	PendingIntent中使用显式的Intent，不使用隐式的Intent
----
	
# 工具篇
<span id = "part36"/>
## 36.Android调试工具Facebook Stetho

[Stetho git地址](https://github.com/liu402883451/stetho)

[借助Stetho在Chrome上调试Android网络&数据库](https://www.jianshu.com/p/03da9f91f41f)	

[Android 调试神器-Stetho(Facebook出品)的使用](https://blog.csdn.net/sbsujjbcy/article/details/45420475)	

---
<span id = "part37"/>
## 37.内存泄漏检查函数库LeakCanary

[LeakCanary git地址](https://github.com/liu402883451/leakcanary)

[LeakCanary 中文使用说明](https://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)

---

<span id = "part38"/>
## 38.基于Facebook Redex实现Android APK的压缩和优化

[Redex git地址](https://github.com/liu402883451/redex)

[Redex 简书](https://www.jianshu.com/p/78ad578251ef)

[mac os cmake安装](https://blog.csdn.net/eli00001/article/details/40082083)

<span id = "part39"/>
## 39.Android Studio你所需要知道的功能

### 39.1 Annotate
	代码中右键--> Git --> Annotate
	可以查看代码变更记录，谁改的，改了什么
### 39.2 .ignore插件
	project右键--> New --> .ignore file生成
	git提交忽略文件配置
### 39.3 Live Templates
	代码模板:Preferences --> Editor --> Live Templates
	栗子: findViewById(R.id.);
----



###END
