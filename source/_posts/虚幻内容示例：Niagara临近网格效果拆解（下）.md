---
title: "虚幻内容示例：Niagara临近网格效果拆解（下）"
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
cover: "/posts/虚幻内容示例：Niagara临近网格效果拆解（上）/虚幻内容示例：Niagara临近网格效果拆解-1773456706336.png"
---


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
