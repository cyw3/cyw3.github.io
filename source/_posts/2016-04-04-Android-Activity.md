---
title: Activity on Android
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---
## 前言

Activity是Android组件中最基本也是最为常见用的[四大组件](http://www.cnblogs.com/pepcod/archive/2013/02/11/2937403.html)（Activity，Service服务,Content Provider内容提供者，BroadcastReceiver广播接收器）之一。Activity是一个应用程序组件，提供一个屏幕，用户可以用来交互为了完成某项任务。在一个Android应用中，一个Activity通常就是一个单独的屏幕，它上面可以显示一些控件也可以监听并处理用户的事件做出响应。

## 目录

1. Activity的状态
2. Activity的生命周期
3. Activity栈
4. Activity的加载模式以及区别
5. Activity 之间通信
6. Activity 的 Intent Filter

## 一、Activity的状态

在 Android 中，Activity 拥有四种基本状态：

活动态Active/Runing：当一个Activity在Activity栈顶，它处于可视的、有焦点、可接受用户输入的激活状态。Android试图尽最大可能保持它活动状态，杀死其它Activity来确保当前活动Activity有足够的资源可使用。当另外一个Activity被激活，这个将会被暂停。

暂停态Paused：当 Activity 被另一个透明或者 Dialog 样式的 Activity 覆盖时的状态。此时它依然与窗口管理器保持连接，系统继续维护其内部状态，所以它仍然可见，但它已经失去了焦点故不可与用户交互。当被暂停，一个Activity仍会被当成活动状态，只不过是不可以接受用户输入。在极特殊的情况下，Android将会杀死一个暂停的Activity来为活动的Activity提供充足的资源。当一个Activity变为完全隐藏，它将会变成停止。

停止态Stoped：当 Activity 被另外一个 Activity 覆盖、失去焦点并不可见时处于 Stoped状态。当一个Activity不是可视的，它“停止”了。这个Activity将仍然在内存中保存它所有的状态和会员信息。尽管如此，当其它地方需要内存时，它将是最有可能被释放资源的。当一个Activity停止后，一个很重要的步骤是要保存数据和当前UI状态。一旦一个Activity退出或关闭了，它将变为待用状态。

待用态Killed：在一个Activity被杀死后和被装在前，它是待用状态的。待用Acitivity被移除Activity栈，并且需要在显示和可用之前重新启动它。

当一个 Activity 实例被创建、销毁或者启动另外一个 Activity时，它在这四种状态之间进行转换，这种转换的发生依赖于用户程序的动作。下图说明了Activity在不同状态间转换的时机和条件：

![ActivityStates.jpg](/img/ActivityStates.jpg)

如上所示，Android 程序员可以决定一个 Activity 的“生”，但不能决定它的“死”，也就时说程序员可以启动一个 Activity，但是却不能手动的“结束”一个 Activity。当你调用 Activity.finish()方法时，结果和用户按下 BACK 键一样：告诉 Activity Manager 该 Activity 实例完成了相应的工作，可以被“回收”。随后 Activity Manager 激活处于栈第二层的 Activity 并重新入栈，同时原 Activity 被压入到栈的第二层，从 Active 状态转到 Paused 状态。例如：从 Activity1 中启动了 Activity2，则当前处于栈顶端的是 Activity2，第二层是 Activity1，当我们调用 Activity2.finish()方法时，Activity Manager 重新激活 Activity1 并入栈，Activity2 从 Active 状态转换 Stoped 状态，Activity1. onActivityResult(int requestCode, int resultCode, Intent data)方法被执行，Activity2 返回的数据通过 data参数返回给 Activity1。

## 二、 Activity的生命周期

Activty的生命周期的也就是它所在进程的生命周期。

![ActivityLoop.jpg](/img/ActivityLoop.jpg)

一个Activity的启动顺序：
onCreate()——>onStart()——>onResume()

当另一个Activity启动时:
第一个Activity onPause()——>第二个Activity onCreate()——>onStart()——> onResume() ——>第一个Activity onStop()

当返回到第一个Activity时：
第二个Activity onPause() ——> 第一个Activity onRestart()——>onStart()——>onResume() ——>第二个Activity onStop()——>onDestroy()

一个Activity的销毁顺序:
（情况一）onPause()——><Process Killed> 
（情况二）onPause()——>onStop()——><Process Killed> 
（情况三）onPause()——>onStop()——>onDestroy()

每一个活动（ Activity ）都处于某一个状态，对于开发者来说，是无法控制其应用程序处于某一个状态的，这些均由系统来完成。但是当一个活动的状态发生改变的时候，开发者可以通过调用 onXX()的方法获取到相关的通知信息。

在实现 Activity 类的时候，通过覆盖（ override ）这些方法即可在你需要处理的时候来调用。

1、onCreate ：当活动第一次启动的时候，触发该方法，可以在此时完成活动的初始化工作。 onCreate 方法有一个参数，该参数可以为空( null )，也可以是之前调用 onSaveInstanceState()方法保存到存储设备中的数据。

2、onStart ：该方法在 onCreate() 方法之后被调用，或者在 Activity 从 Stop 状态转换为 Active 状态时被调用。该方法的触发表示所属活动将被展现给用户。

3、onResume ：当一个活动和用户发生交互的时候，触发该方法。在 Activity 从 Pause 状态转换到 Active 状态时被调用。

4、onPause ：当一个正在前台运行的活动因为其他的活动需要前台运行而转入后台运行的时候，触发该方法。这时候需要将活动的状态持久化，比如正在编辑的数据库记录等。

5、onStop：当一个活动不再需要展示给用户的时候，触发该方法。如果内存紧张，系统会直接结束这个活动，而不会触发 onStop 方法。 所以保存状态信息是应该在onPause时做，而不是onStop时做。活动如果没有在前台运行，都将被停止或者Linux管理进程为了给新的活动预留足够的存储空间而随时结束这些活动。因此对于开发者来说，在设计应用程序的时候，必须时刻牢记这一原则。在一些情况下，onPause方法或许是活动触发的最后的方法，因此开发者需要在这个时候保存需要保存的信息。

6、onRestart ：当处于停止状态的活动需要再次展现给用户的时候，触发该方法。

7、onDestroy ：当活动销毁的时候，触发该方法。和onStop方法一样，如果内存紧张，系统会直接结束这个活动而不会触发该方法。

8、onSaveInstanceState ：系统调用该方法，允许活动保存之前的状态，比如说在一串字符串中的光标所处的位置等。 通常情况下，开发者不需要重写覆盖该方法，在默认的实现中，已经提供了自动保存活动所涉及到的用户界面组件的所有状态信息。

## 三、Activity栈

Android 是通过一种 Activity 栈的方式来管理 Activity 的，一个 Activity 的实例的状态决定它在栈中的位置。处于前台的 Activity 总是在栈的顶端，当前台的 Activity 因为异常或其它原因被销毁时，处于栈第二层的 Activity 将被激活，上浮到栈顶。当新的 Activity 启动入栈时，原 Activity 会被压入到栈的第二层。一个 Activity 在栈中的位置变化反映了它在不同状态间的转换。Activity 的状态与它在栈中的位置关系如下图所示：

![ActivityStack.jpg](/img/ActivityStack.jpg)

每个Activity的状态是由它在Activity栈（是一个后进先出LIFO，包含所有正在运行Activity的队列）中的位置决定的。

一个应用程序的优先级是受最高优先级的Activity影响的。当决定某个应用程序是否要终结去释放资源，Android内存管理使用栈来决定基于Activity的应用程序的优先级。

![ActivityStack2.jpg](/img/ActivityStack2.jpg)

## 四、Activity的加载模式以及区别

在Android的多Activity开发中，Activity之间的跳转可能需要有多种方式，有时是普通的生成一个新实例，有时希望跳转到原来某个Activity实例，而不是生成大量的重复的Activity。加载模式便是决定以哪种方式启动一个跳转到原来某个Activity实例。

在Android里，有4种Activity的启动模式，分别为：

1、标准模式standard：一调用startActivity()方法就会产生一个新的实例。

2、singleTop:如果已经有一个实例位于Activity栈的顶部时，就不产生新的实例，而只是调用Activity中的newInstance()方法。如果不位于栈顶，会产生一个新的实例。

3、singleTask: 会在一个新的task中产生这个实例，以后每次调用都会使用这个，不会去产生新的实例了。

4、singleInstance: 这个跟singleTask基本上是一样，只有一个区别：在这个模式下的Activity实例所处的task中，只能有这个activity实例，不能有其他的实例。

这些启动模式可以在功能清单文件AndroidManifest.xml中的launchMode属性进行设置。

相关的代码中也有一些标志可以使用,比如我们想只启用一个实例,则可以使用 Intent.FLAG_ACTIVITY_REORDER_TO_FRONT 标志，这个标志表示：如果这个activity已经启动了，就不产生新的activity，而只是把这个activity实例加到栈顶来就可以了。

```ruby
 Intent intent =new Intent(ReorderFour.this,ReorderTwo.class);  
 intent.addFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT);
 startActivity(intent);
```

四种加载模式的区别:

1、所属task的区别

一般情况下，standard和singleTop的Activity的目标task，和收到的Intent的发送者在同一个task内，就相当于谁调用它，它就跟谁在同一个Task中。

除非Intent包括参数FLAG_ACTIVITY_NEW_TASK。如果提供了FLAG_ACTIVITY_NEW_TASK参数，会启动到别的task里。
singleTask和singleInstance总是把要启动的Activity作为一个task的根元素，他们不会被启动到一个其他task里。

2、是否允许多个实例

standard和singleTop可以被实例化多次，并且是可以存在于不同的task中。这种实例化时一个task可以包括一个activity的多个实例。

singleTask和singleInstance则限制只生成一个实例，并且是task的根元素。

singleTop 要求在创建intent的时候，如果栈顶已经有要创建的Activity的实例，则将intent发送给该实例，而不创建新的实例。

3、是否允许其它Activity存在于本task内

singleInstance独占一个task，其它Activity不能存在那个task里。

如果它启动了一个新的Activity，不管新的Activity的launch mode如何，新的Activity都将会到别的task里运行（如同加了FLAG_ACTIVITY_NEW_TASK参数）。

而另外三种模式，则可以和其它activity共存。

4、是否每次都生成新实例

standard对于每一个启动Intent都会生成一个activity的新实例。

singleTop的Activity如果在task的栈顶的话，则不生成新的该activity的实例，直接使用栈顶的实例，否则，生成该activity的实例。

比如：现在task栈元素为A-B-C-D（D在栈顶），这时候给D发一个启动intent，如果D是standard的，则生成D的一个新实例，栈变为A－B－C－D－D。如果D是singleTop的话，则不会生产D的新实例，栈状态仍为A-B-C-D。

如果这时候给B发Intent的话，不管B的launchmode是standard还是 singleTop，都会生成B的新实例，栈状态变为A-B-C-D-B。

singleInstance是其所在栈的唯一Activity，它会每次都被重用。

singleTask如果在栈顶，则接受intent，否则，该intent会被丢弃，但是该task仍会回到前台。当已经存在的Activity实例处理新的intent时候，会调用onNewIntent()方法，如果收到intent生成一个Activity实例，那么用户可以通过back键回到上一个状态；如果是已经存在的一个Activity来处理这个intent的话，用户不能通过按back键返回到这之前的状态。

## 五、Activity 之间通信

在 Android 中，不同的Activity实例可能运行在一个进程中，也可能运行在不同的进程中。因此我们需要一种特别的机制帮助我们在 Activity 之间传递消息。

1、使用 Intent 通信

Android 中通过 Intent 对象来表示一条消息，一个 Intent 对象不仅包含有这个消息的目的地，还可以包含消息的内容，这好比一封 Email，其中不仅应该包含收件地址，还可以包含具体的内容。对于一个 Intent 对象，消息“目的地”是必须的，而内容则是可选项。

在通过 Activity. startActivity(intent)启动另外一个 Activity 的时候，我们在 Intent 类的构造器中指定了“收件人地址”。

如果我们想要给“收件人”Activity 说点什么的话，那么可以通过下面这封“e-mail”来将我们消息传递出去：

```ruby
 Intent intent =new Intent(CurrentActivity.this,OtherActivity.class);
  // 创建一个带“收件人地址”的 email 
 Bundle bundle =new Bundle();// 创建 email 内容
 bundle.putBoolean("boolean_key", true);// 编写内容
 bundle.putString("string_key", "string_value"); 
 intent.putExtra("key", bundle);// 封装 email 
 startActivity(intent);// 启动新的 Activity
```

那么“收件人”该如何收信呢？在 OtherActivity类的 onCreate()或者其它任何地方使用下面的代码就可以打开这封“e-mail”阅读其中的信息：

```ruby
 Intent intent =getIntent();// 收取 email 
 Bundle bundle =intent.getBundleExtra("key");// 打开 email 
 bundle.getBoolean("boolean_key");// 读取内容
 bundle.getString("string_key");
```

上面我们通过 bundle对象来传递信息，bundle维护了个 HashMap<String, Object>对象，将我们的数据存贮在这个 HashMap 中来进行传递。但是像上面这样的代码稍显复杂，因为 Intent 内部为我们准备好了一个 bundle，所以我们也可以使用这种更为简便的方法：

```ruby
 Intent intent =new Intent(EX06.this,OtherActivity.class); 
 intent.putExtra("boolean_key", true); 
 intent.putExtra("string_key", "string_value"); 
 startActivity(intent);
```

```ruby
 Intent intent=getIntent(); 
 intent.getBooleanExtra("boolean_key",false); 
 intent.getStringExtra("string_key");
```

2、使用 SharedPreferences

SharedPreferences 使用 xml 格式为 Android 应用提供一种永久的数据存贮方式。对于一个 Android 应用，它存贮在文件系统的 /data/ data/your_app_package_name/shared_prefs/目录下，可以被处在同一个应用中的所有 Activity 访问。Android 提供了相关的 API 来处理这些数据而不需要程序员直接操作这些文件或者考虑数据同步问题。

```ruby
 // 写入 SharedPreferences 
 SharedPreferences preferences = getSharedPreferences("name", MODE_PRIVATE); 
 Editor editor = preferences.edit(); 
 editor.putBoolean("boolean_key", true); 
 editor.putString("string_key", "string_value"); 
 editor.commit(); 
        
 // 读取 SharedPreferences 
 SharedPreferences preferences = getSharedPreferences("name", MODE_PRIVATE); 
 preferences.getBoolean("boolean_key", false); 
 preferences.getString("string_key", "default_value");
```

3、其它方式

Android 提供了包括 SharedPreferences 在内的很多种数据存贮方式，比如 SQLite，文件等，程序员可以通过这些 API 实现 Activity 之间的数据交换。如果必要，我们还可以使用 IPC 方式。

## 六、Activity 的 Intent Filter

Intent Filter 描述了一个组件愿意接收什么样的 Intent 对象，Android 将其抽象为 android.content.IntentFilter 类。在 Android 的 AndroidManifest.xml 配置文件中可以通过 <intent-filter >节点为一个 Activity 指定其 Intent Filter，以便告诉系统该 Activity 可以响应什么类型的 Intent。

当程序员使用 startActivity(intent) 来启动另外一个 Activity 时，如果直接指定 intent 对象的 Component 属性，那么 Activity Manager 将试图启动其 Component 属性指定的 Activity。否则 Android 将通过 Intent 的其它属性从安装在系统中的所有 Activity 中查找与之最匹配的一个启动，如果没有找到合适的 Activity，应用程序会得到一个系统抛出的异常。Activity中Intent Filter 的匹配过程如下：

![ActivityIntentFilter.jpg](/img/ActivityIntentFilter.jpg)

1、Action 匹配
Action 是一个用户定义的字符串，用于描述一个 Android 应用程序组件，一个 Intent Filter 可以包含多个 Action。在 AndroidManifest.xml 的 Activity 定义时可以在其 <intent-filter >节点指定一个 Action 列表用于标示 Activity 所能接受的“动作”，例如：

```ruby
 <intent-filter > 
 <action android:name="android.intent.action.MAIN" /> 
 <action android:name="com.cyw.myaction" /> 
……
 </intent-filter>
```

如果我们在启动一个 Activity 时使用这样的 Intent 对象：

```ruby
 Intent intent =new Intent(); 
 intent.setAction("com.cyw.myaction");
```

那么所有的 Action 列表中包含了“com.cyw.myaction”的 Activity 都将会匹配成功。
Android 预定义了一系列的 Action 分别表示特定的系统动作。这些 Action 通过常量的方式定义在 android.content. Intent中，以“ACTION_”开头。我们可以在 Android 提供的文档中找到它们的详细说明。

2、URI 数据匹配

一个 Intent 可以通过 URI 携带外部数据给目标组件。在 <intent-filter >节点中，通过 <data/>节点匹配外部数据。
mimeType 属性指定携带外部数据的数据类型，scheme 指定协议，host、port、path 指定数据的位置、端口、和路径。如下：

```ruby
 <data android:mimeType="mimeType" android:scheme="scheme" 
 android:host="host" android:port="port" android:path="path"/>
```

如果在 Intent Filter 中指定了这些属性，那么只有所有的属性都匹配成功时 URI 数据匹配才会成功。

3、Category 类别匹配
<intent-filter >节点中可以为组件定义一个 Category 类别列表，当 Intent 中包含这个列表的所有项目时 Category 类别匹配才会成功。
