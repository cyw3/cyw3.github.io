---
title: OpenCV with CodeBlocks
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---

## 前言

近年，机器学习当火。而OpenCV便是机器学习在图像识别领域的热门框架。恰好近来闲来无事，便开始捣鼓一番OpenCV。

## 目录

1. OpenCV介绍
2. minGW版OpenCV配置方式
3. CodeBlocks IDE示例代码实现

## 1.OpenCV介绍

OpenCV的全称是：Open Source Computer Vision Library。OpenCV是一个基于BSD许可（开源）发行的跨平台计算机视觉库，可以运行在Linux、Windows和Mac OS操作系统上。它轻量级而且高效——由一系列 C 函数和少量 C++ 类构成，同时提供了Python、Ruby、MATLAB等语言的接口，实现了图像处理和计算机视觉方面的很多通用算法。

为什么有OpenCV？

计算机视觉市场巨大而且持续增长，且这方面没有标准API，如今的计算机视觉软件大概有以下三种：

1. 研究代码（慢，不稳定，独立并与其他库不兼容）
2. 耗费很高的商业化工具（比如Halcon, MATLAB+Simulink）
3. 依赖硬件的一些特别的解决方案（比如视频监控，制造控制系统，医疗设备）这是如今的现状。而标准的API将简化计算机视觉程序和解决方案的开发。OpenCV致力于成为这样的标准API。

OpenCV致力于真实世界的实时应用，通过优化的C代码的编写对其
执行速度带来了可观的提升，并且可以通过购买Intel的IPP高性能多媒体函数库(Integrated Performance Primitives)得到更快的处理速度。

主要应用方向有：人机互动，人脸识别，运动跟踪，图像分割等等。

## 2.minGW版OpenCV配置方式

首先，我们来看看需要准备的什么吧。


Win7系统

[mingw-get-setup.exe](http://sourceforge.net/projects/mingw/files/Installer/)

[opencv-2.4.9.exe](http://opencv.org/downloads.html)

[cmake-3.2.0-rc1-win32-x86.exe](http://www.cmake.org/download/)


然后，双击opencv-2.4.9.exe，这是opencv的一个自解压程序，指定好安装目录，他将自动把.exe里的文件解压到指定目录。现在，咱们来看看OpenCV的目录结构吧。

> opencv
>> build、sources

> build
>> doc、include、java、python、x64、x86、LICENSE、OpenCVConfig.cmake、OpenCVConfig-version.cmake

可知，在opencv目录下有两个文件夹，分别为build、以及source。其中source是OpenCV的源码。而build是各个语言版本编译过后的库文件，包含c、c++、java、python等语言。x86文件夹和x86文件夹分别对应的是32-bit、64-bit系统下的库。在x64、x86中，有如下目录结构：

> vc10、vc11、vc12

分别对应的是Virtual Studio2010、2011、2012编译后的库。因为不同的编译器(即便相同的编译器，但不同版本，编译后的文件也是不同的)。同编译器编译出来的OpenCV，只能在相应的编译环境下运行。在早前的如OpenCV 2.3.1中还有vc9（对应vs2008）和mingw版本，而在2.4版本之后便只有vc系列了。

而现在，因为我们使用的编译器是MinGW，而该版本的OpenCV没有对应的编译后文件，所以，我们不得不来自己对源码进行编译。

那么，接下来，我们便开始编译工作了。

1)安装MinGW。


很简单。直接双击，指定安装目录，确认安装即可。


2)出现以下界面，至少点击以下两图所示的条目，并安装。


![MinGW0]({{site.baseurl}}/img/MinGW0.png)

![MinGW1]({{site.baseurl}}/img/MinGW1.png)


3)安装cmake

4)运行cmake，在where is the source code中填入OpenCV源代码文件的路径，比如：“.../opencv_2410/sources”；在where to build the binaries中填入编译文件需要存放的路径，比如：“.../MinGW/Debug”(存放路径文件自己定义新建一个即可)。(三点表示省略)

![cmake]({{site.baseurl}}/img/cmake.jpg)

5)点击“Configure”;在Specify the generator for this project中选择MinGW Makefiles(选择刚刚安装的MinGW或者本机已有的)，选中Specify native compilers，点击“Next”。

![cmake1]({{site.baseurl}}/img/cmake1.jpg)

6)选择编译器路径，这里Compilers: C 选择目录为“.../MinGW/bin/gcc.exe”; C++ 选择目录为 “.../MinGw/bin/g++.exe”，点击“Finish”

![cmake2]({{site.baseurl}}/img/cmake2.jpg)

7)然后再次点“Configure”

![cmake3]({{site.baseurl}}/img/cmake3.jpg)

8)等走完进度条，选择需要的Generate选项，此处可以不操作直接点“Generate”，走完进度条便生成了“MinGW Makefiles”

![cmake4]({{site.baseurl}}/img/cmake4.jpg)

9)之后用mingw对其进行编译，cmd打开命令提示符窗口，进到刚才的保存目录，这里是“.../opencv2.4.10/MinGW/Debug”，输入“mingw32-make”，回车；等待运行完毕后，输入 mingw32-make install,回车；（此过程大约需1-2个小时）

10)运行完毕后便生成了mingw版的OpenCV库，进入“.../opencv2.4.10/MinGW/Debug/install”文件夹，便可以看到所需的头文件和库文件

![cmake5]({{site.baseurl}}/img/cmake5.jpg)


## 3.CodeBlocks IDE示例代码实现

我所选用的IDE是

[codeblocks-13.12mingw-setup.exe](http://www.codeblocks.org/)


codeblocks默认使用的编译器是MinGW。

1)双击安装codeblocks

2)启动codeblocks，新建一个“Console application”项目，任意取一个名字。

3)测试代码如下

{% highlight ruby %}
#include "cv.h"  
#include "highgui.h"  
int main()  
{  
    IplImage* img = cvLoadImage("test.jpg");  
    cvNamedWindow( "test", 0 );  
    cvShowImage("test", img);  
    cvWaitKey(0);  
    cvReleaseImage( &img );  
    cvDestroyWindow( "test" );  
    return 0;  
}  
{% endhighlight %}

4)设置opencv相关头文件以及库文件路径

这一步是关键。因为还是有点麻烦的。

(1)右击项目名称，选build options：

![codeblocks0]({{site.baseurl}}/img/codeblocks0.jpg)

(2)弹出窗口，首先添加头文件路径，依次点击：Search directories->Complier->Add，选择头文件所在目录

![codeblocks1]({{site.baseurl}}/img/codeblocks1.png)

(3)选择库文件路径，依次点击Linker->Add，选择vc10下的lib库路径

![codeblocks2]({{site.baseurl}}/img/codeblocks2.png)

(4)最后点击 Linker settings，添加相应库文件，这里如果不知道自己会用到那些库文件的话，可以将vc10/lib下的所有库文件全部添加进去

![codeblocks3]({{site.baseurl}}/img/codeblocks3.png)

5)动态库调用设置

两种方法任选其一即可：

1.可以设置系统变量，将“...\opencv\build\x86\vc10\bin;”添加到系统变量中

2.将...\opencv\build\x86\vc10\bin下的所有dll文件复制到codeblocks对应工程下的bin\Debug文件夹下即可（如果会分辨opencv库文件的话，也可以只将用到的动态库复制即可）。
