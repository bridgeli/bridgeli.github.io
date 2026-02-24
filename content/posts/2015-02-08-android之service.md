---
title: Android之Service
author: Bridge Li
type: post
date: 2015-02-08T15:13:08+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Android
tags:
  - Service
  - 四大组件

---
今天记录一下老夫对Service的理解，先看一下Service的概念，即Service是什么不是什么，那Service是什么呢？  
1. Service是Android的四大组件之一，可以长时间在后台运行；  
2. Service不提供界面交互，即Service不像activity一样，有一个界面做展示；  
3. 即便用户跳转至另一个应用后，Service仍旧在后台运行；  
4. 任意应用组件都可以绑定一个服务，甚至可以用来完成进程间通讯的任务；  
5. 可以使用Service更新ContentProvider，发送Intent以及启动系统的通知等等；

那Service不是什么呢？  
1. Service不是一个单独的进程；  
2. Service不是一个线程！！！（即Service运行于主线程中，根据Service的概念，这个应该有很多人怀疑，持怀疑态度的可以打一下线程号，比较简单，老夫就不多做赘述了）

看完Service是什么和不是什么之后，我们来看看什么时候需要用Service呢？Service一般是在后台做一些费力费时的任务（老夫窃以为和西游记中的沙僧差不多，默默无闻，任劳任怨），例如：下载文件、播放音乐、文件I/O等。

既然在这些时候需要用Service，那么我们怎么启动一个Service呢？启动一个Service用两种方式，分别是startService()和bindService()，那么他们之间又有什么区别和应用于什么场合呢？

首先来看startService()，一旦某个组件start一个Service后，Service开始独立运行，不在与原来的组件产生任何关系，如果想要停止Service，必须手动停止，所以其适用于开启下载、播放音乐之类的；

下面我们再看看bindService()，某个组件bind一个Service后，Service为组件提供一个接口，近似于客户端，会进行交互，当所有bind的组件都结束后，Service会自动停止，有很多系统服务都被系统封装了，例如传感器、定位等。

看了一大堆理论之后，我们来看看怎么启动一个service：

一、startService()，代码如下：

MainActivity用来启动一个Service，

```  
package cn.bridgeli.demo;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity implements OnClickListener {

    private Button start = null;
    private Button stop = null;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        start = (Button) findViewById(R.id.start);
        start.setText("Start Service");
        start.setOnClickListener(this);

        stop = (Button) findViewById(R.id.stop);
        stop.setText("Stop Service");
        stop.setOnClickListener(this);

    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.start:
                Intent startService = new Intent();
                startService.setClass(MainActivity.this, MainService.class);
                startService(startService);
                break;

            case R.id.stop:
                Intent stopService = new Intent();
                stopService.setClass(MainActivity.this, MainService.class);
                stopService(stopService);

                break;

            default:
                break;
        }
    }
} 
```

对应的布局文件activity_main：  
```  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
xmlns:tools="http://schemas.android.com/tools"  
android:layout_width="match_parent"  
android:layout_height="match_parent"  
android:orientation="vertical"  
android:paddingBottom="@dimen/activity_vertical_margin"  
android:paddingLeft="@dimen/activity_horizontal_margin"  
android:paddingRight="@dimen/activity_horizontal_margin"  
android:paddingTop="@dimen/activity_vertical_margin"  
tools:context="cn.bridgeli.demo.MainActivity" >

<Button  
android:id="@+id/start"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"/>

<Button  
android:id="@+id/stop"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"/>

</LinearLayout>  
```

MainService被启动的Service：

```  
package cn.bridgeli.demo;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;

public class MainService extends Service {

    @Override
    public void onCreate() {
        super.onCreate();
        System.out.println("onCreate");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        System.out.println("onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public IBinder onBind(Intent arg0) {
        System.out.println("onBind");
        return null;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        System.out.println("onDestroy");
    }
}  
```

需要说明的组件都需要在AndroidMainifest.xml中进行注册：

```  
<?xml version="1.0" encoding="utf-8"?>  
<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
package="cn.bridgeli.demo"  
android:versionCode="1"  
android:versionName="1.0" >

<uses-sdk  
android:minSdkVersion="8"  
android:targetSdkVersion="21" />

<application  
android:allowBackup="true"  
android:icon="@drawable/ic_launcher"  
android:label="@string/app_name"  
android:theme="@style/AppTheme" >  
<activity  
android:name=".MainActivity"  
android:label="@string/app_name" >  
<intent-filter>  
<action android:name="android.intent.action.MAIN" />

<category android:name="android.intent.category.LAUNCHER" />  
</intent-filter>  
</activity>

<service android:name=".MainService" ></service>  
</application>

</manifest>  
```

