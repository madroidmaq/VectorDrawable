# Android的矢量图支持 - VectorDrawable

## 简介
从Lollipop(Android 5.0)开始, Android引入了对矢量图的支持, 但并不支持svg这种矢量图片格式, 而是以VectorDrawable的方式来实现矢量图的效果。
VectorDrawable也是Drawable的一个直接子类, 像其它Drawable那样通常情况下是在xml中定义, 它对应的xml标签是\<vector/\>, 基本结构如下:

	<vector xmlns:android="http://schemas.android.com/apk/res/android"
	        android:width="24dp"
	        android:height="24dp"
	        android:viewportWidth="24.0"
	        android:viewportHeight="24.0">
	    <path
	        android:fillColor="#FF000000"
	        android:pathData="M6,18c0,0.55 0.45,1 1,1h1v3.5c0,0.83 0.67,1.5 1.5,1.5s1.5,-0.67 1.5,-1.5L11,19h2v3.5c0,0.83 0.67,1.5 1.5,1.5s1.5,
	-0.67 1.5,-1.5L16,19h1c0.55,0 1,-0.45 1,-1L18,8L6,8v10zM3.5,8C2.67,8 2,8.67 2,9.5v7c0,0.83 0.67,1.5 1.5,1.5S5,17.33 5,16.5v-7C5,
	8.67 4.33,8 3.5,8zM20.5,8c-0.83,0 -1.5,0.67 -1.5,1.5v7c0,0.83 0.67,1.5 1.5,1.5s1.5,-0.67 1.5,-1.5v-7c0,-0.83 -0.67,-1.5 -1.5,-1.5zM15.53,
	2.16l1.3,-1.3c0.2,-0.2 0.2,-0.51 0,-0.71 -0.2,-0.2 -0.51,-0.2 -0.71,0l-1.48,1.48C13.85,1.23 12.95,1 12,1c-0.96,0 -1.86,0.23 -2.66,
	0.63L7.85,0.15c-0.2,-0.2 -0.51,-0.2 -0.71,0 -0.2,0.2 -0.2,0.51 0,0.71l1.31,1.31C6.97,3.26 6,5.01 6,7h12c0,-1.99 -0.97,-3.75 -2.47,
	-4.84zM10,5L9,5L9,4h1v1zM15,5h-1L14,4h1v1z"/>
	</vector>

上图表示的就是一个Android小机器人，具体效果看下面的效果图。其中有以下几点需要注意

* vector是VectorDrawable对应的根标签
* `android:width`与`android:height`对应矢量图的实际大小(矢量图是可以无限大, 但通常情况下一个图片都应该有一个原始大小, 假如你将此VectorDrawable作为一个ImageView的src, ImageView的大小都设置为`wrap_content`, 则ImageView对应的实际大小就是这里设置的大小)
* android:viewportWidth与android:viewportHeight是指当前Drawable对应的虚拟Canvas的大小, 之所以说是虚拟的是因为实际上并不存在这样一个Canvas, 又之所以需要这个值是因为在\<path/\>标签中的路径数据要基于具体的坐标系来绘制
* \<path/\>标签对应路径信息, 这里的path与我们自定义绘制图形时用的Path原理一样, 就是记录一些绘图操作, 具体对应其中的pathData.PathData中对应的路径描述符号不需要我们去记, 通常情况下由绘图软件生成svg图片再从svg文件中提取.

----
## 如何使用
在Android5.0及以上版本中，使用向量图和普通的drawable资源并没有什么差别，如下

		<LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:orientation="horizontal">

	        <ImageView
	            android:layout_width="50dp"
	            android:layout_height="50dp"
	            android:src="@drawable/ic_android_black_24dp" />

	        <ImageView
	            android:layout_width="50dp"
	            android:layout_height="50dp"
	            android:tint="@color/colorAccent"
	            android:src="@drawable/ic_android_black_24dp" />

	        <View
	            android:layout_width="50dp"
	            android:layout_height="50dp"
	            android:tint="@color/colorAccent"
	            android:background="@drawable/ic_android_black_24dp"
	            android:id="@+id/view" />

	        <TextView
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:text="Hello World!"
	            android:gravity="center"
	            android:tint="@color/colorAccent"
	            android:drawableStart="@drawable/ic_android_black_24dp" />

	    </LinearLayout>

