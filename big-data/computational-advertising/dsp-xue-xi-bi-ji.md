# DSP 学习笔记

## 什么是DSP(Demand Side Platform)

DSP不是从网络媒体那里包买广告位，也不是采用CPD(Cost Per Day)的方式获得广告位；而是从广告交易平台(AdExchange)来通过实时竞价的方式获得对广告进行曝光的机会，DSP通过广告交易平台对每个曝光单独购买，即采用CPM(Cost Per Impression)的方式获得广告位。

* 广告商或广告公司的，自动化购买广告的软件。
* 管理同DMP 或 ADX 市场交互的接口

## DSP 如何工作

* 代替广告网络的功能（ad network)
* 连接广告商和广告发布者

![工作原理图](https://revx.io/blog/wp-content/uploads/2020/05/How-Demand-Side-Platform-Works.png)

## 关键特性

* 活动管理 Campaign Management
* 目标人群选择 Audience Targeting Options
* 竞价策略 Bidding Strategies
* 实时分析 Real-time Analytics
* 获取优质库存 Access to Premium Inventory:
* 创意支持 Creative Support:

## 什么是程序化广告

* 实时竞价 Real-time Bidding
* 程序化导向 Programmatic direct

## DSP 交易过程

从普通用户在浏览器中地址栏输入网站的网址，到用户看到页面上的内容和广告这短短几百毫秒之内，就需要发生了好几个网络往返(Round Trip)的信息交换。

1. Ad Exchange先向DSP发竞价(bidding)请求，告知DSP这次曝光的属性，如物料的尺寸、广告位出现的URL和类别、以及用户的Cookie ID等；
2. DSP接到竞价请求后，必须在几十毫秒之内决定是否竞价这次曝光，如果竞价，出什么样的价格，然后把竞价的响应发回到Ad Exchange。
3. Ad Exchange判定赢得了该次竞价的DSP，在极短时间内把DSP的广告主对应的广告迅速送到用户的浏览器上。 整个过程如果速度稍慢，Ad Exchange就会认为DSP超时而不接受DSP的竞价响应，广告主的广告投放就无法实现。

## DSPAN -Demand Side Platforms and Ad Networks

中国的特殊模式 ，DSP分为DSP和DSPAN，通常说的DSP只是独立第三方DSP，这类才是严格意义上的程序化投放平台，而这种在国内基本是不存在的，因为大部分的DSP都会有有些杀手锏——直接接入的媒体资源，属于DSPAN，而DSPAN其实是国内某个公司独创出来的一个词汇来的，DSPAN的交易通常是不通过广告交易平台ADX的，是平台自己有资源的交易，跟AD-Network的交易模式更接近，是一个媒体方的广告投放平台，DSPAN是供给是自己，交易平台也是自己，没有中立性可言，存在较大的操作空间。

## REF

* [Demand Side Platform (DSP): A Simple Explanation](https://revx.io/blog/demand-side-platform/)
* [How Demand-Side Platforms Enable Wider Reach & Ad Buying](https://instapage.com/blog/demand-side-platform)
* [A guide to programmatic ad buying in China](https://www.marketing-interactive.com/brief-guide-programmatic-ad-buying-china)
