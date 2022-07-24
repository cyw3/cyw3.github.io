---
title: Gaussian Blur
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---

## 前言

玩了OpenCV之后，感觉自己能分分钟写出个PS软件，真是嘚瑟了。不过，这倒是也是个方向。所以，现在就来写个滤镜吧。

说到底，在计算机眼中，所谓的图片就是一个矩阵，对于他的处理跟线性代数是极其相关的，所以大学狗还是认真学习去吧。好了，不闹了。说到滤镜，说到矩阵，当然要想到PS软件里面的"Gaussian Blur"啦。

## 目录

1. Gaussian Blur的介绍

2. Gaussian Blur的原理

3. Gaussian Blur的例子以及代码实现

## 1. Gaussian Blur的介绍

高斯模糊（英语：Gaussian Blur），也叫高斯平滑，是在Adobe Photoshop、GIMP以及Paint.NET等图像处理软件中广泛使用的处理效果，通常用它来减少图像噪声以及降低细节层次。

这种模糊技术生成的图像，其视觉效果就像是经过一个半透明屏幕在观察图像，这与镜头焦外成像效果散景以及普通照明阴影中的效果都明显不同。高斯平滑也用于计算机视觉算法中的预先处理阶段，以增强图像在不同比例大小下的图像效果。 从数学的角度来看，图像的高斯模糊过程就是图像与正态分布做卷积。由于正态分布又叫作高斯分布，所以这项技术就叫作高斯模糊。图像与圆形方框模糊做卷积将会生成更加精确的焦外成像效果。由于高斯函数的傅立叶变换是另外一个高斯函数，所以高斯模糊对于图像来说就是一个低通滤波器。

