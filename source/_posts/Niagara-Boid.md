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

添加渲染阶段，渲染阶段的主要属性，是说明这个阶段需要针对哪个对象，进行几次迭代。

可选的对象有粒子、RenderTarget、Grid2D，对于RenderTarget则针对每一个纹素进行处理，对于Grid2D则针对每一个网格点。

相当于是在粒子的执行栈中依次加入的执行阶段，每个粒子在Update的过程中全部执行一个阶段之后才会执行下一个Simulation Stage。每个阶段之间的数据传输除了依靠Attribute Reader之外，最主要的就是依靠数据结构Grid2D,Neighbor Grid3D等。

模拟阶段开启：
将Emitter设置为GPU渲染，并打开显示高级属性(右上角的小眼睛)，便能够打开渲染阶段。
![Open Simulation Stage](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Niagara/SimulationStageOpen.png?raw=true)

<!-- more -->

## 常用数据结构

1. Render Target
   
	是GPU的一个特征，他允许将3D场景渲染到中间存储缓冲区或者渲染目标纹理（RTT），而不是帧缓冲区或后缓冲区。 
	
    默认的渲染目标为后备缓冲，物理上就是下一帧所需要的绘制的信息，显存的宽高与游戏的分辨率相同。可以使用RenderTarget类创建另一个渲染目标，在显存中保留一块新区域用于绘制。
	是一种可以在游戏运行的时候写入的纹理。

2. Grid2D
   	
    粒子系统模拟阶段的数据接口，是一种存储数据的结构，有点类似高级版的RenderTarget，需要定义Grid2D的大小。
       
    Grid2D中的数据可以通过命名空间自动添加。
    ![Namespace](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Niagara/Namespace.png?raw=true)

3. Neighbor Grid 3D
   
   是一种基于位置的哈希查找结构。在它最简单的形式中，它允许将粒子划分一个网格，然后稍后从查找位置进行查询。
    
    - Neighbor Grid3D的使用需要在粒子系统的系统层进行初始化，初始化的主要参数有：
	- World Bbox Size 三维大小
	- Cell Size 单元格大小
	- Max Neighbor Per Cell : 每个单元的最大邻居数。这个值将在计算线性索引值中被使用到。
	
    Neighbor Grid3D相关的方法位于源码中\Niagara\Source\Niagara\Private\NiagaraDataInterfaceRW.cpp和NiagaraDataInterfaceNeighborGrid3D.cpp。

	Boid鸟群就是通过三维空间中的NeighborGrid3D结构，去获取鸟周围的邻居鸟类的粒子信息
    
    在进行关于NeighoborGrid3D的相关操作时，首先需要将粒子从自身的坐标系归一化，常用的转换矩阵为:
    [
        [Scale.x 0 0 0],
        [0 Scale.y 0 0],
        [0 0 Scale.z 0],
        [0.5,0.5,0.5 1]
    ]

    更新NeighborGrid3D的过程一般是：
    
     - 归一化坐标系
     - 得到粒子所在网格位置的x,y,z三个轴相对应的序号索引值
     - 如果该三维的序号在设置的NeighborGrid3D的范围中，将该三维的序号转换为线性索引值
     - 获取该线性索引值序号对应的格子中的粒子数量，加1
     - 如果该粒子数量小于Max Neighbor Per Cell，则获取对应格子存储空间中第i个存储位置的索引值（i为粒子数量）
     - 将粒子序号存储在上一步中的索引值内。

    对应的自定义代码则是
    ```
    //根据粒子位置将粒子分配到对应的子空间Neighbor Grid3D中
	OutPosition = Position;
	
	#if GPU_SIMULATION
	
	float3 UnitPos;
	NeighborGrid.SimulationToUnit(Position, SimulationToUnit, UnitPos);
	
	int3 Index;
	NeighborGrid.UnitToIndex(UnitPos, Index.x,Index.y,Index.z);
	
	int3 NumCells;
	NeighborGrid.GetNumCells(NumCells.x, NumCells.y, NumCells.z);
	
	int MaxNeighborsPerCell;
	NeighborGrid.MaxNeighborsPerCell(MaxNeighborsPerCell);
	        
	if (Index.x >= 0 && Index.x < NumCells.x && 
	    Index.y >= 0 && Index.y < NumCells.y && 
	        Index.z >= 0 && Index.z < NumCells.z)
	{
	    int LinearIndex;
	    NeighborGrid.IndexToLinear(Index.x, Index.y, Index.z, LinearIndex);
	
	    int PreviousNeighborCount;
	    NeighborGrid.SetParticleNeighborCount(LinearIndex, 1, PreviousNeighborCount);
	
	    if (PreviousNeighborCount < MaxNeighborsPerCell)
	    {
	        int NeighborGridLinear;
	        NeighborGrid.NeighborGridIndexToLinear(Index.x, Index.y, Index.z, PreviousNeighborCount, NeighborGridLinear);
	
	        int IGNORE;
	        NeighborGrid.SetParticleNeighbor(NeighborGridLinear, InstanceIdx, IGNORE);
	    }		
	}
	#endif

    ```

