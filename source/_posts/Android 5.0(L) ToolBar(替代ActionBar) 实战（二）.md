title: "Android 5.0(L) ToolBar(替代ActionBar) 实战（二）"
date: 2014-10-31 17:16:25
tags:
categories: Android开发

---

上一次我们说到ToolBar的创建，这次我们来讨论一下ToolBar结合ViewPager实现类似微信的左右滑屏多个界面的效果。  
要实现滑屏，首先我们要到Android开发者官网去下载一个示例代码，它封装了Viewpager和Tab配合切换页卡的方法。当然，我们也可以自己来实现，为了节约时间成本，就直接拿来用了。如果在官网找不到或者无奈被墙，可以到这里下载：http://download.csdn.net/detail、wh1990xiao2005/8105345  
把下载好的工具类复制到项目指定的包内，我是在之前的项目中创建了view专用的包，并将这两个类复制了进去。
![](http://i.imgur.com/E4M1Y6k.png)  
这里要注意，复制进去之后有可能会报错，因为包名不符，只需把package包名修改为自己的类所在包名即可。  
打开布局文件，在上次Toolbar的声明下面添加SlidingTabLayout和ViewPager控件，下面放上示例：  

    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
   	xmlns:tools="http://schemas.android.com/tools"  
    xmlns:app="http://schemas.android.com/apk/res/com.xwh.toolbardemo"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    tools:context="com.xwh.toolbardemo.MainActivity" >  
  
    	<android.support.v7.widget.Toolbar  
        	android:id="@+id/demo_toolbar"  
        	android:layout_width="fill_parent"  
        	android:layout_height="wrap_content"  
        	android:background="?attr/colorPrimary"  
        	android:minHeight="?attr/actionBarSize"  
        	app:title="@string/hello_world" />  
  
    	<com.xwh.toolbardemo.view.SlidingTabLayout  
       		android:id="@+id/demo_tab"  
        	android:layout_width="fill_parent"  
        	android:layout_height="wrap_content"  
        	android:layout_below="@+id/demo_toolbar" />  
  
    	<android.support.v4.view.ViewPager  
        	android:id="@+id/demo_vp"  
        	android:layout_width="fill_parent"  
        	android:layout_height="fill_parent"  
        	android:layout_below="@+id/demo_tab" />  
	</RelativeLayout>  

下面打开MainActivity.java，实例化相关的对象，包括ToolBar，这里不多说废话了，直接上代码：  

     private void findView() {  
        toolbar = (Toolbar) findViewById(R.id.demo_toolbar);  
        slidingTabLayout = (SlidingTabLayout) findViewById(R.id.demo_tab);  
        viewPager = (ViewPager) findViewById(R.id.demo_vp);  
    }  

ToolBar不用多说，SlidingTabLayout实际上是上方的Tab视图，而ViewPager是页卡的容器，而页卡是稍后添加的多个Fragment。  
下面我们来写三个Layout的xml文件和相关的三个Fragment的Java代码，分别代表三个页卡。  
第一个页卡随意放置一个图片，代码如下：  

    <?xml version="1.0" encoding="utf-8"?>  
	<ImageView xmlns:android="http://schemas.android.com/apk/res/android"  
    	android:layout_width="match_parent"  
   		android:layout_height="match_parent"  
    	android:contentDescription="@string/app_name"  
    	android:src="@drawable/ic_launcher" >  
	</ImageView>  

第二个页卡放置一个模拟时钟，代码如下：

    <?xml version="1.0" encoding="utf-8"?>  
	<AnalogClock xmlns:android="http://schemas.android.com/apk/res/android"  
    	android:layout_width="match_parent"  
    	android:layout_height="match_parent" >  
	</AnalogClock>  

第三个页卡放置一个数字时钟，代码如下：

    <?xml version="1.0" encoding="utf-8"?>  
	<DigitalClock xmlns:android="http://schemas.android.com/apk/res/android"  
    	android:layout_width="match_parent"  
    	android:layout_height="match_parent"  
    	android:gravity="center"  
    	android:textSize="40sp" >  
	</DigitalClock>  

相应的，创建三个Fragment的Java代码文件，这里只列出其中一个文件的程序清单，另外两个按此自行编写：  

    package com.xwh.toolbardemo.fragment;  
  
	import com.xwh.toolbardemo.R;  
	import android.os.Bundle;  
	import android.support.v4.app.Fragment;  
	import android.view.LayoutInflater;  
	import android.view.View;  
	import android.view.ViewGroup;  
  
	public class Fragment_Tab_1 extends Fragment {  
	    @Override  
	    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {  
	        return inflater.inflate(R.layout.fragment_tab_1, container, false);  
	    }  
	}  

下面我们还需一个适配器（Adapter），用于将这三个Fragment适配到ViewPager之上，下面列出适配器的Java程序代码清单：  
    package com.xwh.toolbardemo.adapter;  
  
	import java.util.ArrayList;  
	import android.support.v4.app.Fragment;  
	import android.support.v4.app.FragmentManager;  
	import android.support.v4.app.FragmentPagerAdapter;  
  
	public class ViewPager_Adapter extends FragmentPagerAdapter {  
  
    	private ArrayList<Fragment> fragments;  
  
    	public ViewPager_Adapter(FragmentManager fm, ArrayList<Fragment> fragments) {  
    	    super(fm);  
    	    this.fragments = fragments;  
	    }  
  
    	@Override  
    	public Fragment getItem(int pos) {  
        	return fragments.get(pos);  
    	}  
  
    	@Override  
    	public int getCount() {  
	        return fragments.size();  
	    }  
	  
	    @Override  
	    public CharSequence getPageTitle(int position) {  
        		return "Tab_" + position;  
    		}  
		}  

这里需要注意：下方的getPageTitle()方法是必要的，他用于相应Tab名称的显示。  
到此，我的程序架构如下图所示：

![](http://i.imgur.com/fbT3xlx.png)

接下来，我们将三个Fragment通过Adapter适配到ViewPager上，这一过程，我们只对MainActivity做操作。完整代码如下所示：  

    package com.xwh.toolbardemo;  
  
	import java.util.ArrayList;  
	import com.xwh.toolbardemo.adapter.ViewPager_Adapter;  
	import com.xwh.toolbardemo.fragment.Fragment_Tab_1;  
	import com.xwh.toolbardemo.fragment.Fragment_Tab_2;  
	import com.xwh.toolbardemo.fragment.Fragment_Tab_3;  
	import com.xwh.toolbardemo.view.SlidingTabLayout;  
	import android.support.v4.app.Fragment;  
	import android.support.v4.view.ViewPager;  
	import android.support.v7.app.ActionBarActivity;  
	import android.support.v7.widget.Toolbar;  
	import android.os.Bundle;  
  
	public class MainActivity extends ActionBarActivity {  
  
    	private Toolbar toolbar;  
	    private SlidingTabLayout slidingTabLayout;  
	    private ViewPager viewPager;  
	  
	    private ArrayList<Fragment> fragments;  
	    private ViewPager_Adapter viewPager_Adapter;  
	  
	    @Override  
	    protected void onCreate(Bundle savedInstanceState) {  
        	super.onCreate(savedInstanceState);  
        	setContentView(R.layout.activity_main);  
        	findView();  
    	}  
	  
	    // 初始化控件  
	    private void findView() {  
        	// 实例化控件  
        	toolbar = (Toolbar) findViewById(R.id.demo_toolbar);  
	        slidingTabLayout = (SlidingTabLayout) findViewById(R.id.demo_tab);  
	        viewPager = (ViewPager) findViewById(R.id.demo_vp);  
	        // 设置ViewPager  
	        fragments = new ArrayList<Fragment>();  
        	fragments.add(new Fragment_Tab_1());  
        	fragments.add(new Fragment_Tab_2());  
        	fragments.add(new Fragment_Tab_3());  
        	viewPager_Adapter = new ViewPager_Adapter(getSupportFragmentManager(),  
                fragments);  
	        viewPager.setOffscreenPageLimit(fragments.size());  
	        viewPager.setAdapter(viewPager_Adapter);  
        	// 设置SlidingTab  
        	slidingTabLayout.setViewPager(viewPager);  
    	}  
	}  

到此，我们可以Ctrl+F11运行看效果了。如果不出意外的话，会是下面的效果：  

![](http://i.imgur.com/2aYEMQC.png)