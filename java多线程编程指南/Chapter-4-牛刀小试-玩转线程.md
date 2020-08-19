---
title: Chapter 4 牛刀小试 玩转线程
date: 2020-04-17 10:20:08
categories:
- 多线程
- Java
- Java多线程编程实战指南 核心篇 黄文海
tags:
- 线程
- Java
---

# Chapter 4 牛刀小试：玩转线程

1. 多线程可以按照数据的分割方式实现，也可以按照任务的分割方式实现；

2. 合理设置线程数：对于CPU密集型线程，线程数可以设置为N~cpu~+1;如果是I/O密集型线程，优先考虑多线程数目设置为1，如果一个线程不够用可以考虑设置为N~cpu~*2；

3. 通常线程合理大小可以用一个公式来描述：
   $$
   N_{threads} = N{cpu} * U{cpu} * (1 + \frac{WT}{ST})
   $$
   其中，
   $$
   N_{threads}
   $$
   为最终合理线程数，N~cpu~是CPU的数目，U~cpu~是目标CPU使用率(通常取0.75,其范围是1<= U~cpu~ <=1)，WT(Wait Time)是程序花费在等待上的时长,ST(Service Time)是程序实际占用处理器执行计算的时长，其数值可以利用jvisualvm来观察，Total Time是总时长,Total Time(CPU)是ST，然后前者减后者的就是WT。