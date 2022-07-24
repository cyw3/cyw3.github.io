---
title: 投资组合的收益率走势图
author: yalechen
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---

## 前言
各位看官好，我是小陈，今天又来献丑了。作为金融小白，小陈我最近一直在恶补投资方面的知识。而前些天，正好涌哥叫我用况客API实现一个投资组合的收益率走势图，感觉自己一身代码神功又有用武之地了。

所谓投资组合是说，投资持有各种金融产品的组合，需要分段分批分散投入，目的是分散风险。当然，投资人都是希望自己投的股票能够不断的涨，把钱花在刀刃上，以最少的钱套取最大的利益，但是这是不现实的，毕竟股市，尤其是中国股市（大家都懂的），是变化莫测的。所以，如何尽可能的减少风险，利益最大化，成为了关键所在。而接下来我将说到的“收益率走势图”，将会是一个不错的工具，让我们趋利避害。



## 目录
1.投资组合以及收益率走势图的介绍

2.收益率走势图效果图展示

3.收益率走势图的代码实现以及优化

## 1.投资组合以及收益率走势图的介绍

美国经济学家马考维茨(Markowitz)曾这样说过：“若干种证券组成的投资组合，其收益是这些证券收益的加权平均数，但是其风险不是这些证券风险的加权平均风险，投资组合能降低非系统性风险。”

马考维茨最著名的作品便是《投资组合理论》，他还因此获得了诺贝尔经济学奖。当然，本文的重点不是说要探讨马考维茨的“投资组合理论”云云（其实我也不懂）。但是这也足够说明了，合理的投资组合，能够让投资者获取更大的利益。投资者持有多种金融产品，不同的产品可能具有不同的市场地位和价值优势，需要综合评价企业的价值能力，进行投资组合分析。至于收益率走势图，显然是我们进行投资组合分析时的一大工具了。

收益率是指投资的回报率，一般以年度百分比表达，根据当时市场价格、面值、息票利率以及距离到期日时间计算。对公司而言，收益率指净利润占使用的平均资本的百分比。而通过收益率走势图，我们可以实时地知道，我们所持股票的自买入以来的收益情况，以及可以大体预测股票的大致走向，如果辅以好的算法的话，甚至可以如同化身“预言家”，对未来股票走势了如指掌。



