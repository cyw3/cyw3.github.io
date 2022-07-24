---
title: 凸包算法剖析
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---

## 前言

前天，刚考完证券，以前帮别人搞金融时候的一些问题也算有了点自己的答案。当然，今天不是说这个。
正好也是前几天，好基友跑来问我凸包算法，一时就楞住了，好像还真没有搞过计算机几何方面的。于是赶忙就Google了下，把凸包的几种算法都搞了个清楚。所以，今天就来讲讲凸包算法吧。

## 何为凸包

首先，我们的搞清楚，什么叫做凸包。
假设平面上有p0~p12共13个点，过某些点作一个最小多边形，使这个多边形能把所有点都“包”起来。当这个多边形是凸多边形的时候，我们就叫它“凸包”。如图： 

![convexHull1.gif]({{site.baseurl}}/img/convexHull1.gif)

换种说法便是：令S是平面上的一个点集，封闭S中所有顶点的最小凸多边形，称为S的凸包。

然后，什么是凸包问题？ 我们把这些点放在二维坐标系里面，那么每个点都能用 (x,y) 来表示。 现给出点的数目为13，已知各个点的坐标，求构成凸包的点。

## 解一：穷举法（暴力法）

时间复杂度：O(n³）。 

思路：两点确定一条直线，如果剩余的其它点都在这条直线的同一侧，则这两个点是凸包上的点，否则就不是。

步骤：

1、将点集里面的所有点两两配对，组成 n(n-1)/2 条直线。

2、对于每条直线，再检查剩余的 (n-2) 个点是否在直线的同一侧。

如何判断一个点 p3 是在直线 p1p2 的左边还是右边呢？（坐标：p1(x1,y1)，p2(x2,y2)，p3(x3,y3)）

![convexhull2.png]({{site.baseurl}}/img/convexhull2.png)

当上式结果为正时，p3在直线 p1p2 的左侧；当结果为负时，p3在直线 p1p2 的右边。

{% highlight ruby %}

package com.cyw.algorithms;

import java.util.*;

public class ConvexHullExhaustive {
	public static class Line {//线
		 Point p1, p2;
		 Line(Point p1, Point p2) {
		  this.p1 = p1;
		  this.p2 = p2;
		 }
		}

	public static class Point{//点
		  int x;
		  int y;
		}

	List<Point> pts = null;//点集
	List<Line> lines = new ArrayList<Line>();//点集pts的凸包

	public void setPointList(List<Point> pts) {
	     this.pts = pts;
	}

	public List<Line> eval() {
		    lines.clear();
		    if (pts == null) { return lines; }
		    int n = pts.size();
		    int a, b, c, cc, l, r;
		    for (int i = 0; i < n; i++) {
		     for (int j = i+1; j < n; j++) {//穷举点集中任意两点组成的直线ax+by=c
		       a = pts.get(j).y - pts.get(i).y;//
		       b = pts.get(i).x - pts.get(j).x;
		       c = pts.get(i).x * pts.get(j).y -pts.get(i).y * pts.get(j).x;
		       l = r = 0;
		       int k;
		       for (k = 0; k < n; k++) {//穷举点集中的每一点
		         cc = a * pts.get(k).x + b * pts.get(k).y;
		         if (cc > c) l++;
		         else if (cc < c) r++;
		         if (l * r != 0) break;//直线两侧都有点,
		       }
		      if (k == n) {//凸包中添加一条边
		        lines.add(new Line(pts.get(i), pts.get(j)));
		      }
		     }
		   }
		   return lines;
		  }
}

{% endhighlight %}

## 解二：分治法

时间复杂度：O(n㏒n)。

思路：应用分治法思想，把一个大问题分成几个结构相同的子问题，把子问题再分成几个更小的子问题。然后我们就能用递归的方法，分别求这些子问题的解。最后把每个子问题的解“组装”成原来大问题的解。 

步骤：

1、把所有的点都放在二维坐标系里面，在横坐标方向上按照大小排序。那么横坐标最小和最大的两个点 P1 和 Pn 一定是凸包上的点。直线 P1Pn 把点集分成了两部分，即 X 轴上面和下面两部分，分别叫做上包和下包。

2、对上包，找到一个距离直线 P1Pn 最远的点，即下图中的点 Pmax 。

3、作直线 P1Pmax 、PnPmax，把直线 P1Pmax 左侧的点当成是上包，把直线 PnPmax 右侧的点也当成是上包。显然直线段P1Pmax与直线段PmaxPn把点集分为三个集合。由凸包的性质可控制，P1PmaxPn三点围成的三角形中的点不可能作为凸包的顶点，所以只需考虑直线P1Pmax左边的点以及直线PmaxPn右边的点。

4、重复步骤 2、3。递归求解得到凸多边形的边。

5、对下包也作类似操作。

6、合并这些边即得所求的凸包。

![convexhull3.png]({{site.baseurl}}/img/convexhull3.png)

然而怎么求距离某直线最远的点呢？我们还是用到解一中的公式： 

![convexhull2.png]({{site.baseurl}}/img/convexhull2.png)

设有一个点 P3 和直线 P1P2 （坐标：p1(x1,y1)，p2(x2,y2)，p3(x3,y3)），对上式的结果取绝对值，绝对值越大，则距离直线越远。

注意：在步骤一，如果横坐标最小的点不止一个，那么这几个点都是凸包上的点，此时上包和下包的划分就有点不同了，需要注意。

{% highlight ruby %}

package com.cyw.algorithms;

import java.util.*;   

public class ConvexHullDivideAndConquer {

	/*
	*	分治法求凸包
	*/
//	public class QuickTuBao {
	  private List<Point> pts = null;//给出的点集S
	  private List<Line> lines = new ArrayList<Line>();//点集pts的凸包，多边形上的边以及边上的顶点
		
	  public void setPointList(List<Point> pts) {
	      this.pts = pts;
	  }

	  public ConvexHullDivideAndConquer(List<Point> pts){
	      this.pts=pts;
	  }

	  public ConvexHullDivideAndConquer(){
	  }
	  
	  //求凸包，结果存入lines中
	  public List<Line> eval() {
	      lines.clear();
	      if (pts == null || pts.isEmpty()) { return lines; }
	      
	      List<Point> ptsLeft = new ArrayList<Point>();//左凸包中的点
	      List<Point> ptsRight = new ArrayList<Point>();//右凸包中的点
			
	        //按x坐标对pts排序，从小到大
	       Collections.sort(pts, new Comparator<Point>() {
	             public int compare(Point p1, Point p2) {
	              if(p1.x-p2.x>0) return 1;
	              if(p1.x-p2.x<0) return -1;
	              return 0;
	             } 
				
			});  
	         
	            Point p1 = pts.get(0);//最左边的点
	          //最右边的点,用直线p1p2将原凸包分成两个小凸包
	            Point p2 = pts.get(pts.size()-1);
	            Point p3 = null;
	            double area = 0;
	            for (int i = 1; i < pts.size()-1; i++) {//穷举所有的点,
		              p3 = pts.get(i);
		            //求此三点所成三角形的有向面积。对点进行分类，是这两点直线的左边还是右边
		              area = getArea(p1, p2, p3);
		              if (area > 0) {
		                 ptsLeft.add(p3);//p3属于左
		                } else if (area < 0) {
		                  ptsRight.add(p3);//p3属于右
		                }
	              }
	            
	              d(p1, p2, ptsLeft);//分别求解
	              d(p2, p1, ptsRight);
	              return lines;
	   }

	  //递归
	   private void d(Point p1, Point p2, List<Point> s) {
	     //s集合为空
	     if (s.isEmpty()) {
	       lines.add(new Line(p1, p2));
	       return;
	     }
	     //s集合不为空，寻找Pmax
	     double area = 0;
	     double maxArea = 0;
	     Point pMax = null;
	     for (int i = 0; i < s.size(); i++) {
		      area = getArea(p1, p2, s.get(i));//最大面积对应的点就是Pmax
		      if (area > maxArea) {
		        pMax = s.get(i);
		        maxArea = area;
		       }
	      }
	      //找出位于(p1, pMax)直线左边的点集s1
	      //找出位于(pMax, p2)直线左边的点集s2
	       List<Point> s1 = new ArrayList<Point>();
	       List<Point> s2 = new ArrayList<Point>();
	       Point p3 = null;
	       for (int i = 0; i < s.size(); i++) {
		         p3 = s.get(i);
		         if (getArea(p1, pMax, p3) > 0) {
		            s1.add(p3);
		         } else if (getArea(pMax, p2, p3) > 0) {
		            s2.add(p3);
		         } 
	        }
	       //递归
	      d(p1, pMax, s1);
	      d(pMax, p2, s2);	
	    }
	   
		// 三角形的面积等于返回值绝对值的二分之一
		// 当且仅当点p3位于直线(p1, p2)左侧时，表达式的符号为正
		private double getArea(Point p1, Point p2, Point p3) {
	           return p1.x * p2.y + p3.x * p1.y + p2.x * p3.y -
	             p3.x * p2.y - p2.x * p1.y - p1.x * p3.y;
		}
//	}
		
	public static class Line {//线   
		 Point p1, p2;   
		 Line(Point p1, Point p2) {   
			  this.p1 = p1;   
			  this.p2 = p2;   
		 }   
		  public double getLength() {
			double dx = Math.abs(p1.x - p2.x);
			double dy = Math.abs(p1.y - p2.y);
			return Math.sqrt(dx * dx + dy * dy);
		  }
	}   
		  
	public static class Point{//点   
	  double x;   
	  double y;   
	  public Point(double x,double y){
	     this.x=x;
	     this.y=y;
	  }
	}   

	
	public static void main(String [] args){
		ConvexHullDivideAndConquer ch = new ConvexHullDivideAndConquer();
		List<Point> pts = new ArrayList<Point>();
		pts.add(new Point(0,1));
		pts.add(new Point(0,3));
		pts.add(new Point(1,2));
		pts.add(new Point(2,2));
		pts.add(new Point(3,1));
		pts.add(new Point(3,3));
		
		List<Line> result = new ArrayList<Line>();
		ch.setPointList(pts);
		result = ch.eval();
		
		for(Line l : result){
			System.out.println("("+l.p1.x+","+l.p1.y+")("+l.p2.x+","+l.p2.y+")");
		}
	}
}

{% endhighlight %}

## 解三：Jarvis步进法

时间复杂度：O(nH)。（其中 n 是点的总个数，H 是凸包上的点的个数）

思路：

1、纵坐标最小的那个点一定是凸包上的点，例如图上的 P0。

2、从 P0 开始，按逆时针的方向，逐个找凸包上的点，每前进一步找到一个点，所以叫作步进法。

3、怎么找下一个点呢？利用夹角。假设现在已经找到 {P0，P1，P2} 了，要找下一个点：剩下的点分别和 P2 组成向量，设这个向量与向量P1P2的夹角为 β 。当 β 最小时就是所要求的下一个点了，此处为 P3 。

![convexhull4.png]({{site.baseurl}}/img/convexhull4.png)

注意：

1、找第二个点 P1 时，因为已经找到的只有 P0 一个点，所以向量只能和水平线作夹角 α，当 α 最小时求得第二个点。

2、共线情况：如果直线 P2P3 上还有一个点 P4，即三个点共线，此时由向量P2P3 和向量P2P4 产生的两个 β 是相同的。我们应该把 P3、P4 都当做凸包上的点，并且把距离 P2 最远的那个点（即图中的P4）作为最后搜索到的点，继续找它的下一个连接点。

## 解四：Graham扫描法

时间复杂度：O(n㏒n)

思路：Graham扫描的思想和Jarris步进法类似，也是先找到凸包上的一个点，然后从那个点开始按逆时针方向逐个找凸包上的点，但它不是利用夹角，而是利用了左转判定。

左转判定 ，这一思想在凸包算法里十分的重要，是如何判断两个向量 p1=(x1,y1) 和 p2=(x2,y2) 是否左转的一个判定方法。非常简单的，只需判断 x1*y2-x2*y1 的正负值即可，为正说明 p1 到 p2 为左转。即是说，它可以用来判断一个向量到另一个向量是逆时针转还是顺时针转。

![convexhull5.png]({{site.baseurl}}/img/convexhull5.png)

步骤：

1、把所有点放在二维坐标系中，则纵坐标最小的点一定是凸包上的点，如图中的P0。

2、把所有点的坐标平移一下，使 P0 作为原点，如上图。

3、计算各个点相对于 P0 的幅角 α ，按从小到大的顺序对各个点排序。当 α 同时，距离 P0 比较近的排在前面。例如上图得到的结果为P1，P2，P3，P4，P5，P6，P7，P8。我们由几何知识可以知道，结果中第一个点 P1 和最后一个点 P8 一定是凸包上的点。 

以上，我们已经知道了凸包上的第一个点P0和第二个点P1，我们把它们放在栈里面。现在从步骤3求得的那个结果里，把 P1 后面的那个点拿出来做当前点，即 P2 。接下来开始找第三个点：

4、连接P0和栈顶的那个点，得到直线 L 。看当前点是在直线 L 的右边还是左边。如果在直线的右边就执行步骤5；如果在直线上，或者在直线的左边就执行步骤6。

5、如果在右边，则栈顶的那个元素不是凸包上的点，把栈顶元素出栈。执行步骤4。

6、当前点是凸包上的点，把它压入栈，执行步骤7。

7、检查当前的点 P2 是不是步骤3那个结果的最后一个元素。是最后一个元素的话就结束。如果不是的话就把 P2 后面那个点做当前点，返回步骤4。

最后，栈中的元素就是凸包上的点了。 
以下为用Graham扫描法动态求解的过程： 

![convexhull6.gif]({{site.baseurl}}/img/convexhull6.gif)

{% highlight ruby %}

import java.util.Scanner;
public class ConvexHullWithGraham { 
     Point[] ch; //点集p的凸包
     Point[] p ; //给出的点集
     int n;
     int l;
     int len=0;
    public ConvexHullWithGraham(Point[] p,int n,int l){
       this.p=p;
       this.n=n;
       this.l=l;
       ch= new Point[n]; 
    }
   //小于0,说明向量p0p1的极角大于p0p2的极角  
   public  double multiply(Point p1, Point p2, Point p0) { 
       return ((p1.getX() - p0.getX()) * (p2.getY() - p0.getY()) - 
    		   (p2.getX()- p0.getX()) * (p1.getY()- p0.getY())); 
   } 
   //求距离
   public  double distance(Point p1, Point p2) { 
       return (Math.sqrt((p1.getX() - p2.getX()) * (p1.getX() - p2.getX()) + (p1.getY() - p2.getY()) 
               * (p1.getY() - p2.getY()))); 
   } 
   public void answer(){
    double sum = 0; 
       for (int i = 0; i < len - 1; i++) { 
           sum += distance(ch[i], ch[i + 1]); 
       } 
       if (len > 1) { 
           sum += distance(ch[len - 1], ch[0]); 
       } 
       sum += 2 * l * Math.PI; 
       System.out.println(Math.round(sum)); 
   }
   public  int Graham_scan() { 
       int k = 0, top = 2; 
       Point tmp; 
       //找到最下且偏左的那个点   
       for (int i = 1; i < n; i++) 
           if ((p[i].getY() < p[k].getY()) 
                   || ((p[i].getY() == p[k].getY()) && (p[i].getX() < p[k].getX()))) 
               k = i; 
       //将这个点指定为pts[0],交换pts[0]与pts[k] 
       tmp = p[0]; 
       p[0] = p[k]; 
       p[k] = tmp; 
       //按极角从小到大,距离偏短进行排序   
       for (int i = 1; i < n - 1; i++) { 
           k = i; 
           for (int j = i + 1; j < n; j++) 
               if ((multiply(p[j], p[k], p[0]) > 0) 
                       || ((multiply(p[j], p[k], p[0]) == 0) && (distance( 
                               p[0], p[j]) < distance( 
                               p[0], p[k])))) 
                   k = j; //k保存极角最小的那个点,或者相同距离原点最近  
           tmp = p[i]; 
           p[i] = p[k]; 
           p[k] = tmp; 
       } 
       //前三个点先入栈  
       ch[0] = p[0]; 
       ch[1] = p[1]; 
       ch[2] = p[2]; 
        //判断与其余所有点的关系   
       for (int i = 3; i < n; i++) { 
            //不满足向左转的关系,栈顶元素出栈   
           while (top > 0 && multiply(p[i], ch[top], ch[top - 1]) >= 0) 
               top--; 
            //当前点与栈内所有点满足向左关系,因此入栈. 
           ch[++top] = p[i]; 
       } 
       len=top+1;
       return len; 
   } 
  
   /**
样例: 
Sample Input 
9 100 
200 400 
300 400 
300 300 
400 300 
400 400 
500 400 
500 200 
350 200 
200 200 

Sample Output 
1628 
	*/
   public static void main(String[] args)  { 
    
       Scanner in=new Scanner(System.in);
       int n = in.nextInt(); 
       int l = in.nextInt();
       int x, y; 
       Point[] p = new Point[n]; 
       for (int i = 0; i < n; i++) { 
           x = in.nextInt(); 
           y = in.nextInt();
           p[i] = new Point(x, y); 
       } 
      
       ConvexHullWithGraham ma=new ConvexHullWithGraham(p,n,l); 
       ma.Graham_scan(); 
       ma.answer();
   } 
} 
{% endhighlight %}

## 解五：Melkman算法

Melkman凸包算法继承Graham扫描法的主要思想，并更近一步地采用双端队列，动态地在队列两头进行增删操作，维护“凸性”。

基本思想是：首先跟Graham扫描法一样，纵坐标最小的点作为基点，得到其他各点对该点的夹角，存储在栈中并排序。取最初的三点组合成为一个三角形。从一个三角形开始，对接下来的从栈中取出的每一个点，判断该点是否在该三角形内部，不在则添加该点，使得三角形不断地扩大范围，多边形的边数也在变化。获得到的点，序号可以连成一个闭环。另外是否在该多边形内部的判定是使用的左转判定，即是根据从双向表的两端开始，如果该点在在两边的左边（逆时针方向），则该点在多边形的内部。

具体的步骤是：

1、初始化。跟Graham一样，取一点，得到角度，然后排序各点。

2、接着 向双向表中依次装入P2,P0,P1,P2. 注意：这是一个收尾相连的三角形，三点不共线。

3、设置 双向表的左右指针分别是bot、top。该范围内是，不断扩大范围的凸包点多边形。

4、反复i++，即是按照顺序，判断Pi是否在该多边形的内部，直到不在内部，即是d[t-1]、d[t-2]、Pi(逆时针方向)不做左转，或者d[bot+1]、d[bot+2]、Pi不做左转。

5、循环以下步骤直至d[t-1]、d[t-2]、Pi做左转：

    t--;	//退栈，判断是否在新的凸包内部。防止出现凹处将Pi从右压入d

6、循环以下步骤直至d[bot+1]、d[bot+2]、Pi做左转

    b++;	//退栈，判断是否在新的凸包内部。防止出现凹处将Pi从左压入d

注意：每次执行第5、6步时候，两端的都需要压入相同的数，使得凸包闭环。

	这个算法可以在各点有序的前提下，每次获知一个点，就可以将先前的凸包改造成新的凸包。

	因此，这是一个在线算法，有着其他算法无法比拟的优势。

![melkman.gif]({{site.baseurl}}/img/melkman.gif)

{% highlight ruby %}
package com.cyw.algorithms;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class ConvexHullWithMelkman {
	private Point[] pointArray;//坐标数组
	private final int N;//数据个数
	private int D[]; // 数组索引，双向表

	public ConvexHullWithMelkman(List<Point> pList) {
		this.pointArray = new Point[pList.size()];
		N = pList.size();
		int k = 0;
		for (Point p : pList) {
			pointArray[k++] = p;
		}
		D = new int[2 * N];
	}

	/**
	 * 求凸包点
	 * 
	 * @return 所求凸包点
	 */
	public Point[] getTubaoPoint() {
		// 获得最小的Y，作为P0点
		float minY = pointArray[0].getY();
		int j = 0;
		for (int i = 1; i < N; i++) {
			if (pointArray[i].getY() < minY) {
				minY = pointArray[i].getY();
				j = i;
			}
		}
		//交换内容
		swap(0, j);

		// 计算除第一顶点外的其余顶点到第一点的线段与x轴的夹角
		for (int i = 1; i < N; i++) {
			pointArray[i].setArCos(angle(i));
		}

		quickSort(1, N - 1); // 根据所得到的角度进行快速排序

		int bot = N - 1;
		int top = N;
		D[top++] = 0;
		D[top++] = 1;
		int i;

		for (i = 2; i < N; i++) {// 寻找第三个点 要保证3个点不共线！！==0 代表共线
			if (isLeft(pointArray[D[top - 2]], pointArray[D[top - 1]], pointArray[i]) != 0)
				break;
			D[top - 1] = i; // 共线就更换顶点
		}

		D[bot--] = i;
		D[top++] = i; // i是第三个点 不共线！！

//		最初的一次
		int t;
		if (isLeft(pointArray[D[N]], pointArray[D[N + 1]], pointArray[D[N + 2]]) < 0) {
			// 此时队列中有3个点，要保证3个点a,b,c是成逆时针的，不是就调换ab
			t = D[N];
			D[N] = D[N + 1];
			D[N + 1] = t;
		}

		//开始
		for (i++; i < N; i++) {
			// 如果成立就是i在凸包内，跳过 //top=n+3 bot=n-2
//			此时top、bot没有改变
			if (isLeft(pointArray[D[top - 2]], pointArray[D[top - 1]],pointArray[i]) > 0 &&
					isLeft(pointArray[D[bot + 1]], pointArray[D[bot + 2]],pointArray[i]) > 0) {
				continue;
			}
			
			//非左转 则退栈
			while (isLeft(pointArray[D[top - 2]], pointArray[D[top - 1]],
					pointArray[i]) <= 0) {
				top--;
			}
			D[top++] = i;
			
			//反向表非左转 则退栈
			while (isLeft(pointArray[D[bot + 1]], pointArray[D[bot + 2]],
					pointArray[i]) <= 0) {
				bot++;
			}
			D[bot--] = i;
		}

		// 凸包构造完成，D数组里bot+1至top-1内就是凸包的序列(头尾是同一点)
		Point[] resultPoints = new Point[top - bot - 1];
		int index = 0;
		for (i = bot + 1; i < top - 1; i++) {
			System.out.println(pointArray[D[i]].getX() + ","
					+ pointArray[D[i]].getY());
			resultPoints[index++] = pointArray[D[i]];
		}
		return resultPoints;
	}

	/**
	 * 判断ba相对ao是不是左转
	 * 
	 * @return 大于0则左转
	 */
	private float isLeft(Point o, Point a, Point b) {
		float aoX = a.getX() - o.getX();
		float aoY = a.getY() - o.getY();
		float baX = b.getX() - a.getX();
		float baY = b.getY() - a.getY();

		return aoX * baY - aoY * baX;
	}

	/**
	 * 实现数组交换
	 * 
	 * @param i
	 * @param j
	 */
	private void swap(int i, int j) {
		Point tempPoint = new Point();
		tempPoint.setX(pointArray[j].getX());
		tempPoint.setY(pointArray[j].getY());
		tempPoint.setArCos(pointArray[j].getArCos());

		pointArray[j].setX(pointArray[i].getX());
		pointArray[j].setY(pointArray[i].getY());
		pointArray[j].setArCos(pointArray[i].getArCos());

		pointArray[i].setX(tempPoint.getX());
		pointArray[i].setY(tempPoint.getY());
		pointArray[i].setArCos(tempPoint.getArCos());
	}

	/**
	 * 快速排序
	 * 
	 * @param top
	 * @param bot
	 */
	private void quickSort(int top, int bot) {
		int pos;
		if (top < bot) {
			pos = loc(top, bot);
			quickSort(top, pos - 1);
			quickSort(pos + 1, bot);
		}
	}

	/**
	 * 移动起点，左侧为小，右侧为大
	 * @param top
	 * @param bot
	 * @return 移动后的位置
	 */
	private int loc(int top, int bot) {
		double x = pointArray[top].getArCos();
		int j, k;
		j = top + 1;
		k = bot;
		while (true) {
//			
			while (j < bot && pointArray[j].getArCos() < x)
				j++;
//			
			while (k > top && pointArray[k].getArCos() > x)
				k--;
			if (j >= k)
				break;
			swap(j, k);
		}
		swap(top, k);
		return k;
	}

	/**
	 * 角度计算
	 * 
	 * @param i 指针
	 * @return
	 */
	private double angle(int i) {
		double j, k, m, h;
		j = pointArray[i].getX() - pointArray[0].getX();
		k = pointArray[i].getY() - pointArray[0].getY();
		m = Math.sqrt(j * j + k * k); // 得到顶点i 到第一顶点的线段长度
		if (k < 0)
			j = (-1) * Math.abs(j);
		h = Math.acos(j / m); // 得到该线段与x轴的角度
		return h;
	}

	/**data.txt:
0	1
0	4
1	3
2	2
3	1
3	4
	 */
	public static void main(String args[]) {
		// File file = new File("G:/yl.txt");
		File file = new File("D:/data.txt");
		BufferedReader br = null;
		try {
			br = new BufferedReader(new FileReader(file));
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
		List<Point> pointList = new ArrayList<Point>();
		String str = null;
		try {
			str = br.readLine();
		} catch (IOException e) {
			e.printStackTrace();
		}
		while (str != null) {
			String[] s = str.split("\\t", 2);
			float x = Float.parseFloat(s[0].trim());
			float y = Float.parseFloat(s[1].trim());
			Point p = new Point();
			p.setX(x);
			p.setY(y);
			// System.out.println("文件数据：" + x + ", " + y);
			pointList.add(p);
			try {
				str = br.readLine();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		System.out.println("数据个数：" + pointList.size());

		ConvexHullWithMelkman m = new ConvexHullWithMelkman(pointList);
		m.getTubaoPoint();
	}
}
{% endhighlight %}

以上几种算法的参考代码，我已经UP到了我的GItHub上面了，分别对应的是：

[![javaConvexHull.png]({{site.baseurl}}/img/javaConvexHull.png)](https://github.com/cyw3/CywLeetCode/tree/master/LeetCode/src/com/cyw/algorithms)

可以点击图片，到GitHub上面按需下载。
