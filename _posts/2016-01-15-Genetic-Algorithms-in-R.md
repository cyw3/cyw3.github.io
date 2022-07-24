---
title: 遗传算法的R语言实现
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---

## 前言
遗传算法的操作使用适者生存的原则，在潜在的种群中逐次产生一个近似最优解的方案，在每一代中，根据个体在问题域中的适应度值和从自然遗传学中借鉴来的再造方法进行个体选择，产生一个新的近似解。这个过程会导致种群中个体的进化，得到的新个体比原来个体更能适应环境，就像自然界中的改造一样。

## 目录

1. 遗传算法原理

2. R语言实现

## 1.遗传算法原理

在遗传算法里，优化问题的解是被称为个体，它表示为一个变量序列，叫做染色体或者基因串。
编码：染色体一般被表达为简单的字符串或数字串，也有其他表示法，这一过程称为编码。
基本的过程是：

1)首先要创建种群，算法随机生成一定数量的个体，有时候也可以人工干预这个过程进行，
以提高初始种群的质量。在每一代中，每一个个体都被评价，并通过计算适应度函数得到一个适应度数值。种群中的个体被按照适应度排序，适应度高的在前面。（适应度函数）

2)产生下一代个体的种群，通过选择过程和繁殖过程完成。

选择过程，是根据新个体的适应度进行的，但同时并不意味着完全的以适应度高低作为导向，因为单纯选择适应度高的个体将可能导致算法快速收敛到局部最优解而非全局最优解，我们称之为早熟。作为折中，遗传算法依据原则：适应度越高，被选择的机会越高，而适应度低的，被选择的机会就低。初始的数据可以通过这样的选择过程组成一个相对优化的群体。

繁殖过程，表示被选择的个体进入交配过程，包括交配(crossover)和突变(mutation，交配对应算法中的交叉操作。一般的遗传算法都有一个交配概率，范围一般是0.6~1，这个交配概率反映两个被选中的个体进行交配的概率。例如，交配概率为0.8，则80%的“夫妻”个体会生育后代。每两个个体通过交配产生两个新个体，代替原来的“老”个体，而不交配的个体则保持不变。交配过程，父母的染色体相互交換，从而产生两个新的染色体，第一个个体前半段是父亲的染色体，后半段是母亲的，第二个个体则正好相反。不过这里指的半段並不是真正的一半，这个位置叫做交配点，也是随机产生的，可以是染色体的任意位置。

突变过程，表示通过突变产生新的下一代个体。一般遗传算法都有一个固定的突变常数，又称为变异概率，通常是0.1或者更小，这代表变异发生的概率。根据这个概率，新个体的染色体随机的突变，通常就是改变染色体的一个字节（0变到1，或者1变到0）。

遗传算法实现将不断的重复这个过程：每个个体被评价，计算出适应度，两个个体交配，然后突变，产生下一代，直到终止条件满足为止。一般终止条件有以下几种：

进化次数限制
* 计算耗费的资源限制，如计算时间、计算占用的CPU，内存等
* 个体已经满足最优值的条件，即最优值已经找到
* 当适应度已经达到饱和，继续进化不会产生适应度更好的个体
* 人为干预

过程：

1. 创建初始种群

2. 循环：产生下一代

3. 评价种群中的个体适应度

4. 定义选择的适应度函数

5. 改变该种群（交配和变异）

6. 返回第二步

7. 满足终止条件结束


## 2.R语言实现

一般在进行遗传算法的时候，需要考虑的参数有：

* 种群规模(Population)，即种群中染色体个体的数目。
* 染色体的基因个数(Size)，即变量的数目。
* 交配概率(Crossover)，用于控制交叉计算的使用频率。交叉操作可以加快收敛，使解达到最有希望的最优解区域，因此一般取较大的交叉概率，但交叉概率太高也可能导致过早收敛。
* 变异概率(Mutation)，用于控制变异计算的使用频率，决定了遗传算法的局部搜索能力。
* 中止条件(Termination)，结束的标志。

以下是常见的R遗传算法包：

* mcga包，多变量的遗传算法，用于求解多维函数的最小值。
* genalg包，多变量的遗传算法，用于求解多维函数的最小值。
* rgenoud包，复杂的遗传算法，将遗传算法和衍生的拟牛顿算法结合起来，可以求解复杂函数的最优化化问题。
* gafit包，利用遗传算法求解一维函数的最小值。不支持R 3.1.1的版本。
* GALGO包，利用遗传算法求解多维函数的最优化解。不支持R 3.1.1的版本。

那么我们今天就只针对mcga包，还有genalg包进行讲解一下吧。

1)mcga

