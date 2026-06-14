---
title: 【第一章】GAS 底层基建与 C++ 初始化实践
date: 2026-03-23 13:00:00
hidden: true
tags: [UE5, C++, 游戏开发, GAS, 作品集]
categories: [UE 技术, GAS]
---

## 1. 核心模块引入与配置

在 UE5 引擎中勾选启用 Gameplay Abilities 插件是第一步。

![](/images/gasBuild1.png)

<center>图 1： GAS 核心插件</center>

要让项目真正获得 GAS 的底层 C++ 支持，必须在项目的构建文件中进行手动配置。

![](/images/gasBuild2.png)

<center>图 2： 项目结构示意</center>

我在项目的 `Build.cs` 文件中，加入了 GAS 相关的三大核心代码库：`GameplayAbilities`、`GameplayTags` 和 `GameplayTasks`。

![](/images/gasBuild3.png)

<center>图 3： 相关代码</center>

这一行代码相当于给项目"安装"了 GAS 插件的 C++ 底层操作，它为我们后续的代码开发正式引入了技能系统、标签系统和任务系统的支持。
## 2. 角色基类 (WyBaseCharacter) 与网络同步策略

为了方便后期扩展与代码管理，我创建了一个基于 C++ 的基类（`WyCharacterBase`），作为玩家 (Player) 和敌人 (Enemy) 的共同基类，并对其目录结构进行了规范整理。我第一步让基类继承了 AbilitySystem 的接口，如果不继承该接口，后续所有技能和属性无法挂在该角色上。

![](/images/gasBuild4.png)

<center>图 4： 角色基类的接口</center>

在基类的头文件中，我为其手动硬编码挂载了核心的 `AbilitySystemComponent` (ASC)，并专门声明了网络同步模式 (Replication Mode) 的可配置变量。这里使用 Mixed 模式：一是为了同步完整 GE 信息，支持客户端预测，让玩家感觉不到延迟；二是为了只同步极简的 Tags 和 Attributes，节省带宽。

![](/images/gasBuild5.png)

<center>图 5： 在头文件中声明 ASC 与 Mixed 同步模式</center>

## 3. 暴露接口与双端激活

首先我返回了 ASC 这个组件，让系统发现我"硬件"的存在。

![](/images/gasBuild6.png)

<center>图 6： 返回 ASC</center>

然后明确客户端和服务端的身份，服务器在控制角色那一刻先认证挂在角色上的 ASC，客户端收到服务器传来数据那一刻认证。

![](/images/gasBuild7.png)

<center>图 7： 双端激活</center>

## 4. 角色初始化

利用模板给角色一些初始值，比如胶囊体的尺寸、旋转、移动、视角、速度等，完成了角色基类的 GAS 基础配置。

![](/images/gasBuild8.png)

<center>图 8： 初始化配置</center>
