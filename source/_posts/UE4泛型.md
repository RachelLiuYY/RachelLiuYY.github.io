---
title: UE4泛型
date: 2021-06-24 14:32:35
tags: [UE4] [generic]
---

## UE4自定义泛型蓝图节点
> 起因是在存取数据表的时候，希望蓝图节点的输入能够普适所有的结构体（只要继承自FTableRowBase）就可以，所以就去查找了相关的实现方法。

关键词： UE4, Wildcard, customThunk, Generic

想实现的效果就是类似
![avatar]()


参考：
1. https://zhuanlan.zhihu.com/p/149838096
2. https://zhuanlan.zhihu.com/p/149869329
3. https://zhuanlan.zhihu.com/p/148209184
4. https://neil3d.github.io/unreal/blueprint-wildcard.html