实值优化问题。使用的变量值表示基因序列，而不是字节码，因此不需要编解码的处理。
mcga实现了遗传算法的交配和突变的操作，并且可以进行大范围和高精度的搜索空间的计算，算法的主要缺点是使用了256位的一元字母表。

{% highlight ruby %}
install.packages("mcga")
library(mcga)

#查看mcga的定义
mcga
{% endhighlight %}

这是mcga的方法定义。

{% highlight ruby %}
    function (popsize, chsize, crossprob = 1, mutateprob = 0.01, 
             elitism = 1, minval, maxval, maxiter = 10, evalFunc) 
{% endhighlight %}

参数说明：

* popsize，个体数量，即染色体数目
* chsize，基因数量，限参数的数量
* crossprob，交配概率，默认为1.0
* mutateprob，突变概率，默认为0.01
* maxiter，繁殖次数，即循环次数，默认为10
* evalFunc，适应度函数，用于给个体进行评价
* elitism，精英数量，直接复制到下一代的染色体数目，默认为1
* minval，随机生成初始种群的下边界值
* maxval，随机生成初始种群的上边界值

例子：fx=(x1-5)^2 + (x2-55)^2 +(x3-555)^2 +(x4-5555)^2 +(x5-55555)^2

{% highlight ruby %}
# 定义适应度函数 
f <- function(x){
  #总括号
  return ((x[1]-5)^2+(x[2]-55)^2+(x[3]-555)^2+(x[4]-5555)^2+(x[5]-55555)^2)
}

# 运行遗传算法
m <- mcga(   popsize=200, 
             chsize=5, 
             minval=0.0, 
             maxval=999999, 
             maxiter=2500, 
             crossprob=1.0, 
             mutateprob=0.01, 
             evalFunc=f)

# 最优化的个体结果
print(m$population[1,])

# 执行时间
m$costs[1]    
{% endhighlight %}

同样，当题目改为fx=(x1-5)^2 + (x2-55)^2 +(x3-555)^2 +(x4-5555)^2 +(x5-55555)^2时候：
    
{% highlight ruby %}
#这是适应度函数
f<-function(x){
  return ((x[1]-7)^2 + (x[2]-77)^2 +(x[3]-777)^2 +(x[4]-7777)^2 +(x[5]-77777)^2)
}

m<-mcga(popsize=200, 
  chsize=5, 
  minval=0.0, 
  maxval=999999999.9, 
  maxiter=2500, 
  crossprob=1.0, 
  mutateprob=0.01, 
  evalFunc=f)
cat("Best chromosome:\n")
print(m$population[1,])
cat("Cost: ",m$costs[1],"\n")
{% endhighlight %}

2)genalg包

实现了遗传算法，还提供了遗传算法的数据可视化，给用户更直观的角度理解算法。
默认图显示的最小和平均评价值，表示遗传算法的计算进度。
直方图显出了基因选择的频率，即基因在当前个体中被选择的次数。
参数图表示评价函数和变量值，非常方便地看到评价函数和变量值的相关关系。

{% highlight ruby %}
    install.packages("genalg")
    library(genalg)
{% endhighlight %}

这是genalg函数定义：