![pic2 向量图效果](https://github.com/MaAnQing/VectorDrawable/blob/master/pic/VectorDrawable.png "向量图效果")
由上可知，VectorDrawable不仅支持普通的src属性，还支持tint、background、drawableStart

最低Android版本4.3 最高版本7.1，在Android4.4设备上运行以上代码代码发现可以正常运行不会出现crash，VectorDrawable明明是Android5.0以后才支持的，为什么在4.4上也可以正常运行呐，查找资料才知道，当minSdkVersion小于5.0时，gradle编译的时候会把VectorDrawable生成相应的位图资源打包在APP里（VectorDrawable并不会打包在app里），运行时都会引用位图资源。当minSdkVersion大于等于5.0时会直接引用VectorDrawable。反编译App查看生成位图的位置，其包含在mdpi,hdpi,xhdpi,xxhdpi,xxxhdpi中，当然也可以指定生成特定分辨率的位图资源，需要在gradle文件中配置，如下

	defaultConfig {
	    vectorDrawables.generatedDensities = ['hdpi','xxhdpi']
	}

这样在minSdkVersion小于5.0会存在一个问题，就是app中包含不同分辨率的位图资源这会导致app体积增大。有没有什么办法可以让低版本设备直接支持VectorDrawable而不是生产响应的位图资源呐，答案是肯定的。

----
## 在低版本中支持向量图

* 配置使用SupportLibrary

首先需要在你的build.gradle配置文件中增加如下配置

	android {
	    defaultConfig {
	        vectorDrawables.useSupportLibrary = true
	    }
	}

上面的配置的作用是强制gradle在编译时不自动生成兼容低版本的位图资源.

* 添加SupportLibrary依赖

其次需要添加23.2.0 以后的Support Library，如下

	compile 'com.android.support:appcompat-v7:23.2.0'

这里有一个坑，就是在23.2.0中首次发布了这个功能，随后发现在内存和配置更新中存在问题，于是就在23.3.0中去除了这个功能，在随后的一个版本23.4.0中又修复了这个问题，但是需要在代码中手动设置一个flag。原文如下：

> First up, this functionality was originally released in 23.2.0, but then we found some memory usage and Configuration updating issues so we it removed in 23.3.0. In 23.4.0 (technically a fix release) we’ve re-added the same functionality but behind a flag which you need to manually enable.

所以应添加23.4.0及以上的Support依赖

	compile 'com.android.support:appcompat-v7:23.4.0'
并在Activity的前面添加flag，开启该功能

	static {
	    AppCompatDelegate.setCompatVectorFromResourcesEnabled(true);
	}

* 代码引用

最后就是如何使用了，ImageView在引用VectorDrawable资源时使用 `app:srcCompat` 取代 `android:src`，或者使用`android:src`引用一个`selector`资源；若想在`background`中使用`Vectordrawable`资源用作View的背景，并没有一个类似`app:backgroundCompat`方法，这里可以使用`selector`包裹一层，当然也可以在代码中设置

	Resources resources = context.getResources(Resources, int, Theme);
	Theme theme = context.getTheme();
	Drawable drawable = VectorDrawableCompat.create(resources, R.drawable.vector_drawable, theme);
	view.setBackground(drawable);

注：上面代码只是明确知道你所要加载的resource是VectoreDrable，若是在不知道自己加载是普通的位图资源还是矢量图，上面写的就会抛出NotFoundException异常。这也很好理解，当你要创建矢量图的时候发现取到的资源并不是矢量图而是普通的位图资源，所以就会抛出异常。我在`VectorDrawableCompat `中并没有找到判断`drawable id`是否为矢量的方法，不过这里有一个取巧的办法，那就是捕获异常，如下


	    Drawable drawable ;
	    try {
	        drawable = ContextCompat.getDrawable(context, R.drawable.icon);
	    } catch (Resources.NotFoundException e) {
	        drawable = VectorDrawableCompat.create(context, item.icon, context.getTheme()) ;
	    }

若有更好的办法，请提出来。

----
## Show me code
首先是布局文件，如下

		<LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:orientation="horizontal">

	        <ImageView
	            android:layout_width="50dp"
	            android:layout_height="50dp"
	            app:srcCompat="@drawable/ic_android_black_24dp"
	            />

	        <ImageView
	            android:layout_width="50dp"
	            android:layout_height="50dp"
	            android:tint="@color/colorAccent"
	            android:src="@drawable/selector"
	            />

	        <View
	            android:layout_width="50dp"
	            android:layout_height="50dp"
	            android:id="@+id/view" />

	        <TextView
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:text="Hello World!"
	            android:gravity="center"
	            android:drawableStart="@drawable/selector"
	            />

	    </LinearLayout>

其中包含的selector资源如下

	<?xml version="1.0" encoding="utf-8"?>
	<selector xmlns:android="http://schemas.android.com/apk/res/android">
	    <item android:drawable="@drawable/ic_android_black_24dp" android:state_pressed="true" />
	    <item android:drawable="@drawable/ic_android_black_24dp" />
	</selector>

Activity代码如下

	public class MainActivity extends AppCompatActivity {

	    static {
	        AppCompatDelegate.setCompatVectorFromResourcesEnabled(true);
	    }

	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);

	        Drawable drawable = VectorDrawableCompat.create(getResources(), R.drawable.ic_android_black_24dp, getTheme());
	        findViewById(R.id.view).setBackground(drawable);

	    }
	}

以上效果和上图一样

reference：
[https://developer.android.com/studio/write/vector-asset-studio.html#delete](https://developer.android.com/studio/write/vector-asset-studio.html#delete)
[https://medium.com/@chrisbanes/appcompat-v23-2-age-of-the-vectors-91cbafa87c88#.2lyh3ubze](https://medium.com/@chrisbanes/appcompat-v23-2-age-of-the-vectors-91cbafa87c88#.2lyh3ubze)
[http://www.jianshu.com/p/e3614e7abc03?hmsr=toutiao.io&utm\_medium=toutiao.io&utm\_source=toutiao.io](http://www.jianshu.com/p/e3614e7abc03?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)