本文介绍"高斯模糊"的算法，这是一个非常简单易懂的算法。本质上，它是一种[数据平滑技术（data smoothing）](https://en.wikipedia.org/wiki/Smoothing)。


## 2. Gaussian Blur的原理

> 所谓"模糊"，可以理解成每一个像素都取周边像素的平均值。

是的，"平均值"是**模糊**的关键。他能够将清晰度高的照片，运用平均值，将图片的细节隐藏，使这个像素矩阵的变化更为平滑。

咱们先来看看个例子：

![gs0]({{site.baseurl}}/img/gs0.png)

还未进行模糊处理过的图片，里面的像素数值是不定，细节暴露的。就比如上图，中间数值是2，周边是1。

![gs1]({{site.baseurl}}/img/gs1.png)

"中间点"取"周围点"的平均值，就会变成1。在数值上，这是一种"平滑化"。在图形上，就相当于产生"模糊"效果，"中间点"失去细节。

显然，计算平均值时，取值范围越大，"模糊效果"越强烈。

![gs2]({{site.baseurl}}/img/gs2.jpg)

上面分别是原图、模糊半径3像素、模糊半径10像素的效果。模糊半径越大，图像就越模糊。从数值角度看，就是数值越平滑。

这就是*模糊*的原理。而以上这个例子的模糊算法便是最常见的3x3模板的均值模糊，即取一定半径内的像素值之平均值作为当前点的新的像素值。

其实，这个时候，本文也算是结束了。因为，你会发现，其他图像模糊算法是大同小异的，只是在求均值的时候因为侧重点的不同而赋予这个矩阵的各个像素点以不同的权重而已。不过，我还是会继续讲下去的，不然会被打死。

那么我们接着来，大家一定听说过'Gaussian'这位伟大的学者吧。那么他有名的"高斯分布"，也就是"正态分布"也是知道的吧。对的，就是你想的那样。所谓的"高斯模糊"，就是将*高斯分布*和*模糊*两者结合起来的，然后就随随便便的冠上了伟大的"Gsussian"的名头，让人还以为高大上，很难，其实就是求均值。

前面我们说到了"权重"，是想让这个矩阵中筛选出对于中间点是相对更加重要的点，并给它在求均值的时候更大的影响力。而高斯模糊中的权重函数，真是高斯分布。

![gs3]({{site.baseurl}}/img/gs3.png)

由高斯分布可以知道，距离中点越近的点，显然他的权重是越大的；相反，越远，权重就越小。

计算平均值的时候，我们只需要将"中心点"作为原点，其他点按照其在正态曲线上的位置，分配权重，就可以得到一个加权平均值。

接下来祭出高斯分布的密度函数的计算公式：

![GAOSI]({{site.baseurl}}/img/GAOSI.png)

其中，μ是x的均值，σ是x的方差。因为计算平均值的时候，中心点就是原点，所以μ等于0。

![gaosi1]({{site.baseurl}}/img/gaosi1.png)

又因为像素矩阵是二维的，而现在这个密度函数是一维的。根据一维高斯函数，可以推导得到二维高斯函数：

![gaosi2]({{site.baseurl}}/img/gaosi2.png)

## 3. Gaussian Blur的步骤以及代码实现

根据高斯模糊的原理，我们已经知道，要根据二维高斯函数求取像素矩阵的均值，以此来进行模糊处理。

1）求取权重矩阵

咱们还是使用的阮一峰老师博客的例子：

假定使用的3x3模板，中心点的坐标是（0,0），那么距离它最近的8个点的坐标如下，更远的点以此类推：

![gs5]({{site.baseurl}}/img/gs5.png)

为了计算权重矩阵，需要设定σ的值。假定σ=1.5，分别将这个九宫格中的各个点的坐标代入上面的二维高斯函数计算公式，求出各自的G(x,y).则模糊半径为1的权重矩阵如下：

![gs6]({{site.baseurl}}/img/gs6.png)

这9个点的权重总和等于0.4787147，如果只计算这9个点的加权平均，还必须让它们的权重之和等于1，因此上面9个值还要分别除以0.4787147，得到最终的权重矩阵。

![gs7]({{site.baseurl}}/img/gs7.png)


2）计算高斯模糊

就是说计算中间点的值。

假设现有9个像素点，灰度值（0-255）如下：

![gs8]({{site.baseurl}}/img/gs8.png)

各自乘上各自对应的权重。

![gs9]({{site.baseurl}}/img/gs9.png)

![gs10]({{site.baseurl}}/img/gs10.png)

将这九个值相加即可得到中间点的数值。这就是中间点高斯模糊值。

对所有点重复这个过程，就得到了高斯模糊后的图像。如果原图是彩色图片，可以对RGB三个通道分别做高斯模糊。

对于边缘的点，就是把已有的点拷贝到另一面的对应位置，模拟出完整的矩阵。


那么接下是[代码实现](https://github.com/cyw3/CywLeetCode/blob/master/LeetCode/src/com/cyw/algorithms/GaussianBlur.java)：

{% highlight ruby %}

import java.awt.Color;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import javax.imageio.ImageIO;
 
public class GaussianBlur {

	/**
　*　简单高斯模糊算法
　*　@param　args
　*　@throws　IOException　[参数说明]
　*　@return　void　[返回类型说明]
　*　@exception　throws　[违例类型]　[违例说明]
　*　@see　[类、类#方法、类#成员]
　*/
	public static void main(String [] args) throws IOException {
		BufferedImage img = ImageIO.read(new File("BigData.jpg"));
		System.out.println(img);
		int height = img.getHeight();
		int width = img.getWidth();
		
		/**
		 * 计算一个九宫格的平均值
		 */
		int[][] matrix = new int[3][3];
		int[] values = new int[9];
		for (int i = 0;i<width;i++){
			for(int j = 0;j<height;j++){
				readPixel(img,i,j,values);
				fillMatrix(matrix,values);
				img.setRGB(i, j, avgMatrix(matrix));
			}
		}
		ImageIO.write(img,"jpeg",new File("test.jpg"));//保存在工程为test.jpeg文件
	}
 
	private static void readPixel(BufferedImage img, int x, int y, int[] pixels){
		int xStart = x - 1;
		int yStart = y - 1;
		int current = 0;
		//
		for (int i=xStart;i<3+xStart;i++){
			for(int j=yStart;j<3+yStart;j++){
				int tx=i;
				if(tx<0){
					tx=-tx;
				}else if(tx>=img.getWidth()){
					tx=x;
				}

				int ty=j;
				if(ty<0){
					ty=-ty;
				}else if(ty>=img.getHeight()){
					ty=y;
				}
				pixels[current++]=img.getRGB(tx,ty);
			}
		}
	}
 
	private static void fillMatrix(int[][] matrix, int... values) {
		int filled=0;
		for(int i=0;i<matrix.length;i++){
			int[] x=matrix[i];
			for(int j=0;j<x.length;j++){
				x[j]=values[filled++];
			}
		}
	}
 
	private static int avgMatrix(int[][] matrix){
		int r=0;
		int g=0;
		int  b=0;
		for(int i=0;i<matrix.length;i++){
			int[] x=matrix[i];
			for(int j=0;j<x.length;j++){
				if(j==1){
					continue;
				}
				Color c=new Color(x[j]);
				r+=c.getRed();
				g+=c.getGreen();
				b+=c.getBlue();
			}
		}
		return new Color(r/8,g/8,b/8).getRGB();
	}
}
{% endhighlight %}


再来个wiki上面的例子吧，大家可以巩固一下：

[![GSexample]({{site.baseurl}}/img/GSexample.png)](https://zh.wikipedia.org/wiki/%E9%AB%98%E6%96%AF%E6%A8%A1%E7%B3%8A#.E9.AB.98.E6.96.AF.E7.9F.A9.E9.98.B5.E7.A4.BA.E4.BE.8B)