{% highlight ruby %}
    rbga(stringMin=c(), stringMax=c(),
         suggestions=NULL,
         popSize=200, iters=100,
         mutationChance=NA,
         elitism=NA,
         monitorFunc=NULL, evalFunc=NULL,
         showSettings=FALSE, verbose=FALSE)
{% endhighlight %}

* stringMin，设置每个基因的最小值
* stringMax，设置每个基因的最大值
* suggestions，建议染色体的可选列表
* popSize，个体数量，即染色体数目，默认为200
* iters，迭代次数，默认为100
* mutationChance，突变机会，默认为1/(size+1)，它影响收敛速度和搜索空间的探测，低机率导致更快收敛，高机率增加了搜索空间的跨度。
* elitism，精英数量，默认为20%，直接复制到下一代的染色体数目
* monitorFunc，监控函数，每产生一代后运行
* evalFunc，适应度函数，用于给个体进行评价
* showSettings，打印设置，默认为false
* verbose，打印算法运行日志，默认为false


例如：设fx=abs(x1-sqrt(exp(1)))+abs(x2-log(pi))，计算fx的最小值，其中x1,x2为2个不同的变量。

{% highlight ruby %}
# 定义适应度函数
f<-function(x){
  return(abs(x[1]-sqrt(exp(1)))+abs(x[2]-log(pi)))
} 

# 定义监控函数
monitor <- function(obj){
  xlim = c(obj$stringMin[1], obj$stringMax[1]);
  ylim = c(obj$stringMin[2], obj$stringMax[2]);
  plot(obj$population, xlim=xlim, ylim=ylim, 
       xlab="pi", ylab="sqrt(50)");
}  

# 运行遗传算法 在命令行中运行
m2 = rbga(c(1,1),
           c(3,3),
           popSize=100,
           iters=1000,
           evalFunc=f,
           mutationChance=0.01,
           verbose=TRUE,
           monitorFunc=monitor
           )

#计算结果
m2$population[1,]
{% endhighlight %}

默认图输出，用于描述遗传过程的进展，X轴为迭代次数，Y轴评价值，评价值越接近于0越好。
在1000迭代1000次后，基本找到了精确的结果。

{% highlight ruby %}
    plot(m2)
{% endhighlight %}

直方图输出，用于描述对染色体的基因选择频率，即一个基因在染色体中的当前人口被选择的次数。
当x1在1.65区域时，被选择超过80次；当x2在1.146区域时，被选择超过了80次。通过直方图，我们可以理解为更优秀的基因被留给了后代。

{% highlight ruby %}
    plot(m2,type='hist')
{% endhighlight %}

参数图输出，用于描述评价函数和变量的值的相关关系。对于x1，评价值越小，变量值越准确，但相关关系不明显。对于x2，看不出相关关系。

{% highlight ruby %}
    plot(m2,type='vars')
{% endhighlight %}

以下是genalg包官方给出的例子：

{% highlight ruby %}
# optimize two values to match pi and sqrt(50)
evaluate <- function(string=c()) {
  returnVal = NA;
  if (length(string) == 2) {
    returnVal = abs(string[1]-pi) + abs(string[2]-sqrt(50));
  } else {
    stop("Expecting a chromosome of length 2!");
  }
  returnVal
}

monitor <- function(obj) {
  # plot the population
  xlim = c(obj$stringMin[1], obj$stringMax[1]);
  ylim = c(obj$stringMin[2], obj$stringMax[2]);
  plot(obj$population, xlim=xlim, ylim=ylim, 
       xlab="pi", ylab="sqrt(50)");
}

rbga.results = rbga(c(1, 1), c(5, 10), monitorFunc=monitor, 
                    evalFunc=evaluate, verbose=TRUE, mutationChance=0.01)

plot(rbga.results)
plot(rbga.results, type="hist")
plot(rbga.results, type="vars")
{% endhighlight %}
