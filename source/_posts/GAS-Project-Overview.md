---
title: 基于UE5 GAS框架的游戏技能插件
date: 2026-03-22 18:00:00
tags: [UE5, C++, 游戏开发, GAS, 作品集]
categories: [UE 技术, GAS]
sticky: 100
---

## 项目背景

为了解决技能系统耦合度高、扩展性差的问题，利用 GAS 框架制作了可复用的技能框架插件。

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

## 系列目录

* {% post_link GAS-Part1-Initialization 第一章：GAS 底层基建与 C++ 初始化实践 %}
* {% post_link GAS-Part2-Dash-Decoupling 第二章：冲刺技能实现、Task 运用与表现解耦 %}
* {% post_link GAS-Part3-Attribute-UI 第三章：属性集构建与 UI 解耦：消耗、冷却与动态技能框 %}
* {% post_link GAS-Part4-TargetData-Damage 第四章：TargetData 目标提取与伤害输出闭环 %}
* {% post_link GAS-Part5-Plugin 第五章：框架剥离与独立插件打包实战 %}