---
title: Android之Activity之间的参数传递
author: Bridge Li
type: post
date: 2015-02-01T15:04:36+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - Android
tags:
  - activity
  - parameter

---
前面写了很多activity的东西，什么生命周期之类、常用控件以及这些控件怎么布局之类的，今天从另外一个角度在看activity：一个应用其实就是不同activity的切换，然后展示一些数据给用户，但是不同的activity之前肯定有着联系，那么他们之间怎么联系呢？说白了就是他们之间的参数是如何传递的呢？在讲不同的activity之间相互传参之前，我们先看这样两个问题：①：我们有时候需要从一个activity跳转到另一个activity，并且同时把参数带过去，然后一般就没有第一个activity啥事了，这个很常见所以就不举例子了，我们暂且称这一种为：普通的参数传递；②：有些时候呢，却不是这样，我们需要在第一个activity中打开第二个activity，在第二个activity中选取一个内容后，把我们选取的参数带回第一个activity，然后在第一个activity中处理带回来的参数，例如：我们在上传头像时，我们在第一个activity中，打开第二个activity，然后在第二个activity中，选取图片、裁剪图片，然后把处理好之后的图片带回到第一个activity，我们暂且称这种叫：关心结果的参数传递。  
在明白有这两种需求之后，我们发现不同的activity之间参数传递其实很简单，不同的activity之间传递参数是依靠一个对象：android.content.Intent，只要有这个对象就可以满足我们在不同的activity之间传递参数的需求了。下面就让我们结合这两种情况，看看不同activity之间参数如何传递。

一、普通的参数传递  
废话不多说，先看例子：

主activity的代码如下：  
```  
package cn.bridgeli.demo;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

public class MainActivity extends Activity {

    private EditText factorOne = null;
    private TextView symbol = null;
    private EditText factorTwo = null;
    private Button calc = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        factorOne = (EditText) findViewById(R.id.factorOne);
        symbol = (TextView) findViewById(R.id.symbol);
        symbol.setText("乘以");
        factorTwo = (EditText) findViewById(R.id.factorTwo);

        calc = (Button) findViewById(R.id.calc);
        calc.setText("计算");
        calc.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View arg0) {
                String firstNum = factorOne.getText().toString();
                String secondNum = factorTwo.getText().toString();
                Intent intent = new Intent();
                intent.setClass(MainActivity.this, ResultActivity.class);
                intent.putExtra("firstNum", firstNum);
                intent.putExtra("secondNum", secondNum);
                startActivity(intent);
            }
        });
    }
}
```

其对应的布局文件：activity_main.xml

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

<EditText  
android:id="@+id/factorOne"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"/>

<TextView  
android:id="@+id/symbol"  
android:layout_width="wrap_content"  
android:layout_height="wrap_content"/>

<EditText  
android:id="@+id/factorTwo"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"/>

<Button  
android:id="@+id/calc"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"/>  
</LinearLayout>  
```

在MianActivity中第38行，就是说明从一个activity跳转到另一个activity，41行就是启动另一个activity，参数intent中封装了要传递过去的所有参数，下面让我们看看ResultActivity的代码：

```  
package cn.bridgeli.demo;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.widget.TextView;

public class ResultActivity extends Activity {

    private TextView result = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result);

        Intent intent = getIntent();
        int firstNum = Integer.parseInt(intent.getStringExtra("firstNum"));
        int secondNum = Integer.parseInt(intent.getStringExtra("secondNum"));
        int temp = firstNum * secondNum;

        result = (TextView) findViewById(R.id.result);
        result.setText(temp + "");

    }
}  
```

对应的布局文件activity_result.xml就比较简单了，代码如下：

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

<TextView  
android:id="@+id/result"  
android:layout_width="wrap_content"  
android:layout_height="wrap_content"/>  
</LinearLayout>  
```

因为代码比较简单，老夫就不多啰嗦了，相信读者一看就明白了。需要说明白的是，每一个activity都需要在mainifest.xml中进行配置，配置文件如下：  
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

<activity  
android:name=".ResultActivity"  
android:label="@string/app_name" >  
</activity>  
</application>

</manifest>  
```

在这个配置文件里面有很多东西，目前我们只需要关注activity的配置就行了，这样的话相信读者一看就明白了，需要多说一句的是：一个app既然有很多个不同的activity，那么这些activity哪一个是启动的主activity呢？就是哪个是app启动起来我们看到的第一个activity呢？很简单，就是有配置：

```  
<intent-filter>  
<action android:name="android.intent.action.MAIN" />

<category android:name="android.intent.category.LAUNCHER" />  
</intent-filter>  
```

的那一个。  
如果有看过我之前的文章的同学，相信已经发现我在前面写常见控件的那篇文章时，已经写过，当时只是没说那么明白而已，下面我们看第二种

二、关心结果的参数传递

同样废话不多说，直接看例子，主activity的代码如下：

```  
package cn.bridgeli.demo;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends Activity {

    private Button start = null;
    private TextView result = null;
    public static final int REQUSET = 1;

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == MainActivity.REQUSET && resultCode == RESULT_OK) {
            String str = "账号：" + data.getStringExtra(RequestActivity.USERNAME) + "n" + "密码：" + data.getStringExtra(RequestActivity.PASSWORD);
            result.setText(str);
        }
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        result = (TextView) findViewById(R.id.result);
        start = (Button) findViewById(R.id.start);
        start.setText("启动");
        start.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.setClass(MainActivity.this, RequestActivity.class);
                startActivityForResult(intent, REQUSET);
            }
        });

    }
}
```

其实经过这么多相信大家也应该猜到了，布局文件activity_main也会很简单，代码如下：

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

<TextView  
android:id="@+id/result"  
android:layout_width="wrap_content"  
android:layout_height="wrap_content"/>  
</LinearLayout>  
```

另一个activity的代码如下：

```  
package cn.bridgeli.demo;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;

public class RequestActivity extends Activity {

    private Button start = null;
    private EditText username = null;
    private EditText password = null;
    public static final String USERNAME = "username";
    public static final String PASSWORD = "password";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_result);

        username = (EditText) findViewById(R.id.username);
        password = (EditText) findViewById(R.id.password);
        start = (Button) findViewById(R.id.start);
        start.setText("计算");
        start.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.putExtra(USERNAME, username.getText().toString());
                intent.putExtra(PASSWORD, password.getText().toString());
                setResult(RESULT_OK, intent);
                finish();
            }
        });
    }
}
```

对应的布局文件activity_result代码如下：

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

<EditText  
android:id="@+id/username"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"  
android:inputType="text"  
android:hint="username"/>

<EditText  
android:id="@+id/password"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"  
android:inputType="textPassword"  
android:hint="password"/>

<Button  
android:id="@+id/start"  
android:layout_width="fill_parent"  
android:layout_height="wrap_content"/>

</LinearLayout>  
```

代码比较简单就不多啰嗦了，读者自己看吧，相信以读者的聪明一看就会很明白，另外这个例子对应的mainifest文件和上一个例子几乎一样，就省去不写了

PS：第一次写Android的东西，既是初学还是自学，也没做过项目，所以不免有错误的地方和有点抓不住重点，还请大家能留言交流，谢谢