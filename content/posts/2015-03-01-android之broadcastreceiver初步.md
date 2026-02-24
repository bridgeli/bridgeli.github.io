---
title: Android之BroadcastReceiver初步
author: Bridge Li
type: post
date: 2015-03-01T14:26:01+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Android
tags:
  - BroadcastReceiver
  - 广播

---
今天接着写自己学习Android开发的笔记，这次记录一下BroadcastReceiver，看这个名字我们就知道他是干嘛的了，广播接收器吗，那么他有什么用呢？老夫以为用途还是比较大的，例如用户玩游戏的时候，我们必须在监听到手机来电事件、短信事件之后，暂停游戏保存游戏当时的数据，等用户接完电话、处理完短信之后再接着玩游戏。  
之前我曾经说过，Android的四大组件都需要注册，方能使用，那么broadcast作为四大组件之一，也肯定需要注册，不同于其他组件的是broadcast有两种注册方法：  
1，在AndroidManifest.xml中进行注册；  
2.在代码中进行注册  
既然有这两种注册方式，那么他们肯定会有区别，他们的区别又是什么呢？  
在AndroidManifest.xml中进行注册，属于全局性的，也就是说无论你这个应用是否在运行，只要有某一他监听的广播被发出，那么他都会被监听到，这个的典型应用就是手机的黑名单功能，无论这个应用是否在运行，应该都可以监听用户手机的来电，进行过滤；而在代码中进行注册呢？肯定就不是全局的了，只有你这个应用启动的时候，他才会监听他所监听的事件，这个的典型应用就是，用于更新应用的UI，在activity启动的时候注册BroadcastReceiver，在activity不可见之后取消注册，因为activity不可见的时候更新UI，除了浪费CPU浪费电之外没有任何意义，好，下面我们就看看这两种方法的实现。

1，在AndroidManifest.xml中进行注册

```  
package cn.bridgeli.demo;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity implements OnClickListener {

    private Button send = null;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        send = (Button) findViewById(R.id.send);
        send.setText("Send Broadcast");
        send.setOnClickListener(this);

    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.start:
                Intent intent = new Intent();
                intent.setAction(Intent.ACTION_EDIT);
                sendBroadcast(intent);
                break;

            default:
                break;
        }
    }
}  
```

对应的布局文件其实很简单

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
android:id="@+id/send"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"/>

</LinearLayout>  
```

我们的BroadcastReceiver代码如下：

```  
package cn.bridgeli.demo;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;

public class MyBroadcastReceiver extends BroadcastReceiver {

    public MyBroadcastReceiver() {
        super();
        System.out.println("MyBroadcastReceiver");
    }

    @Override
    public void onReceive(Context arg0, Intent arg1) {
        System.out.println("onReceive");
    }

}  
```

下面就是在AndroidManifest.xml中注册的代码，也是最为关键的代码

```  
<receiver android:name=".MyBroadcastReceiver">  
  <intent-filter >  
    <action android:name="android.intent.action.EDIT"/>  
  </intent-filter>  
</receiver>  
```

需要说明的是，intent的action一定要和注册代码中的action相匹配，即为这个receive就监听这个事件，这个发送的广播就是这个事件。

另外大家会发现我在receive中增加了一个构造方法，大家可以发现当你多次点击send broadcast时，构造方法会被多次调用，这就说明当onReceive()方法结束之后，receive的生命周期也就结束了，这和其他的组件的生命周期略有不同

2.在代码中进行注册

```  
package cn.bridgeli.demo;

import android.app.Activity;
import android.content.IntentFilter;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;

public class MainActivity extends Activity implements OnClickListener {

    private Button register = null;
    private Button unRegister = null;
    private MyBroadcastReceiver myBroadcastReceiver = null;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        register = (Button) findViewById(R.id.register);
        register.setText("Register Broadcast");
        register.setOnClickListener(this);
        unRegister = (Button) findViewById(R.id.unRegister);
        unRegister.setText("unRegister Broadcast");
        unRegister.setOnClickListener(this);

    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.register:
                IntentFilter filter = new IntentFilter();
                filter.addAction("android.provider.Telephony.SNS_RECEIVE");
                myBroadcastReceiver = new MyBroadcastReceiver();
                registerReceiver(myBroadcastReceiver, filter);
                break;
            case R.id.unRegister:
                unregisterReceiver(myBroadcastReceiver);
                break;

            default:
                break;
        }
    }
}  
```

对应的布局文件也是很简单，就两个button，就不写了，对应的MyBroadcastReceive也很简单也不多说了，主要注意的是，onReceive()方法有一个参数intent，这里面封装了广播传递给我们的数据，至于这些数据是什么，怎么处理，大家可以自己研究一下，另外Android本身给我们提供了哪些广播事件，大家可以自己看看，老夫在这里就不多说了

PS：BroadcastReceiver作为Android的四大组件之一，用法肯定不会是如此之简单，老夫是第一次写Android的东西，既是初学还是自学，也没做过项目，所以不免有错误的地方和有点抓不住重点，还请大家能留言交流，谢谢