---
title: UE4泛型
date: 2021-06-24 14:32:35
tags: [UE4]
---

## UE4自定义泛型蓝图节点
> 起因是在存取数据表的时候，希望蓝图节点的输入能够普适所有的结构体（只要继承自FTableRowBase）就可以，所以就去查找了相关的实现方法。

关键词： UE4, Wildcard, customThunk, Generic

想实现的效果就是类似GetDataTableRow的输出节点

![Get DataTable Row](https://raw.githubusercontent.com/RachelLiuYY/RachelLiuYY.github.io/Hexo/source/_posts/Image/GetDataTableRow.png)

UE4的蓝图和C++都是静态类型的编程语言，因此想要蓝图节点支持任意类型的参数要么使用基类指针作为参数然后在具体实现时Cast，根据反射信息具体处理；要么使用Wildcard（通配符）实现。

<!-- more -->

因此需要先了解UE4中相应的实现方法：
1. 继承UK2Node类，并根据需要实现其派生类，是对蓝图节点最深入的定制开发，可以在编辑模式时动态添删除蓝图节点的针脚。
2. 使用UFUNCTION中CustomThunk说明符以及相应的类型说明符标识wildcard参数，并为该蓝图函数自定义DECLARE_FUNCTION()函数体。

第二种方式主要是利用CustomThunk标识符，使得UHT(Unreal Header Tool)不要生成默认的蓝图包装函数，而是自定定义函数体，这种方式需要手工控制蓝图的“栈”，但是相比方法1来说不需要处理蓝图的编辑器UI部分，相对简单。

## 泛型蓝图节点组成
### 函数声明
在UFUNCTION宏中包含CustomThunk说明符，泛型中具体的参数和依赖关系由meta说明符列表决定。

标识泛型函数通配符(wildcard)参数的说明符也有四种，分别为标识单个变量SingleVariable wildcard类型的"CustomStructureParam"说明符、标识容器Array的"ArrayParm"说明符、标识容器Map的"MapParam"说明符、标识容器Set的"SetParam"说明符以及各种辅助说明符。

- ArrayTypeDependentParams可以描述数组参数的依赖关系
- MapKeyParam和MapValueParam说明符标识的参数可以与MapParam说明符标识的参数相互依赖
- SetParam说明符列表的参数之间可以使用"," "|"两种分隔符；多个泛型参数的依赖关系由分隔符的类型决定，以逗号","分隔的多个泛型Set参数之间相互独立，以"|"分隔的多个泛型Set参数之间相互依赖。


### 自定义函数体
定义了泛型蓝图函数的Thunk函数体，主要的作用是从蓝图虚拟机中的“栈”获取传递的参数。

### 真正执行的函数逻辑


每一种Property都有两个基本属性，PropertyAddress和Property Size。不同类型的Property除了内存地址不一样，所占用的内存空间也不同。

对于派生与FProperty类的类型，都可以直接使用FProperty* 指示空间大小。

对于Map/Array/Set则需要分别使用FMapProperty*/FArrayProperty*/FSetProperty*来表示。


参考：
1. https://zhuanlan.zhihu.com/p/149838096
2. https://zhuanlan.zhihu.com/p/149869329
3. https://zhuanlan.zhihu.com/p/148209184
4. https://neil3d.github.io/unreal/blueprint-wildcard.html