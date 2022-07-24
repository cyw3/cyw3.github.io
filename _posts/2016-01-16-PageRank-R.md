---
title: R语言实现PageRank
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---

## 前言
当今，一个算法能够支撑起一个商业帝国，这句话还真不是吹的，你看Google就知道了。Google当年就是靠PageRank雄起的，将网页搜索业务做得风生水起。

而今天，我们就来看看如何使用R语言来实现PageRank算法的。

## 目录

1.PageRank的原理

2.PageRank实现步骤

3.R代码实现

## 1.PageRank的原理

其实说白了，PageRank原理就是：

> PageRank让链接来”投票”

一个页面的“得票数”由所有链向它的页面的重要性来决定。到一个页面的超链接相当于对该页投一票。

一个页面的PageRank是由所有链向它的页面（“链入页面”）的重要性经过递归算法得到的。

一个有较多链入的页面会有较高的等级，相反如果一个页面没有任何链入页面，那么它没有等级。

PageRank的计算基于以下两个`基本假设`：

数量假设：如果一个页面节点接收到的其他网页指向的入链数量越多，那么这个页面越重要。

质量假设：指向页面A的入链质量不同，质量高的页面会通过链接向其他页面传递更多的权重。所以越是质量高的页面指向页面A，则页面A越重要。

要提高PageRank有3个`要点`：

* 反向链接数
* 反向链接是否来自PageRank较高的页面
* 反向链接源页面的链接数

基于以上特点，我们再来看看PageRank的基本公式吧。

![pr]({{site.baseurl}}/img/pr.png)


## 2.PageRank实现步骤

- 构造邻接矩阵
  
    列：源页面

    行：目标页面

- 概率矩阵(转移矩阵)

  计算公式在这儿体现跟连接数有关的

- 求开率矩阵的特征值

  一般使用的是迭代的算法。当然也可以使用R中的API直接给出特征值。
  而是用迭代的话，在一轮更新页面PageRank得分的计算中，每个页面将其当前的PageRank值平均分配到本页面包含的出链上，这样每个链接即获得了相应的权值。而每个页面将所有指向本页面的入链所传入的权值求和，即可得到新的PageRank得分。当每个页面都获得了更新后的PageRank值，就完成了一轮PageRank计算。


## 3.R代码实现

分别用下面3种方式实现PageRank:
  
- 未考虑阻尼系统的情况
- 包括考虑阻尼系统的情况
- 直接用R的特征值计算函数

但是，在这之前，我们还有新建一个page.csv文件，方便我们后面的实现说明。

{% highlight ruby %}
1,2
1,3
1,4
2,3
2,4
3,4
4,2
{% endhighlight %}

### 1)未考虑阻尼系统的情况

以下是函数定义。

{% highlight ruby %}
#构建邻接矩阵
adjacencyMatrix<-function(pages){
  n<-max(apply(pages,2,max))
  A <- matrix(0,n,n)
  for(i in 1:nrow(pages)) A[pages[i,]$dist,pages[i,]$src]<-1
  A
}

#变换概率矩阵
probabilityMatrix<-function(G){
  # 按列相加
  cs <- colSums(G)
  cs[cs==0] <- 1
  n <- nrow(G)
  A <- matrix(0,nrow(G),ncol(G))
  for (i in 1:n) A[i,] <- A[i,] + G[i,]/cs
  A
}

#递归计算矩阵特征值。跟权重有关的
eigenMatrix<-function(G,iter=100){
  # iter<-10
  n<-nrow(G)
  x <- rep(1,n)
  # 迭代,x为每一轮计算的权重，也是pR值。
  for (i in 1:iter) x <- G %*% x
  x/sum(x)
}

pages<-read.table(file="page.csv",header=FALSE,sep=",")
names(pages)<-c("src","dist");pages
{% endhighlight %}

以下便开始执行程序。

{% highlight ruby %}
> A<-adjacencyMatrix(pages);A

            [,1] [,2] [,3] [,4]
[1,]    0    0    0    0
[2,]    1    0    0    1
[3,]    1    1    0    0
[4,]    1    1    1    0

> G<-probabilityMatrix(A);G

      [,1] [,2] [,3] [,4]
[1,] 0.0000000  0.0    0    0
[2,] 0.3333333  0.0    0    1
[3,] 0.3333333  0.5    0    0
[4,] 0.3333333  0.5    1    0

> q<-eigenMatrix(G,10);q

    [,1]
[1,] 0.0000000
[2,] 0.4036458
[3,] 0.1979167
[4,] 0.3984375
{% endhighlight %}

### 2)包括考虑阻尼系统的情况

dProbabilityMatrix是对于adjacencyMatrix函数的重构。

{% highlight ruby %}
#变换概率矩阵,考虑d的情况
dProbabilityMatrix<-function(G,d=0.85){
  cs <- colSums(G)
  cs[cs==0] <- 1
  n <- nrow(G)
  # (1-d)/n
  delta <- (1-d)/n
  A <- matrix(delta,nrow(G),ncol(G))
  for (i in 1:n) A[i,] <- A[i,] + d*G[i,]/cs
  A
}

> pages<-read.table(file="page.csv",header=FALSE,sep=",")
> names(pages)<-c("src","dist");pages

  src dist
1   1    2
2   1    3
3   1    4
4   2    3
5   2    4
6   3    4
7   4    2

> A<-adjacencyMatrix(pages);A

      [,1] [,2] [,3] [,4]
[1,]    0    0    0    0
[2,]    1    0    0    1
[3,]    1    1    0    0
[4,]    1    1    1    0

> G<-dProbabilityMatrix(A);G
        [,1]   [,2]   [,3]   [,4]
[1,] 0.0375000 0.0375 0.0375 0.0375
[2,] 0.3208333 0.0375 0.0375 0.8875
[3,] 0.3208333 0.4625 0.0375 0.0375
[4,] 0.3208333 0.4625 0.8875 0.0375

> q<-eigenMatrix(G,100);q
      [,1]
[1,] 0.0375000
[2,] 0.3738930
[3,] 0.2063759
[4,] 0.3822311
{% endhighlight %}

增加阻尼系数后，ID=1的页面，就有值了PR(1)=(1-d)/n=(1-0.85)/4=0.0375，即无外链页面的最小值。


### 3)直接用R的特征值计算函数

增加的函数：calcEigenMatrix
用于直接计算矩阵特征值，可以有效地减少的循环的操作，提高程序运行效率。

{% highlight ruby %}
#直接计算矩阵特征值.即特解
calcEigenMatrix<-function(G){
  x <- Re(eigen(G)$vectors[,1])
  x/sum(x)
}

> pages<-read.table(file="page.csv",header=FALSE,sep=",")
> names(pages)<-c("src","dist");pages
    src dist
1   1    2
2   1    3
3   1    4
4   2    3
5   2    4
6   3    4
7   4    2

> A<-adjacencyMatrix(pages);A
      [,1] [,2] [,3] [,4]
[1,]    0    0    0    0
[2,]    1    0    0    1
[3,]    1    1    0    0
[4,]    1    1    1    0

> G<-dProbabilityMatrix(A);G
        [,1]   [,2]   [,3]   [,4]
[1,] 0.0375000 0.0375 0.0375 0.0375
[2,] 0.3208333 0.0375 0.0375 0.8875
[3,] 0.3208333 0.4625 0.0375 0.0375
[4,] 0.3208333 0.4625 0.8875 0.0375

> q<-calcEigenMatrix(G);q
[1] 0.0375000 0.3732476 0.2067552 0.3824972
{% endhighlight %}

