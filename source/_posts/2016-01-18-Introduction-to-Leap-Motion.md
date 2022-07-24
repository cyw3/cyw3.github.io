---
title: Introduction to Leap Motion
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---

## 前言

最近是要完成一个关于Leap Motion的项目，所以就顺便到[官网](https://developer.leapmotion.com/documentation/)翻译了下文章。一下首先是翻译的可以直接上手的[Setting up a project](https://developer.leapmotion.com/documentation/java/devguide/Project_Setup.html)，以及[Hello World](https://developer.leapmotion.com/documentation/java/devguide/Sample_Tutorial.html#id24)。

## 目录

1.Setting up a project

2.Hello World

## 1.Setting up a project

Leap Motion java SDK 使用的封装了Leap Motion API 类定义的标准jar包，以及一系列的原生库(native libraries)，让你可以开发Leap的java程序，与Leap Motion进行数据交换。通常，创建一个java项目包含以下步骤：添加LeapJava.jar文件到你的项目的classpath文件中，设置JVM库路径参数以至于你的JVM能够找得到Leap Motion的原生库。

### Leap Motion Java libraries

Leap Motion的jar包是跨平台的，但是他的原生库必须要与系统平台以及用于执行程序的JVM的体系结构进行匹配。

为了能偶在Java程序中使用到Leap Motion的SDk，你需要添加LeapJava.jar文件到classpath文件中，然后设置java库的路径到Leap Motion原生库的位置。

需要使用到Leap Motion Java SDK的以下Java和原生库。

- 32-bit Windows:
    - LeapSDK/lib/LeapJava.jar--Leap Motion Java API的类定义。
    - LeapSDk/lib/x86/LeapJava.dll--在Windows系统下的32-bit Leap Motion Java库。
    - LeapSDK/lib/x86/Leap.dll--在Windows系统下的32-bit Leap Motion库。
- 64-bit Windows:
    - LeapSDK/lib/LeapJava.jar--Leap Motion Java API的类定义。
    - LeapSDK/lib/x64/LeapJava.dll--在Windows系统下的64-bit Leap Motion Java库。
    - LeapSDK/lib/x64/Leap.dll--在Windows系统下的64-bit Leap Motion库。
- 32- or 64-bit Mac OS:
    - LeapSDK/lib/LeapJava.jar--Leap Motion Java API的类定义。
    - LeapSDK/lib/libLeapJava.sylib--Mac系统下的Leap Motion Java库。
    - LeapSDK/lib/libLeap.sylib--Mac系统下的Leap Motion库。

### Compile from the command line

使用java编译器，javac编译，设置classpath选项以指定LeapJar文件。举个例子，要编译包含在Leap Motion SDk中的Sample.java，你可以使用一下命令行：

```ruby

javac -classpath <LeapMotion>/lib/LeapJava.jar Sample.java


```

(<LeapSDK>是指你安装的Leap Motion SDK所在的文件夹)

### Run from the command line

为了运行Leap驱动的程序，java在运行时需要找到Leap Motion原生本地库。LeapJava.jar必须已经设置在classpath文件里面了。你可以设置java的'java.library.path'参数以识别原生库。更关键的是，在Windows系统下，你必须制定32-bit 或者64-bit库以匹配你使用的JVM的体系架构。

在MAC上，你可以使用一下命令行运行'Sample'程序：

```ruby

java -classpath ".:<LeapSDK>/lib/LeapJava.jar" -Sjava.library.path=<LeapSDK>/lib Sample

```


在Windows下，你可以使用以下命令行在64-bit JVm下运行Sample程序：

```ruby

java -classpath ".;<LeapSDK>/lib/LeapJava.jar" -Djava.library.path=<LeapSDK>/lib/x64 Sample


```


### Eclipse

在eclipse IDE里，你要添加LeapJava.jar文件到项目工程中，作为一个外部jar包，然后设置正确的原生Leap Motion库的路径为Jar包的一个属性。

1. 在Eclipse的文件菜单上选择*New > Java Project*.
2. 在*Create Java Project*页面上给项目工程指定一个名称，然后如必要的话设置其他属性(Leap Motion SDK支持Java 6和7)
3. 点击*Next*键，进入Java Setting页面
4. 选择*Livbrary*选项
5. 点击*Add External Jars...*按钮
6. 导航到LeapJava.jar包
7. 点击*Open*键，添加LeapJava.jar到项目工程中
8. 接着，在Library列表里面点击LeapJava.jar条目前面的小三角形，出现Library的属性
![java setting page](https://developer-china-cdn.leapmotion.com/documentation/images/Setup_Eclipse_Jar_Properties.png)

9. 选择*Native Library location*条目
10. 点击*Edit*按钮
11. 导航到半酣Leap Motion原生库的文件夹

在Windows下，需要确保你要选择的文件夹下包含适合你的目标体系架构的正确的库。如果你的是32-bit JVM，要用SDk里面x86文件夹下的Leap Motion库。如果你的是64-bit JVM，要用SDk里面x64文件夹下的Leap Motion库。在Mac上，Leap Motion库支持任意体系架构。

12. 点击*OK*设置路径

*注意：或者你也可以通过项目属性对话框添加Leap Motion库到一个已经存在的项目工程之中。*

### IntelliJ

在IntelliJ IDE上，你要添加LeapJava.jar文件到项目工程中作为一个库，需要分别使用JVM参数设置Leap Motion原生库的路径，'java.library.path'.JVM 参数可以通过使用IntelliJ Run/Debug组态来设置。

以下步骤添加LeapJava.jar到项目工程中：

1. 按通常的方法创建了一个工程之后，选择*File > Project Structure*菜单打开设置对话框
2. 在project Setting下点击*Library*
3. 点击在库列表顶部的小'+'号按钮，打开*Select Library Files*对话框
4. 从你的Leap Motion SDK添加'LeapJava.jar'

通过创建一个*Run/Debug configuration*来设置到原生Leap Motion库的路径：

1. 选择*Run > Edit Configurations...* 菜单
2. 点击在配置离别上面的小'+'号按钮
3. 选择应用，创建一个新的应用配置
4. 指定一个名称
5. 设置VM选项域来设置'-Djava.library.path'参数指向Leap Motion原生库所在的文件夹
6. 点击*OK*

![IntelliJ](https://developer-china-cdn.leapmotion.com/documentation/images/Setup_IntelliJ_Properties.png)

*在Windows下，确保选择跟你的JVM体系匹配的文件夹。如果你的是32-bit JVM，要用SDk里面x86文件夹下的Leap Motion库。如果你的是64-bit JVM，要用SDk里面x64文件夹下的Leap Motion库。在Mac上，Leap Motion库支持任意体系架构。*

### NetBeans

在NetBeans IDE下，你要添加LeapJava.jar文件到项目工程中作为一个库，需要分别使用JVM参数设置Leap Motion原生库的路径，'java.library.path'.JVM 参数可以通过使用IntelliJ Run/Debug组态来设置。

以下步骤添加LeapJava.jar到项目工程中：

1. 按通常的方法创建了一个工程之后，选择*File > Project Properties*菜单打开设置对话框
2. 点击*Library*
3. 点击*Add Jar/Folder* 按钮
4. 在你的Leap Motion SDk中找到'LeapJava.jar'
5. 点击*OK*添加jar包到你的project中
6. 接着，在你的项目属性列表中点击*Run*
7. 设置VM选项域来设置'-Djava.library.path'参数指向Leap Motion原生库所在的文件夹

![NetBeans](https://developer-china-cdn.leapmotion.com/documentation/images/Setup_Netbeans_Properties.png)

*在Windows下，确保选择跟你的JVM体系匹配的文件夹。如果你的是32-bit JVM，要用SDk里面x86文件夹下的Leap Motion库。如果你的是64-bit JVM，要用SDk里面x64文件夹下的Leap Motion库。在Mac上，Leap Motion库支持任意体系架构。*

## 2. Hello World

这篇文章演示了如何连接Leap Motion控制器，还有基本的访问跟踪数据操作。在阅读完这篇文章之后以及紧随文章编写属于你自己的基本程序，你应该拥有了扎实的基础，可以开始你的应用开发之旅了。

首先，是一小段的背景知识...

### How the Leap Motion Controller Works

Leap Motion控制器着重的包括硬件和软件组件。

Leap Motion硬件主要包括一对立体声红外摄像机和照明LED的。摄像传感器朝上(当该装置处于其标准定向的时候)。下面的这个插图从Leap Motion传感器的视角来展示了一名用户的手。

![hands](https://developer-china-cdn.leapmotion.com/documentation/images/Leap_View.jpg)

Leap Motion软件接收来自传感器的数据，然后分析这个数据，尤其是手掌、手指、手臂，或者工具的数据(追踪其他类型的数据，可能会在将来添加对应的对象的，而这是当前能跟踪的实体)。软件保持着人手的内部模型，然后与传感器发送来的数据组成的模型进行比较，以决定最好的匹配样式。传感器数据将会一帧一帧的被分析，设备会传送每一帧的数据到你的Leap Motion驱动的应用里面。被你的应用接收到的'Frame'对象包含所有已知的位置，速度和跟踪实体的辨识。比如手掌，手指，还有工具。构造，比如说手势，和跨越多个帧的动作，都需要不断刷新每一帧。如果需要控制器提供的跟踪数据的概述的话，可以阅读[API Overview](https://developer.leapmotion.com/documentation/java/devguide/Leap_Overview.html)。

Leap Motion软件可以在客户端计算机上作为一种服务(Windows)或者是一个守护线程(Mac和Linux)。原生的Leap Motion驱动的应用可以使用Leap Motion提供的动态库(作为Leap Motion SDk的一部分提供)连接到这个服务。web应用可以连接到有服务托管的WebSocket服务器上面。WebSocket提供以JSON格式信息的跟踪数据--每一帧数据每一条信息。JavaScript库，LeapJS，提供了包含这个数据的API。想要更多信息的话，可以阅读[System Architecture](https://developer.leapmotion.com/documentation/java/devguide/Leap_Architecture.html)。

### Set Up the Files

这个教程也使用命令行编译器和连接器(如有必要)，为了让我们更加庄主与代码而不是环境。对于如何在主流IDE上使用Leap Motion SDK创建工程，可以先看看这篇文章[Setting up a Project](https://developer.leapmotion.com/documentation/java/devguide/Project_Setup.html)。

1. 如果你还没有准备好，那么可以从[Developer site](https://developer.leapmotion.com/)下载解压最新版本的Leap Motion SDk，然后安装最新的Leap Motion服务。
2. 打开一个终端或者控制台窗口，导航到SDk 'Sample'文件夹下。
3. 'Sample.java'包含了这篇教程的最终代码，但是出于能够更加深入理解本篇教程的目的下，你可以重命名已经存在的这个Sample文件，然后在这个文件夹下创建一个新的、空白的'Sample.java'文件。保留已经存在的Sample文件，以供参考。
4. 在你的'Sample.java'文件中，添加导入Leap Motion库的代码：

```ruby

import com.leapmotion.leap.*;


```

5. 添加主题代码，定义一个java命令行程序：

```ruby

class Sample {
    public static void main(String[] args) {

        // Keep this process running until Enter is pressed
        System.out.println("Press Enter to quit...");
        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

这个代码简单的打印了一条信息，然后等键盘输入任意键后退出。

### Get Connected

下一步是在程序中添加一个控制器对象[Controller](https://developer.leapmotion.com/documentation/java/api/Leap.Controller.html)--这个对象将会帮助我们连接到Leap Motion 服务/守护线程。

```ruby

class Sample {
    public static void main(String[] args) {
        Controller controller = new Controller();

        // Keep this process running until Enter is pressed
        System.out.println("Press Enter to quit...");
        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

当你创建一个Controller对象的时候，他会自动的链接到Leap Motion设备，一旦连接建立，你可以通过使用Controlller.frame()方法得到所要的跟踪数据。

这个连接的过程是异步的，所以你不能在一行创建'Controller'，然后期待能偶在下一行语句得到数据。你必须等到连接完成之后才行。但是这需要多久呢？

### To Listen or not to Listen?

你可以在Controlller对象上添加监听器对象[Listener](https://developer.leapmotion.com/documentation/java/api/Leap.Listener.html)，这回提供一个基于事件的响应机制以响应'Controlller'的状态变化。这是本篇教程使用的方法--但是他也不总是最好的办法。

**监听器的问题：**Listener对象使用了独立的线程来调用你实现的每一个事件的代码。所以，使用监听器机制可以展示出贯穿你的代码中的复杂性。你需要确保的是，在你的监听器子类中实现的代码能够以一种线程安全的方式访问到你的程序。比如，你可能无法访问到跟GUI控制器有关的变量，除了主线程以外。此外，还可能有与线程的创建和清理相关的额外开销。

**Avoiding Listeners:**你可以禁止使用监听器对象通过简单地轮询Controller对象的帧(或其他的状况)，如果对于你的程序是必要的话。很多程序都已经有了一个事件循环，或者动画循环来驱动用户的输入和动画。如果这样的话，你可以在每一轮的循环中获得数据，这将尽可能的加快你使用数据的速度。

API中监听器类定义了当Controller事件触发时会被调用的函数的签名。你可以通过创建'listener'的子类，和实现你感兴趣的事件的回调函数的方式，创建一个监听器。

那么，继续我们的教程，添加'SampleListener'类到你的程序之中。为了简单，我们添加'SampleListener'类到'Sample'类一样的文件下面吧。

```ruby

class SampleListener extends Listener {

    public void onConnect(Controller controller) {
        System.out.println("Connected");
    }

    public void onFrame(Controller controller) {
        System.out.println("Frame available");
    }
}

```

如果你已经看过了官方给的示例代码，你可能已经注意到了，回调函数的多次出现。你可能也想着吧他们都添加到自己的文件里面，如果你希望的话，但是为了保持代码简单，我们希望只专注于0nConnect()和onFrame()方法。

现在使用你刚写的类创建一个'SampleListener'对象。然后把它添加到你的控制器里。

```ruby

class Sample {
    public static void main(String[] args) {
        // Create a sample listener and controller
        SampleListener listener = new SampleListener();
        Controller controller = new Controller();

        // Have the sample listener receive events from the controller
        controller.addListener(listener);

        // Keep this process running until Enter is pressed
        System.out.println("Press Enter to quit...");
        try {
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Remove the sample listener when done
        controller.removeListener(listener);
    }
}

```

现在，你恶意运行测试一下你的'Sample'程序了。可以跳到后面查看Running the Sample的内容。

如果代码什么的搜没有错误的话(还有就是你的Leap Motion硬件设备已经连接到计算机)，那么你会看到这样的字符串"Connect"打印在终端窗口上，紧接着的是一系列以很快的速度输出的"Frame available"。如果报错而且你搞不懂为什么的话，你可以在我们的[开发者论坛](https://developer.leapmotion.com/)上向我们寻求帮助。

无论你什么时候开发出错，都可以试着打开可视化诊断工具。这个程序将展示一个Leap Motion跟踪数据的可视化观察器。你可以比较你在程序中看到的和你在可视化观察器中看到的(这两者都是用的相同的API)，来查找问题。

### On Connect

当你的控制器对象成功连接到Leap Motion服务/守护线程之后，而且你的Leap Motion硬件已经插入计算机，控制器对象将会改变它的isConnect()方法的属性为'true'，然后调用你的[Listener.onConnect](https://developer.leapmotion.com/documentation/java/api/Leap.Listener.html#javaclasscom_1_1leapmotion_1_1leap_1_1_listener_1a06446e436cca86cdd72c0d88c487d52d)的回调。

控制器连接之后，你可以使用[Controller.enableGesture()](https://developer.leapmotion.com/documentation/java/api/Leap.Controller.html#javaclasscom_1_1leapmotion_1_1leap_1_1_controller_1ac2ee4e779963f135abccc183ab2f35fb)和[Controller.setPolicy()](https://developer.leapmotion.com/documentation/java/api/Leap.Controller.html#javaclasscom_1_1leapmotion_1_1leap_1_1_controller_1a272a235782d654778a1873a28f49a794)方法设置控制器的参数。比如：你可以通过以下'onConnect()'方法来设置滑动手势的使能化：

```ruby

public void onConnect(Controller controller) {
    System.out.println("Connected");
    controller.enableGesture(Gesture.Type.TYPE_SWIPE);
}

```

### On Frame

在Leap Motion系统中，所有跟踪数据送达都是通过[Frame](https://developer.leapmotion.com/documentation/java/api/Leap.Frame.html)对象来实现的。你可以从你的控制器(当他连接之后)通过调用Controller.frame()方法来获得'Frame'对象。当一个新的frame数据有效时，你的Listener子类的onFrame()回调方法将会被调用。当你不在使用监听器时，你可以比较[id()](https://developer.leapmotion.com/documentation/java/api/Leap.Frame.html#javaclasscom_1_1leapmotion_1_1leap_1_1_frame_1a912887c5fe73e39a0463019d92107ab0)的值和你处理的上一帧，判断是否是一个新的frame(帧)。注意，通过设置frame()函数的'history'参数，你可以获得更早之前的帧(里面至少有60以上的帧的数据被保存着)。所以，如有必要的话，你可以使轮询速率变得比Leap Motion数据帧的速率还慢，可以处理每一帧的数据。

为了获得数据帧frame，要添加frame()方法到你的onframe()回调函数里面：

```ruby

public void onFrame(Controller controller) {
    Frame frame = controller.frame();
}

```

然后，打印出Frame对象的一些属性：

```ruby

public void onFrame(Controller controller) {
    Frame frame = controller.frame();

    System.out.println("Frame id: " + frame.id()
                   + ", timestamp: " + frame.timestamp()
                   + ", hands: " + frame.hands().count()
                   + ", fingers: " + frame.fingers().count()
                   + ", tools: " + frame.tools().count()
                   + ", gestures " + frame.gestures().count());
}

```

那么，再次运行你的Sample，把一只或者两支手掌放在Leap Motion设备上面，你会在控制台窗口看到每一帧的基本属性。

本次教程在这就结束了，但是你可以看看官方提供的Sample文件中的其他代码。比如说，在一个Frame里面如何获得手掌Hand，手指Finger，手臂Arm，骨头Bone 还有手势Gesture对象。

### Running the Sample

运行一个示例应用：

1. 编译示例应用：
    - 在Windows系统下，确保'Sample.java'和'LeapJava.jar'在当前文件夹下面，然后输入以下命令行在命令行提示符后面，并运行：
    ```ruby
    javac -classpath LeapJava.jar Sample.java
    ```
    
    - 在Mac系统上，确保'Sample.java'和'LeapJava.jar'在当前文件夹下面，然后在终端里面运行以下命令行：
    
    ```ruby
    
    javac -classpath ./LeapJava.jar Sample.java
    
    ```
    
    
2. 运行示例应用：
    - 在Windows系统下，确保'Sample.class','LeapJava.jar','LeapJava.dll',和'Leap.dll'在同一个文件夹下。如果你是用的是32-bit版本的JVM，需要使用SDk下的lib\x86文件夹下的.dll文件。如果你是用的是64-bit版本的JVM，需要使用SDk下的lib\x64文件夹下的.dll文件.在命令行提示符后输入并运行以下命令行：
    ```ruby
    java -classpath "LeapJava.jar;." Sample
    ```

    - 在Mac系统下，确保'Sample.class','LeapJava.jar','LeapJava.dll',和'Leap.dll'在同一个文件夹下。在终端中运行命令行：
    
    ```ruby
    
    java -classpath "./LeapJava.jar:." Sample
    
    ```

当应用程序初始化并连接之后到Leap之后，你会看到"Connect"信息被打印在标准输出中。你还会看到frame的信息，在每一次'onframe'事件被调用后。


