---
title: 申万行业分类日间表格
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---

## 前言
申银万国证券股份有限公司（简称：申银万国），是国内最早的一家股份制证券公司，也是目前国内规模最大、经营业务最齐全、营业网点分布最广泛的综合类证券公司之一。为能够实时掌握申银万国各行业的股市数据，包括当天收盘价、成交量、涨跌幅等，以应对变化多端的市场，采取正确的举措，收获最大的利益，所以一个能够实时生成申万行业分类日间表格是十分需要的。接下来，我们便使用况客Api实现这个的表格。

## 目录
1、申万行业分类日间表格的介绍

2、使用况客API实现的具体思路

## 1.申万行业分类日间表格的介绍

在开始动手之前，我们先看看我们要制作的成品是什么样子的吧。如下图：

<iframe style="width:100%;height:600px;border:0;padding:0;margin:0;" src="https://console.qutke.com/diagram/info/569767d1310a3c464064932f?config={ %22color%22:%22%23ffffff%22}"></iframe>

上图，是搜索的申万行业一级行业中各个个股当天的基本数据，包括了当天收盘价、成交量、涨跌幅、周涨跌幅、月涨跌幅、年涨跌幅等等。（注：此处搜索的申万行业一级行业是“商业贸易”，即SW1 <- '商业贸易'）当然，你也可以根据需要选定表格的内容。

从表格，我们可以了解到我们关注的个股的情况，看他目前是张还是跌，涨跌幅多大，通过比较周涨跌幅、月涨跌幅、年涨跌幅来判断该个股是否值得投资，以更好的做出正确的判断、举措。

## 2.使用况客API实现表格的具体思路
1、系统环境

{% highlight ruby %}
		win7
		RStudio
		qutke
{% endhighlight %}

需要提前下载况客R语言Api，即qutke，GitHub链接是：https://github.com/qutke/qutke
可以安装注册git，通过git命令来下载相应的R包。

{% highlight ruby %}
git clone https://github.com/qutke/qutke.git 
{% endhighlight %}

也可以通过RStudio命令来下载：

{% highlight ruby %}
library(devtools)
install_github('qutke/qutke')
{% endhighlight %}

2、咱们需要初始化，为正式编写代码做个准备吧，就像木匠一样，先把原材料以及工具准备好，才干活是不。其中，我们需要安装需要使用的R包，这里我们要用得到的有与date日期相关的lubridate包，以及我们刚下载的qutke。接着，通过数据库key，我们进行初始化，并与数据库连接上。

{% highlight ruby %}
#安装相应的需要使用的R包
library('lubridate')
library(qutke)
key<-'ff5ed58edf645c6581e8148db1130dc310fbab5fdccc4b2a9ea0be30f4128ace'
init(key)
{% endhighlight %}

3、定义变量，取得需要的参数

{% highlight ruby %}
date <- Sys.Date()
#获取交易日期，虽然在init(key)函数里面就已经取得了tradingDay，并且保存在e$TRADINGDAY变量中。
#不过这里为了介绍getDate()函数。于是就写明了吧。
lastYearDate <- as.Date(paste(year(date)-1,month(date),day(date),sep='-'))
tradingDay <- getDate(data='tradingDay',startdate=lastYearDate,enddate=date,key=key)
length <- length(tradingDay)
date <- tradingDay[length]
#进行筛选,判断其SW1,此时只对‘商业贸易’这一行业进行筛选
sw1 <- '商业贸易'
#通过getIndustry()函数，取得SW1，即‘商业贸易’这一行业相对应的个股
industry <- getIndustry(data='industryType',date=date,SW1=sw1,key=key)
#符合条件的个股的股票代码
qtid <- industry$qtid

#获得今年的第一个交易日日期
FirstDay <- as.Date(paste(year(date)-1,'1','1',sep='-'))
FirstDay <- (getDate(data='tradingDay',startdate=FirstDay,enddate=date,key=key))[1]
{% endhighlight %}

4、依照股票代码qtid来筛选获取数据库中这些个股的数据，并进行计算。此处，我们可以直接获得当天的收盘价、成交量，但是涨跌幅是需要进行计算的。
涨跌幅的计算公式是：(当前最新成交价（或收盘价）-开盘参考价)÷开盘参考价×100%
一般情况： 开盘参考价=前一交易日收盘价
除权息日： 开盘参考价=除权后的参考价
我们参考以上涨跌幅的计算公式，可以计算当天的涨跌幅，至于周涨跌幅，则是要把开盘参考价改为5个交易日前的收盘价即可，至于月涨跌幅、年涨跌幅、年初至今涨跌幅，亦是一个道理。

{% highlight ruby %}
#选用不复权
#通过getDailyQuote()方法获取股票日间行情。
mktDaily <- getDailyQuote(data='mktDaily',qtid = qtid,startdate=date,enddate=date,key=key)
qtid <- mktDaily$qtid
#个股证券名称
SecuAbbr <- (getIndustry(data='industryType',qtid = qtid,date=date,SW1=sw1,key=key))$SecuAbbr
#当日收盘价
close<- mktDaily$close
#当日交易量
volume <- mktDaily$volume
#上一交易日的收盘价
prevClose <- mktDaily$prevClose
#当日涨跌幅
quoteChangeDaily <- (close-prevClose)/prevClose
#获取近期的数据，分别为近一周、近一月、近一年、年初
mktWeek <- getDailyQuote(data='mktDaily',qtid = qtid,startdate=tradingDay[length-5],enddate=tradingDay[length-5],key=key)
mktMonth <- getDailyQuote(data='mktDaily',qtid = qtid,startdate=tradingDay[length-20],enddate=tradingDay[length-20],key=key)
mktYear <- getDailyQuote(data='mktDaily',qtid = qtid,startdate=tradingDay[1],enddate=tradingDay[1],key=key)
mktFYear <- getDailyQuote(data='mktDaily',qtid = qtid,startdate=FirstDay,enddate=FirstDay,key=key)
#通过公式计算涨跌幅
#近一周的涨跌幅
quoteChangeWeek <- (close-mktWeek$close)/mktWeek$close
#近一月的涨跌幅
quoteChangeMonth <- (close-mktMonth$close)/mktMonth$close
#近一年的涨跌幅
quoteChangeYear <- (close-mktYear$close)/mktYear$close
#年初至今的涨跌幅
quoteChangeFYear <- (close-mktFYear$close)/mktFYear$close
{% endhighlight %}

5、生成表格data.frame

{% highlight ruby %}
stock1 <- data.frame('1'=qtid,'2'=SecuAbbr,'3'=paste(year(date),month(date),day(date),sep='-'),'4'=close,
                     '5'=quoteChangeDaily,'6'=volume,'7'=quoteChangeWeek,
                     '8'=quoteChangeMonth,'9'=quoteChangeYear,
                     '10'=quoteChangeFYear,'11'=sw1)
#修改data.frame的列名
names(stock1)<-c('代码','名称','日期','收盘价','涨跌幅(%)','成交量(万元)','周涨跌幅(%)','月涨跌幅(%)','年涨跌幅(%)','年初至今涨跌幅（%）','SW1')
#发送到况客的数据服务器中，服务器将保存你生成的dataframe，并可以在况客投研平台看到可视化界面
postData(stock1,name='stock1',key=key)
{% endhighlight %}

6、登录况客投研平台，在 创建图表 选项中对生成的图表进行修改，并增加涨跌提示、搜索、排序等功能，然后可以将表格嵌入网站、博客之中，进行展示。

























