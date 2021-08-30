---
layout: article
title: Niagara_LOD
date: 2021-08-06 11:12:12
tags: [UE_Niagara]
---

## 在Niagara中实现LOD效果

> 主要借助Niagara中的VisibilityTag参数实现的
> 
> 基本思路：获取摄像机到粒子的距离，判断是几级LOD，赋值VisibilityTag

<!-- more -->

需要写一个模块，将模块添加在Particle Update下面，因为要动态的判断相机距离粒子的距离
![Module](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Niagara_LOD.png?raw=true)

其中Dis参数是用来Debug使用的，为了查看具体是每个粒子到视点的距离
具体Debug使用的材质为
![Mat](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Debug_Mat.png?raw=true)
在粒子系统里的效果
![Debug](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Niagara_Debug.png?raw=true)

LODLength对应的参数就是LOD分级的各个距离

粒子的VisibilityTag默认没有开启设置，需要在参数面板添加设置，可以在Render部分的细节面板中的绑定模块看到，当粒子的值与RenderVisibility的值相等时，粒子显示；否则不显示。
![VisibilityTag](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Niagara_VisibilityTag.png?raw=true)

最终实现的效果：
![Final](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Animation.gif?raw=true)

