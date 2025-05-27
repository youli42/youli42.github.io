---
title: 虚幻大典籍：Niagara流体概述
comments: true
toc: true
donate: true
share: true
categories: unreal
tags:
    - unreal
    - 官方示例
    - 特效
    - Niagara
cover: /posts/虚幻大典籍：Niagara流体概述/ContentExamples.png
---
# 虚幻大典

虚幻内容示例，人称虚幻大典，是虚幻官方对引擎功能的系统性展示。

- 虚幻内容示例官方文档：[内容示例](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/content-examples-sample-project-for-unreal-engine)
- Fab下载页面：[Content Examples](https://www.fab.com/listings/4d251261-d98c-48e2-baee-8f4e47c67091)

# Niagara Fluids | Niagara 流体
示例概述：

二维与三维气体及液体模拟示例。展示如何设置碰撞体、光照、父子级联模拟、质量与性能平衡等要素。所有演示案例均遵循以下核心原则：
1）所有模拟逻辑均通过Niagara模块实现，而非依赖引擎底层隐藏代码；
2）高分辨率网格通常视觉效果更优，但会显著增加资源消耗。需谨慎分析流体模拟对性能与内存的影响；
3）二维模拟在运行效率与内存占用方面普遍优于三维模拟；
4）碰撞体设置支持多样化实现方案。
必须在PIE（编辑器内运行模式）中执行方可在此房间外查看完整模拟效果
当前支持平台：Windows dx11/dx12 系统及 Windows/Linux Vulkan 系统
（注：PIE为游戏引擎术语，指Play-In-Editor模式；dx11/dx12为微软图形接口标准；Vulkan为跨平台图形API）

# Gas Simulations | 气体模拟

官方介绍：

气体模拟通过在网格中追踪密度和温度数据，并使其根据产生流体行为的速度场进行运动。适用于爆炸、火焰、烟雾、灰尘、云层等效果。

2D气体系统：
优势：运算速度快且内存占用低。适合游戏场景及部分影视镜头。
特点：效果通常呈现平面化特征，缺乏小尺度细节。更适用于火把类火焰及小型爆炸元素的制作。不适合需要占据大范围三维空间的特效。

3D气体系统：
资源消耗：相较2D系统需要更多内存和计算资源。
适用场景：专为核心实时特效（游戏场景）及影视级制作设计。元素具有立体感，并能精细调控视觉效果。在性能允许的情况下，可模拟任意规模的气态介质效果。