## Boid鸟群

Boids主要包含三个部分：
1. Alignment：与周围的鸟群速度平行
2. Separation：离得太近会产生排斥力
3. Cohesion：向周围鸟群的聚合点移动

![Boid](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Niagara/Boid.png?raw=true)

实现的代码逻辑：
1. 定义主要参数：
    
    - RepulsiceForce 对应Separation排斥力
    - VelocityMatchingForce 对应Alignment的速度平行力
    - CohesionCenter 对应Cohesion聚合力的中心位置 
2. 利用Neighbor Grid 3D获取周围子空间的粒子序号，进行遍历
3. 通过点乘计算出邻接粒子与当前粒子的夹角，将其重映射到[0,1]中，如果这个角度在当前粒子的视线范围内并且距离在设置的阈值中，再进行后续的计算，并且记录邻居数量+1
4. 计算权重，权重越大代表距离当前粒子越近
5. 将范围内的粒子位置加入CohesionCenter+=OtherPos
6. 计算排斥力的方向（图中绿色箭头1）
   ```
   float3 VectorTowardCurrentBird = NormalizedVectorToOtherParticle * float3(-1, -1, -1);
   float3 CollisionAvoidanceVector = cross( NormalizedFacingVector, VectorTowardCurrentBird );
   CollisionAvoidanceVector = cross( CollisionAvoidanceVector, VectorTowardCurrentBird );
   ```
   示例中还将排斥力方向乘了MovementPerferenceVector，这个向量一般是各个方向的权重，比如说可以定义为(1,1,0.5)，则在z方向偏移的权重会小一点

![Example1](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Niagara/Example1.png?raw=true)

7. 如果邻接粒子在最小距离之内，才计算它的排斥力，需要记录MinDistanceNeighborCount+=1，并且同样计算一个权重，权重越大，距离越近。
8. 	在计算排斥力时，还要计算两个粒子的速度方向夹角。
	
    如果夹角小于零，证明两者速度是朝向彼此的Facing each other，那么最后的排斥力RepulsiveForce+=CollisionAvoidanceVector*权重；
	否则排斥力的方向为绿色箭头2，并且需要计算Alignment力，力为OtherVelocity-Velocity。
9. 	以上为三种力的计算，经过计算之后，需要利用MinDistanceNeighborCount以及NeighborCount去更新粒子的属性。
	
    对于所有的MinDistanceNeighborCount（也就是在视线范围内的鸟群）
	
    基本逻辑都是 力/数量*力的系数（自定义的）
10. 对于所有的NeighborCount
	
    更新CohesionCenter ，需要除以NeighborCount+1个数，然后再与粒子本身的位置相减得出。
	
    由于在更新NeighborCount时没有进行粒子距离与最大距离的判断，所以需要定义距离的权值，权值越大，距离越近。

![Example2](https://github.com/RachelLiuYY/RachelLiuYY.github.io/blob/Hexo/source/_posts/Image/Niagara/Example2.png?raw=true)

11. CohensionForce为VectorToCenter的方向*Cohesion力的大小（这个地方可以乘以上一步的权重
12. 最后输出的OutForce为定义的CohesionForce + RepulsiveForce + VelocityMatchingForce

