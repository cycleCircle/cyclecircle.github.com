---
layout: post
title: "Android Screen Lock"
date: 2015-07-29 10:20:12 +0800
comments: true
categories: 
---


在Android上面做锁屏是一件挺麻烦的事情，要解决的事情还是比较多的

### 重启

首先就是要重启后能够自动启动，这个还是比较容易的，只要设置一个开机启动就好了

``` java 
<receiver android:name=".BootReceiver">  
    <intent-filter >  
        <action android:name=  
             "android.intent.action.BOOT_COMPLETED"/>  
   </intent-filter>  
</receiver>  

<uses-permission  
 android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
```

在收到广播就启动自己

``` java 
public class BootReceiver extends BroadcastReceiver {  

  @Override  
  public void onReceive(Context context, Intent intent) {  
    Intent myIntent = new Intent(context, MainActivity.class);  
    myIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);  
    context.startActivity(myIntent);  
  }  
}  
```


### 返回键和HOME键

处理返回键和HOME键，这个也是比较简单的，重写一下onKeyDown方法就好了

``` java 
	@Override
	public boolean onKeyDown(int keyCode, KeyEvent event)
	{
		if(keyCode == KeyEvent.KEYCODE_BACK && event.getAction() == KeyEvent.ACTION_DOWN)
		{
			return true;
		}
		if(keyCode == KeyEvent.KEYCODE_HOME)
		{
			return true;
		}
		return super.onKeyDown(keyCode, event);
	}
```

Home键的处理，也可以通过声明AndroidManifest文件来进行处理，但这个有一个问题，就是要选择我们的应用
才行

``` java 
<intent-filter>  
    <action android:name="android.intent.action.MAIN" />  
 
    <category android:name="android.intent.category.HOME" />  
    <category android:name="android.intent.category.LAUNCHER" />  
    <category android:name="android.intent.category.DEFAULT" />  
</intent-filter>  
```


### 电源键

处理电源键是很麻烦的一件事，在4.0以下的版本里面有一个非常简单的解决方式

``` java 

@Override  
public void onAttachedToWindow() {  
    getWindow().addFlags(  
        WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD);  
    getWindow().addFlags(  
        WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED);  
}     
```


### 系统对话框

有些时候，系统可能会弹出一些系统对话框来，比如升级，或低电量这些，这些有可能会把锁屏给绕过，进入
系统，

所以为了摆脱这些系统弹框的困扰，可用的解决方法为：当Activity失去焦点时，发送一个关闭所有系统
对话框的广播

``` java 
@Override  
public void onWindowFocusChanged(boolean hasFocus) {  
  super.onWindowFocusChanged(hasFocus);  
  if(!hasFocus) {  
    Intent closeDialog =   
          new Intent(Intent.ACTION_CLOSE_SYSTEM_DIALOGS);  
    sendBroadcast(closeDialog);  
  }  
}  
```


### 虚拟键盘

如果键盘是需要的，那就最好用自己的实现


### 状态栏

这个也是一个比较难处理的

在Android4.0以下的系统，可以指定窗口的类型为`TYPE_SYSTEM_ALERT`，这样的窗口一般会显示在最上面

``` java 
	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		
		final WindowManager.LayoutParams params = new WindowManager.LayoutParams(
				WindowManager.LayoutParams.MATCH_PARENT,
				WindowManager.LayoutParams.MATCH_PARENT,
				WindowManager.LayoutParams.TYPE_SYSTEM_ALERT,
				WindowManager.LayoutParams.FLAG_LAYOUT_IN_SCREEN,
				PixelFormat.TRANSLUCENT);

		wm = (WindowManager) getApplicationContext()
		    .getSystemService(Context.WINDOW_SERVICE);

		mTopView = (ViewGroup) View.inflate(this, R.layout.activity_main, null);
		getWindow().setAttributes(params);
		wm.addView(mTopView, params);
		
	}
```


还可以当状态栏一出现，就把它给隐藏掉，但这个需要权限

``` java 
<uses-permission  
        android:name="android.permission.EXPAND_STATUS_BAR"/>  

@Override  
public void onWindowFocusChanged(boolean hasFocus)<br>  
{  
   if(!hasFocus)  
   {  
           Object service  = getSystemService("statusbar");  
           Class<?> statusbarManager =   
              Class.forName("android.app.StatusBarManager");  
           Method collapse = <br>  
              statusbarManager.getMethod("collapse");  
           collapse .setAccessible(true);  
           collapse .invoke(service);  
   }  
}  
```

而且在Android4.1后，还可以使用SDK来隐藏状态栏（这个可能会有问题，慎用）

``` java 
View decorView = getWindow().getDecorView();  
int uiOptions = View.SYSTEM_UI_FLAG_FULLSCREEN;  
decorView.setSystemUiVisibility(uiOptions);  
ActionBar actionBar = getActionBar();  
actionBar.hide(); 
```

另外还有一个方法就是，创建一个透明的视图对象，拦截掉状态栏上的点击事件，这个视图要求需要
`SYSTEM_ALERT_WINDOW`标识