## 2.收益率走势图效果图展示
哎呀，我也不说什么太虚的了。直接开干。以下便是我们代码实现的效果图了。怎么样？很棒吧！觉得羡慕吧！羡慕就继续往下看，走起。[yieldChart](https://i.qutke.com/apps/569f4501310a3c46406493bd)
<iframe style="width:100%;height:600px;border:0;padding:0;margin:0;" src="https://console.qutke.com/diagram/info/569e7b5b310a3c4640649396?config={ %22color%22:%22%23ffffff%22}"></iframe>


## 3.收益率走势图的代码实现以及优化
1.系统环境

跟上一篇文一样，我们需要准备好开发所需要的环境。就像学习一样，得要有个好的学习氛围嘛。

```ruby
    win7系统
    RStudio编辑器
    qutke包
```

需要提前下载况客R语言Api，即qutke，[GitHub链接](https://github.com/qutke/qutke)
可以安装注册git，通过git命令来下载相应的R包。

```ruby
git clone https://github.com/qutke/qutke.git 
```

也可以通过RStudio命令来下载：

```ruby
	library(devtools)
	install_github('qutke/qutke')
```

2.初始化，弹药准备

```ruby
#安装相应的需要使用的R包
library('lubridate')
library(qutke)
key<-'ff5ed58edf645c6581e8148db1130dc310fbab5fdccc4b2a9ea0be30f4128ace'
init(key)
```


3.参数选定

在制作走势图之前，我们得知道，是什么样的组合，什么时候买入的。这里，我参考了雪球里的一个热门组合“丁丁丁涨不停ZH078564”，分别是科大讯飞“002230.SZ"，以及登云股份“002715.SZ”。日期就选在“2015-10-01”吧。

```ruby
qtid <- c('002230.SZ','002715.SZ')
date <- '2015-10-01'
```


4.获取股票基本信息

依次获取这两支股票的从买入日期至今的每天的收盘价。

```ruby
#股票基本信息
md <- getMD(data='keyMap',qtid=qtid,key=key)
#股票日间行情(前复权)
dailyQuote <- getDailyQuote(data='mktFwdDaily',qtid=qtid,startdate=date,enddate=Sys.Date(),key=key)
#当日收盘价(前复权)
stock1 <-dailyQuote[which(dailyQuote$qtid==qtid[1]),]
stock2 <-dailyQuote[which(dailyQuote$qtid==qtid[2]),]
#x轴，作为时间轴。取买入日期至今。
tradingDay <- getDate(data='tradingDay',startdate=stock1$date[1],enddate=Sys.Date(),key=key)
```

5.数据处理，以备后面生成dataframe表格。此时，fwdAdjClose是每只股票在每一个交易日的收盘价。

```ruby
#归一化
i <- 1
length <- length(tradingDay)
fwdAdjClose1 <- c()
fwdAdjClose2 <- c()
while(i<=length){
  fwdAdjClose1 <- c(fwdAdjClose1,stock1[i,]$fwdAdjClose)
  if(is.null(fwdAdjClose1[i])||is.na(fwdAdjClose1[i])) 
    fwdAdjClose1[i] <- fwdAdjClose1[i-1]
  fwdAdjClose2 <- c(fwdAdjClose2,stock2[i,]$fwdAdjClose)
  if(is.null(fwdAdjClose2[i])||is.na(fwdAdjClose2[i])) 
    fwdAdjClose2[i] <- fwdAdjClose2[i-1]
  i=i+1
}
```

6.收益率计算

收益率计算公式是：

某只股票收益率 = （当天的股价-买入价）/ 买入价 *100

```ruby
#计算
fwdAdjClose1 <- (fwdAdjClose1-fwdAdjClose1[1])/fwdAdjClose1[1]*100
fwdAdjClose2 <- (fwdAdjClose2-fwdAdjClose2[1])/fwdAdjClose2[1]*100
```


7.生成dataframe表格，并将数据发送到况客投研中心，以便后期处理

```ruby
#形成dataframe
yieldChart <- data.frame('date'=tradingDay,'2'=fwdAdjClose1,'3'=fwdAdjClose2)
names(yieldChart)<-c('日期',md[1,]$ChiAbbr,md[2,]$ChiAbbr)
postData(yieldChart,name='yieldChart',key=key)
```

8.代码优化

当然，我们不能就这满足了。我们需要在对代码略微修改，使之只需要我们输入交易日期、股票组合，就可以自动生成需要的收益率走势图。

```ruby
getYieldChart<-function(date,qtid=c(),key){
  if(length(qtid)<=0){
    stop("Qtid is more than one.")
  }
  #股票基本信息
  md <- getMD(data='keyMap',qtid=qtid,key=key)
  #股票日间行情(前复权)
  dailyQuote <- getDailyQuote(data='mktFwdDaily',qtid=qtid,startdate=date,enddate=Sys.Date(),key=key)
  #当日收盘价(前复权)
  stock <- list()
  qtidlength <- length(qtid)
  i <- 1
  while(i<=qtidlength){
    ChiAbbr <- md[i,]$ChiAbbr
    qtidSt <- md[i,]$qtid
    stock[[ChiAbbr]] <- dailyQuote[which(dailyQuote$qtid==qtidSt),]
    i <- i+1
  }
  #x轴，作为时间轴。取买入日期至今。
  tradingDay <- getDate(data='tradingDay',startdate=stock[[1]]$date[1],enddate=Sys.Date(),key=key)
  fwdAdjClose <- list()
  fwdAdjClose[['date']] <- tradingDay
  #归一化
  i <- 1
  length <- length(tradingDay)
  while(i<=qtidlength){
    ChiAbbr <- md[i,]$ChiAbbr
    j <- 1
    while(j<=length){
      fwdAdjClose[[ChiAbbr]] <- c(fwdAdjClose[[ChiAbbr]],stock[[ChiAbbr]][j,]$fwdAdjClose)
      if(is.null(fwdAdjClose[[ChiAbbr]][j])||is.na(fwdAdjClose[[ChiAbbr]][j])) 
        fwdAdjClose[[ChiAbbr]][j] <- fwdAdjClose[[ChiAbbr]][j-1]
      j <- j+1
    }
    #计算
    fwdAdjClose[[ChiAbbr]] <- (fwdAdjClose[[ChiAbbr]]-fwdAdjClose[[ChiAbbr]][1])/fwdAdjClose[[ChiAbbr]][1]*100
    i=i+1
  }
  #形成dataframe
  yieldChart <-data.frame(fwdAdjClose)
  postData(yieldChart,name='yieldChart',key=key)
} 
```


这样，就算完成了。（满意脸）