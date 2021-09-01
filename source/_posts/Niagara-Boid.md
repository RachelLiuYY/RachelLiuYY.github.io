---
layout: article
title: Niagara_Boid
date: 2021-08-30 17:13:14
tags: [UE_Niagara]
---

> UE4中对于Niagara在进行进一步的完善，在4.26推出了Simulation Stage支持了更高级的应用，在官方示例中利用渲染阶段创建了许多高级的用例，例如PBD，Boid鸟群，与环境的交互等，还有很多大佬利用Niagara模拟流体等效果。

> 这次主要是通过官方示例中的Niagra_Advanced关卡中的Boid鸟群更好的了解使用流程

## Simulation Stage

模拟阶段是一种迭代粒子、渲染目标纹素或网格集合的网格单元的实验方法。

这解锁了编写迭代约束、基于网格的求解器、将属性传入和传出网格以及直接输出这些属性以渲染目标以在材质中渲染的能力。

模拟阶段开启：
将Emitter设置为GPU渲染，并打开显示高级属性

