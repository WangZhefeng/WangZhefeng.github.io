---
title: 时间序列预测-分解
author: 王哲峰
date: '2022-04-25'
slug: timeseries-model-decomposition
categories:
  - timeseries
tags:
  - ml
---

<style>
details {
    border: 1px solid #aaa;
    border-radius: 4px;
    padding: .5em .5em 0;
}
summary {
    font-weight: bold;
    margin: -.5em -.5em 0;
    padding: .5em;
}
details[open] {
    padding: .5em;
}
details[open] summary {
    border-bottom: 1px solid #aaa;
    margin-bottom: .5em;
}
</style>

<details><summary>目录</summary><p>

- [时间序列分解模型](#时间序列分解模型)
  - [加法模型](#加法模型)
  - [乘法模型](#乘法模型)
  - [加法乘法混合模型](#加法乘法混合模型)
- [时间序列的分解方法](#时间序列的分解方法)
  - [使用移动平均法分离出显性的周期性波动](#使用移动平均法分离出显性的周期性波动)
  - [将业务周期性波动效应和随机波动进行分解](#将业务周期性波动效应和随机波动进行分解)
  - [观察数据波动的拐点将时间序列分段](#观察数据波动的拐点将时间序列分段)
  - [利用线性回归基于移动平均数计算长期趋势](#利用线性回归基于移动平均数计算长期趋势)
  - [分离出循环效应和随机波动](#分离出循环效应和随机波动)
  - [检验时间序列分解的效果](#检验时间序列分解的效果)
- [时间序列分解方法的应用局限性](#时间序列分解方法的应用局限性)
</p></details><p></p>

# 时间序列分解模型

时间序列数据，即数据指标按时间维度统计形成的序列。这种数据在我们的日常报表中非常常见。
观察这类数据的出发点有两个：

* 一是长期追踪，一旦指标出现上涨和下跌，能直观地观察到，进而去调查原因
* 二是判断趋势，通过指标的波动，判断指标在未来的走势
  
第一点相对简单，看到指标变化后从不同维度不断下钻，总能找到原因。
第二点则要从时间序列的波动中看出门道，不是光盯着数据看就可以的，
最常见的逻辑就是 “将时间序列波动的信息进行分解”

在时间序列分析中, 常见的分析方法叫做时间序列的分解(Decomposition of Time Series)。
通过分解，将数据分解成可预测部分和不规则变动(随机波动)部分，
可预测部分占比比不规则变动大很多，那么就具备了预测未来的条件

时间序列分解的原理是将时间序列 `$Y_{t}$` 分为三个或四个部分:

`$$Y_{t} = f(T_{t}, S_{t}, C_{t}, I_{t})$$`

* 长期趋势因素(Secular trend) `$T_{t}$`
    - 数据中对时间的变化相对稳定的一部分因素。往往是长期稳定的上涨或下跌。
      这个数据一般可以通过移动平均或者线性回归等方法进行拟合，因此它是可预测的部分
* 季节/周期变动因素(Seasonal Variation) `$S_{t}$`
    - 传统的时间序列分解方法一般用在长期的宏观经济指标中，因此颗粒度是季度，所以会呈季节性变动。
      在数据运营的场景中，季节数据跨度太长，几乎没有使用的必要性。
      所以将季节变动引申为“周期性波动”，而且是显性的周期性波动，
      例如业务指标在一周内会有周末和工作日的差别，在一个月中会有月初和月末的差别。
      周期性波动因素取决于数据处在周期中的位置，通过固定位置的历史数据(取均值或者其他数学变换)，
      也能对未来的某个位置的周期性因素进行估计，因此它也是可预测的部分
* 循环变动因素(Cyclical Variation) `$C_{t}$`
    - 循环变动和季节变动其实很像，也有周期性因素在。但循环变动的周期是隐性的，
      往往要先将显性的周期性波动排除后，再观察剩下的数据部分是否有循环波动的因素，
      若有，也能通过同比计算等方法将其提出，因此也是可预测的
* 随机变动因素(Irregular Variation) `$I_{t}$`
    - 既然是随机波动，自然是不可预测的
    - 时间序列分解的成功与否，取决于两个因素：一是数据序列本身是隐藏着规律的，不可预测的部分只是其中的一小部分；
      二是分解的方法要合适，尤其是周期的判断要准确。因此，这个方法非常考验使用者的经验和直觉

时间序列分解法试图从时间序列中区分出这四种潜在的因素, 特别是长期趋势因素(T)、季节变动因素(S)、
周期性因素(C). 显然, 并非每一个预测对象中都存在着 T、S、C 这三种趋势, 可能是其中的一种或两种. 
一个具体的时间序列究竟由哪几类变动组合, 采取哪种组合形式, 应根据所掌握的资料、时间序列及研究目的来确定

## 加法模型
   
`$$Y_{t} = T_{t} + S_{t} + C_{t} + I_{t}$$`

其中: 

* `$Y_{t}, T_{t}, S_{t}, C_{t}, I_{t}$` 均有相同的量纲
* `$\sum_{t=1}^{k}S_{t}=0$`，`$k$` 为季节性周期长度
* `$I_{t}$` 是独立随机变量序列，服从正态分布

加法模型中的四种成分之间是相互独立的, 某种成分的变动不影响其他成分的变动. 
各个成分都用绝对量表示, 并且具有相同的量纲

## 乘法模型

`$$Y_{t} = T_{t} \times S_{t} \times C_{t} \times I_{t}$$`

其中:

* `$Y_{t}$`，`$T_{t}$` 有相同的量纲
* `$S_{t}$` 为季节性指数，`$C_{t}$` 为循环指数，两者皆为比例数
* `$\sum_{t=1}^{k}S_{t}=k$`，`$k$` 为季节性周期长度
* `$I_{t}$` 是独立随机变量序列，服从正态分布

乘法模型中四种成分之间保持着相互依存的关系, 一般, 长期趋势用绝对量表示, 
具有和时间序列本身相同的量纲, 其他成分则用相对量表示

上面的乘法模型等价于:

`$$\ln y_{t} = \ln S_t + \ln T_t + \ln C_t + \ln I_{t}$$`

由于上面的等价关系, 所以, 有的时候在预测模型的时候会先取对数, 
然后再进行时间序列的分解, 就能得到乘法的形式

## 加法乘法混合模型

`$$Y_{t} = T_{t} \times S_{t} \times C_{t} + I_{t}$$`

其中: 

* `$Y_{t}, T_{t}, C_{t}, I_{t}$` 有相同的量纲，`$S_{t}$` 是季节指数，为比例数
* `$\sum_{t=1}^{k}S_{t} = k$`，`$k$` 为季节性周期长度
* `$I_{t}$` 是独立随机变量序列，服从正态分布

# 时间序列的分解方法

时间序列数据的预测值就是:

长期趋势(线性回归估计值) + 循环效应(循环周期各位置的均值) + 周期效应(业务周期各位置的均值)

就意味着，能通过时间长度和所在周期的位置给出一个未知时间点的预测值

## 使用移动平均法分离出显性的周期性波动

* 首先，清洗数据，将异常值剔除
* 其次，根据现实业务周期(显性周期)，按周期的长度求移动平均数，
  这样所获得的移动平均数就是排除了周期性波动影响和一部分随机波动的数据(`$SI_{1}$`)，
  即移动平均数为长期趋势、循环波动和一部分随机波动(`$TCI_{2}$`)
* 最后，用原始数据减去移动平均数(`$TCI_{2}$`)，就得到了周期性波动效应和一部分随机波动(`$SI_{1}$`)

## 将业务周期性波动效应和随机波动进行分解

这部分的处理取决于周效应和不规则变动的量级。在实际场景中，若量比较大，建议计算每周中对应某一天的均值，
即得到周一的均值、周二的均值等，这便是加法模型中的周效应

`$S = SI_{1} / I_{1}$`

## 观察数据波动的拐点将时间序列分段

移动平均数包含了数据的长期趋势(`$T$`)、循环变动(`$C$`)

* 首先，长期趋势是会改变的，这种改变往往是运营策略的变化带来的，所以不能教条地假设长期趋势稳定不变
* 其次，在数据的不同阶段，循环的周期也会有所不同

## 利用线性回归基于移动平均数计算长期趋势

原始数据在剔除了业务周期波动和随机波动后(`$SI_{1}$`)，剩下了长期趋势和循环变动(`$TC$`)

长期趋势与时间的增加是有关系的(建模的原始假设)，因此以时间为自变量(`$t$`)(起点为0，之后每天都以1自增的序列)，
以业务周期移动平均数(`$TC$`)为因变量，构建一个线性回归模型。
由时间(`$t$`)和回归模型计算得出的因变量(`$TC$`)的估计值，就是长期趋势 `$T$`，用移动平均数减去 `$T$`，
剩下的部分就是循环效应和一部分的随机波动(不规则变动)。

需要注意的是，估计长期趋势(趋势拟合的方法)并不是只能采用线性回归。
这取决于数据点的分布，有时要用指数回归，有时要用多项式回归。
而且，在数据的不同阶段，使用的长期趋势估计方式也可以是不同的

## 分离出循环效应和随机波动

因为循环效应不是那么容易观察出来的。一个简单的观察办法是：看数据是否有规律地分布在 0 值之上和 0 值之下。
若数据不规则地在 0 值上下跳动，则可以认定这是随机波动，不需要分离循环效应。
若数据一段时间在 0 之上，一段时间在 0 之下，且持续的时间大致相同，那么就有必要分离循环效应

一个波峰加一个波谷所跨越的时间，就是循环的周期(这个规则适用于所有周期性数据的判断)。
计算循环中各个位置的均值，即为循环效应

移动平均数(`$TCI_{2}$`)与线性回归值(`$T$`)的差值，即“循环效应+不规则变动”(`$CI_{2}$`)，
减去循环效应(`$C$`)后(除了阶段2，其他阶段的循环效应认为是0)，
剩下的就是随机波动(`$I_{2}$`)

## 检验时间序列分解的效果

* 第一种手段就是图形法，观察预测值与实际值的契合程度
* 第二种方法是回归分析法
    - 以预测值为自变量，实际值为因变量，建立一个线性回归模型，
      观察模型的拟合优度，通过拟合优度判断预测是否靠谱

# 时间序列分解方法的应用局限性

每种分析方法都有它的局限性，时间序列分解方法也一样，“分解”这种思维，事实上是可以应用在更广泛的业务分析中的，
而不仅是时间序列数据。通过以上案例，需要注意时间序列分解法中的以下几点局限性

1. 原始数据中的随机波动因素占比不能过大
    - 随机波动因素的占比过大，说明我们不可预测的东西过多，那么，剩余的部分再怎么分解也无济于事
2. 分解的过程中，确定移动平均的期数、数据阶段的划分、趋势拟合的方法、循环周期都带有一定的主观判断。
   这就对分析者提出了较高的要求。在应用时，需要不断地改变这些参数来获得更好的结果。而且，经常会出现仁者见仁的局面
3. 用加法模型、乘法模型或混合模型没有定论，需要具体问题具体分析。实际情况中，往往是混合模型用得比较多
4. 需要用在长期的数据序列中。时间序列的分解对时间的长度是有要求的，却没有明确的阈值
    - 至少要在 40 个数据点以上才能讨论所谓的长期趋势。另外，该方法不适合用在比“天”的颗粒度更小的时间维度上
5. 时间序列阶段的改变可预测性较差
    - 将时间序列分解的结果应用于预测时，是不知道何时进入新的阶段的(序列的结构性断点不可预测)。
      今天还在阶段1，明天就进入阶段2了，这可如何是好？有一些缓解这个问题的方法：
        - 一是做“事后诸葛亮”，即连续追踪数据，若连续出现上涨或者下跌，或者出现“史无前例”的最大值和最小值，
          那么就要考虑数据的结构性变化可能出现了，就要放弃原先的建模方式；
        - 二是从业务决策上“明察秋毫”，数据出现结构性变化，往往是较大的决策改变或者产品迭代引起的，
          那么反过来思考，若业务出现一些“重大改变”，也许就应该重新建模了。
6. 真正的预测，只能在阶段内进行
    - 在本例中能预测未来数据的其实也就只有阶段4。但也不用慌，历史往往会重演。
      前面三个阶段的数据特征，一定会出现在未来的某个时间点。
      所以，当数据进入有“历史参考”的某个阶段时，可以用历史经验预测未来的走势