当我们第一次点击Start Service时，Service中的onCreate()方法和onStartCommand()方法都会被调用，onCreate()方法一般用来做一些准备数据的工作，onStartCommand()方法一般就用来执行Service的逻辑了，当我们以后再点击Start Service时，因为Service已经被创建，所以只有onStartCommand()方法被调用了（大家可以自己做实验，验证），当我们点击Stop Service时，onDestroy()方法就会被调用，相信大家一看名字就知道了这个方法是干嘛的了，如果我们此时，再点击Start Service，就相当于第一次点击Start Service；那么另外一个方法什么时候调用呢？那我们就来接着看启动Service的第二种方法。

二、bindService()，代码如下：  
MainActivity同样是用来启动Service

```  
package cn.bridgeli.demo1;

import android.app.Activity;
import android.content.ComponentName;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity implements OnClickListener, ServiceConnection {

    private Button start = null;
    private Button stop = null;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        start = (Button) findViewById(R.id.start);
        start.setText("Start Service");
        start.setOnClickListener(this);

        stop = (Button) findViewById(R.id.stop);
        stop.setText("Stop Service");
        stop.setOnClickListener(this);

    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.start:
                Intent bindService = new Intent();
                bindService.setClass(MainActivity.this, MainService.class);
                bindService(bindService, this, BIND_AUTO_CREATE);
                break;

            case R.id.stop:
                Intent unbindService = new Intent();
                unbindService.setClass(MainActivity.this, MainService.class);
                unbindService(this);

                break;

            default:
                break;
        }
    }

    @Override
    public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
    }

    @Override
    public void onServiceDisconnected(ComponentName arg0) {

    }

}  
```

布局文件和上一个例子一样，就不在重复了，被启动的Service：

```  
package cn.bridgeli.demo1;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;

public class MainService extends Service {
    private MyBinder myBinder = new MyBinder();

    @Override
    public void onCreate() {
        super.onCreate();
        System.out.println("onCreate");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        System.out.println("onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public IBinder onBind(Intent arg0) {
        System.out.println("onBind");
        return myBinder;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        System.out.println("onDestroy");
    }

    class MyBinder extends Binder {

    }
}  
```

注册Service也一样，也不多重复了，这个Service和上一个Service差不多，唯一的区别就是onBind()方法返回了一个IBinder对象，那么这个对象返回给谁了呢？我们仔细看一看MainActivity，会发现其多实现了一个接口：ServiceConnection，这个接口有两个方法：onServiceConnected()和onServiceDisconnected()，相信大家一看这两个方法名就知道他们用来做什么了，其中onServiceConnected()有一个参数Ibinder，这个参数就是onBind()的返回值，我们可以通过强转，把其转化为我们需要的类型，然后调用其方法，就是本例中中的MyBinder中任意方法，最后在activity中展示出来就行了，至于另一个方法相信不用老夫多赘述啦，大家做一个实验即知其用法。

PS：Service作为Android的四大组件之一，用法肯定不会是如此之简单，老夫是第一次写Android的东西，既是初学还是自学，也没做过项目，所以不免有错误的地方和有点抓不住重点，还请大家能留言交流，谢谢

**2015-02-09 10:10补记，**  
经朋友George提醒，Service既然既不是一个单独的进程，也不是一个线程，如果实际开发中这么用的话，例如下载，会导致UI阻塞，有可能会报：ANR (“Application Not Responding”)，意思是“应用没有响应”，这显然不是我们想要的，一个下载操作应该在后台让他默默下载，不影响UI，解决这个问题老夫窃以为其实很简单，开线程吗，也就是说新开一个线程用于下载等操作，不影响主线程的UI交互，那么开线程有几种方法呢？  
1. 继承Thread类；  
2. 实现Runnable接口；  
3. 使用线程池java.util.concurrent.ExecutorService。  
至于是使用继承还是实现接口以及怎么用，这个相信会Java的都会知道(不知道的请用谷歌百度一下)，老夫不在赘述，这里说一下ExecutorService线程池，其初始化可以通过executorService = java.util.concurrent.Executors.newCachedThreadPool();初始化，初始化之后我们就可以在后面的代码中直接：

```  
mExecutorService.execute(new Runnable() {
    @Override
    public void run() {

    }
});  
```

然后在run()方法中对业务逻辑进行处理，和继承Thread类或者实现Runnable接口没啥两样。

最后在说明一点，继承android.os.Handler类，重写其dispatchMessage()方法好像也可以处理UI的返回，这个因为还没有学到，暂且存疑，等将来学到了，会专门写一篇文章讲Handler怎么用。

最后的最后感谢一下George同学的指正，谢谢