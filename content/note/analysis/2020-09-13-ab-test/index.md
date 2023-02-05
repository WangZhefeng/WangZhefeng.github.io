---
title: A/B test
author: 王哲峰
date: '2020-09-13'
slug: ab-test
categories:
  - data analysis
tags:
  - model
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

- [什么是A/B test?](#什么是ab-test)
- [为什么需要A/B test?](#为什么需要ab-test)
- [如何做A/B test?](#如何做ab-test)
- [为什么需要对 A/B test 的结果做分析?](#为什么需要对-ab-test-的结果做分析)
  - [Z 检验计算胜出概率](#z-检验计算胜出概率)
    - [示例](#示例)
  - [区间估计计算范围](#区间估计计算范围)
- [A/B test相关的统计学理论有哪些?](#ab-test相关的统计学理论有哪些)
  - [正太分布](#正太分布)
  - [中心极限定理](#中心极限定理)
  - [区间估计](#区间估计)
  - [统计检验](#统计检验)
  - [Z 检验](#z-检验)
  - [P value](#p-value)
  - [标准正态分布表 Z 值查询方法](#标准正态分布表-z-值查询方法)
</p></details><p></p>

# 什么是A/B test?

- A/B test 是一种为了提高转换率和反向率的测试方法, 常用于网页优化及时常营销
- A/B test 核心在于: 对同一个 `要素` 有两个版本(A/B), 并且有度量哪个更成功的指标的情况下, 
   将A/B两个版本同时做实验, 然后根据度量结果决定哪个版本更好, 从而决定在生产中真正使用哪个版本. 

# 为什么需要A/B test?

产品的改变并不总是意味着进步和提高, 有时候是无法人为评判多种设计方案中哪一种更优秀, 
这时利用 A/B test 可以回答两个问题: 

1. 哪个方案好?
2. 比较结果的可信度是多少?

A/B test 结果是基于用户得到的结果, 用数据说话, 而不是凭空想象去为用户代言, 并且通过一定的数学分析给出结果的可信度. 

A/B test 需要几个前提: 

1. 多个方案并行测试；
2. 每个方案只有一个变量不同；
3. 能够以某种规则优胜劣汰；

其中: 第 2 点暗示了 A/B 测试的应用范围: A/B 测试必须是单变量, 但有的时候, 
我们并不追求知道某个细节对方案的影响, 而只想知道方案的整体效果如何, 那么可以适当增加变量, 
当然测试方案有非常大的差异时一般不太适合做A/B测试, 因为它们的变量太多了, 
变量之间会有很多的干扰, 所以很难通过A/B测试的方法找出各个变量对结果的影响程度. 

在满足上述前提时, 便可以做A/B test了. 

# 如何做A/B test?

一个完整的A/B test主要包括以下几部分: 

1. 确定测试目标. 即建立实验方案好坏的评估标准；
2. 设计分流方案；
3. 实验方案的部署；
4. 数据收集及统计
5. 结果分析, 得出结论；

需要做/可以做的事: 

1. 你需要需要知道在放弃之前需要进行多长时间的测试. 过早或过晚放弃都是不对的；
2. 注意给相同访客呈现出相同版本, 特别是一些价格什么敏感信息. (可以用cookie之类的追踪用户)；
3. 在整个网站中保持A/B测试的一致性, 不要在X页面显示A中测试元素, 而在Y页面显示B种测试元素. 
4. 做很多的A/B测试(为了达到既定的目标)；

不需要做/不可以做的事:

1. 只有在测试了你能控制A/B两种版本之后, 才开始你的测试；
   不要一段时间测试A版本, 一段时间测试B版本, 而是应该同时进行, 将流量分散为A/B两类；
2. 不要过早地下结论, 需要事先预估一个A/B测试的周期；
3. 不要令常来的访客惊讶(或困惑), 如果是测试一个核心功能, 最好只对新用户进行测试；
4. 不要让个人的喜好影响测试的结果, 因为不一定是看起来更合理(或更漂亮)的版本会获得 A/B 测试的胜利, 
   结果是什么就是什么, 数据说话, 保持客观；

测试工具: 

- Web
   - Google Website Optimilzer
   - Visual Website Optimilzer
   - Vertster
   - Preess9 A/B Testing Joomla Plugin
   - Amazon Mechanical Turk
   - Split Test Calculator
   - ABtests.com
- Mobile App
   - clutch.io
   - pathmapp

# 为什么需要对 A/B test 的结果做分析?

## Z 检验计算胜出概率

并不是所有的实验都能做到样本足够大、区分度足够高, 所以需要使用统计假设检验来验证实验的结果；

### 示例

以 `转化率` 为例, 运行 A/B test 一周, 分别对 1000 个样本进行了测试. 

- A版本的转化率为`$7.5\%$`
- B版本的转化率为`$9\%$`

对于这个实验结果, 有两个疑问: 

- 能否肯定B版本比A版本好?
- 有多大的可能是因为一些随机的因素导致了这样的差别?

假设检验能够有效地回答这个问题. 首先设定 `零假设` : 

`$$H0: B版本在转化率效果不会比A版本好$$`

然后, 通过证据(实验观察到的样本)来推翻这个假设. 
如果样本足以推翻上面的零假设, 那么可以认为实验完成了, 
可以的到的结论为: B 版本在转化率效果比A版本好；
否则, 需要继续实验来得到更多的实验观测数据样本来进行推断, 
或者, 干脆就接受这个假设并舍弃B版本, 即认为B版本在转化率方面没有A版本效果好. 

将上面的假设检验抽象为统计学理论: 

- 假设随机变量 `$X = P(B)-P(A)$` 是两个版本实际转化率的差异度, 
  其中, `$P(B)$` 是 `$B$` 的转化率, `$P(A)$` 是A的转化率. 
- 定义假设: `$B版本不比A版本效果好$`
   - 原假设: `$H0: P(B)-P(A) \leq 0$`
   - 备择假设: `$H1: P(B)-P(A)>0$`
- 一个用户, 要么注册, 要么不注册, 所以A和B均满足二项分布, 即
   - `$A \sim b(N, P(A))$`
   - `$B \sim b(N, P(B))$`, `$N$` 是样本数量；
- 根据中心极限定理, `$A$` 和 `$B$` 可以近似为正态分布, 那么, 随机变量 `$X = P(B)-P(A)$` 也服从正太分布:
   - `$X \sim N(0, \frac{P(B)(1-P(B))}{N} + \frac{P(A)(1-P(A))}{N})$`,
      其中分布的期望为 `$0$`, 因为原假设的期望是 `$B$` 版本和 `$A$` 版本没有显著的差异, 即 `$E(X)=0$`
- 对`$X$` 的分布进行标准化, 然后选择 `$5\%$` 的区间作为拒绝域, 
  即如果`$X$` 标准化后的值落在了标准正态分布的最右端 `$5\%$` 的面积里, 
  那么可以具有很强的信心(`$1-5\%=95\%$`)拒绝原假设 `$H0$`, 
  即得到结论 `$B$` 版本比A版本效果好. 
- 假设`$X标准化后的随机变量为 Z$`

## 区间估计计算范围

# A/B test相关的统计学理论有哪些?

## 正太分布

## 中心极限定理

## 区间估计

## 统计检验

## Z 检验

## P value

## 标准正态分布表 Z 值查询方法