---
title: "虚幻内容示例：Niagara临近网格效果拆解"
comments: true
toc: true
donate: true
share: true
date: "2026-03-14 10:32:59"
categories:
    - [unreal, Niagara]
tags:
  - "笔记"
excerpt: ""
cover: "/posts/虚幻内容示例：Niagara临近网格效果拆解/虚幻内容示例：Niagara临近网格效果拆解-1773456706336.png"
---

本文是对虚幻引擎内容示例（Content Examples）中 Niagara 高级示例部分的拆解笔记。

示例项目 Fab 链接：[https://www.fab.com/listings/4d251261-d98c-48e2-baee-8f4e47c67091](https://www.fab.com/listings/4d251261-d98c-48e2-baee-8f4e47c67091)

## 什么是 Neighbor Grid 3D | 临近网格体

Neighbor Grid 3D（临近网格体）是 Niagara 中用于空间查询的数据结构。

### 为什么需要它

- **降低计算复杂度**
    - 在没有空间分区的情况下，若每个粒子都要感知周围粒子，计算量为 $O(n^2)$（遍历所有粒子）。使用网格后，查询范围被限制在相邻体素内，复杂度降低至接近 $O(n)$。
- **突破 GPU 线程限制**
    - GPU 擅长并行计算，但不擅长全局搜索。网格体提供了一个预排序的查找表，让成千上万个线程能同时在局部区域内高效定位数据。
- **实现复杂的群体智能**
    - 像 Boids 算法中的对齐（Alignment）和斥力（Separation）需要频繁读取邻居的速度和位置，网格体是支撑实时高密度群体行为的性能底座。
- **优化显存带宽利用**
    - 通过将空间相近的粒子数据在逻辑上归类，减少了在执行空间查询时对显存的随机访问开销。
### 它是如何做的

它将三维空间划分为均匀的体素网格，每个体素存储该空间范围内粒子索引的列表。通过这种数据结构，粒子可以高效地查询其邻近的其他粒子，而无需遍历所有粒子进行距离计算。
![](./虚幻内容示例：Niagara临近网格效果拆解/虚幻内容示例：Niagara临近网格效果拆解-1773458232714.webp)

这为实现粒子间的连线效果、碰撞检测、群体行为等提供了性能优化的基础。

<!-- more -->

## Position Based Dynamics | 基于位置的动力学
<video width="100%" height="auto" controls>
  <source src="/posts/虚幻内容示例：Niagara临近网格效果拆解/compressed_video.mp4" type="video/mp4">
  您的浏览器不支持 HTML5 视频播放。
</video>

> 视频中的立方体是对 Neighbor Grid 3D 网格的可视化，但NeighborGrid3D 接口本质是 **GPU 内存中的线性缓冲区**（Array），用于存储粒子的索引信息。

一切的基础，也是对 Neighbor Grid 3D 最基本的使用。本节将介绍如何在虚幻引擎中实现 PBD，以及在虚幻中使用 PBD 所需的必要组件和设置。

### 发射器概览
![](./虚幻内容示例：Niagara临近网格效果拆解/虚幻内容示例：Niagara临近网格效果拆解-1773459911026.webp)




## Plexus | 网络连线

使用 Neighbor Grid 3D 获取临近网格信息，基于粒子间距离阈值，实现粒子之间的动态连线效果。


### 发射器概览


## Structural Support | 结构支撑

一个多阶段发射器和基于支撑树结构的支撑关系的 Niagara 实现。

### 发射器概览


## Boids | 鸟群模拟

在虚幻中使用 Niagara 实现的鸟群效果。本节将简单介绍鸟群算法，并阐述虚幻对鸟群行为的扩展实现。


### 基础鸟群算法

经典的 Boids 算法由 Craig Reynolds 于 1986 年提出，通过三条简单规则模拟群体行为：

| 规则 | 英文名     | 描述                                   |
| ---- | ---------- | -------------------------------------- |
| 分离 | Separation | 与邻近个体保持一定距离，避免拥挤碰撞   |
| 对齐 | Alignment  | 与邻近个体的飞行方向趋同               |
| 聚合 | Cohesion   | 向邻近个体的平均位置移动，保持群体聚集 |

虚幻内容实例在这个基础上扩展了一些内容：
既然你在深入研究 Niagara 中的群体仿真，将这些逻辑模块化确实能让粒子行为更具“生物感”。以下是按照你的要求整理的列表：

### 虚幻内容示例对鸟群算法的优化

* **Avoid the Environment | 环境避让**
  * 增加对场景碰撞体（墙壁、地形、静态网格）的感知与避让行为。
* **Form Clusters | 动态聚类**
  * 基于距离阈值的动态子群体形成机制。
* **Boid Proximity Avoidance | 近距离避让**
  * 分离规则的紧急程度分层处理。
* **Match Velocity | 速度匹配**
  * 与邻近鸟群成员的速度趋同行为。
* **Avoid Head-on Collisions | 避免正面碰撞**
  * 预测未来轨迹冲突，提前进行避让。

### 发射器概览
