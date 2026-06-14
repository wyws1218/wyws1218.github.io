---
title: 【项目总览】基于 UE5 GAS 的游戏技能框架
date: 2026-03-22 18:00:00
tags: [UE5, C++, 游戏开发, GAS, 作品集]
categories: [UE 技术, GAS]
sticky: 100
---

## 项目背景与演示

在类似 MOBA 与 MMO 游戏中，技能系统的可扩展性是核心痛点。我想要脱离庞杂的业务逻辑，从零开始使用 C++ 与蓝图混合开发的模式，搭建一套 GAS 底层框架。

目前我已经将该框架的核心运行闭环打通，并逐步封装为独立可拔插的 UE5 插件。

> 🔗 源码仓库：[点击访问我的 GitHub 源码 gasup](https://github.com/wyws1218/GasUp)

## 核心工程亮点

1. C++ 底层基建与网络优化
   * 在 BaseCharacter 中硬编码挂载 ASC，确保最高运行效率。
   * 为玩家与敌人区分了网络同步模式，从底层兼顾了预测精准度与服务器带宽性能。
2. 技能逻辑与视听表现解耦
   * 使用 Gameplay Ability 和 Task 节点处理纯逻辑。
   * 使用 Gameplay Cue 全权接管客户端的粒子特效、音效与模型显隐，实现严格的逻辑与表现分离。
3. 数据驱动设计 告别硬编码
   * 利用 Gameplay Effect 的 Cost 机制替代传统的逻辑判断，提升了代码可读性与策划配表自由度。
4. MVC 架构思考的沉淀
   * 摒弃了将移动逻辑塞入 PlayerController 的做法，将状态收束至 Character，贯彻高内聚设计。

## 详细技术沉淀 系列目录

这里记录了我在搭建这套 GAS 框架时的详细过程、源码解析以及踩坑记录：

* {% post_link GAS-Part1-Initialization 第一章：GAS 底层基建与 C++ 初始化实践 %}
* {% post_link GAS-Part2-Dash-Decoupling 第二章：冲刺技能实现、Task 运用与表现解耦 %}
* {% post_link GAS-Part3-Attribute-UI 第三章：属性集构建与 UI 解耦：消耗、冷却与动态技能框 %}
* {% post_link GAS-Part4-TargetData-Damage 第四章：TargetData 目标提取与伤害输出闭环 %}
* {% post_link GAS-Part5-Plugin 第五章：框架剥离与独立插件打包实战 %}

> 注：点击上方链接进入对应章节查看详细的蓝图截图与 C++ 源码分析。