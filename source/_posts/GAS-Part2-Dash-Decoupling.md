---
title: 【第二章】冲刺技能 (Dash) 的实现与解耦
date: 2026-03-24 13:00:00
hidden: true
tags: [UE5, C++, 游戏开发, GAS, 作品集]
categories: [UE 技术, GAS]
---

## 1. 创建冲刺技能 (GA) 与核心配置

为了将冲刺逻辑与角色实体解耦，首先需要创建一个专门负责处理冲刺逻辑的技能容器。这里我新建一个继承自 `GameplayAbility` 的蓝图类，将其命名为 `GA_Dash`。

![](/images/dash1.png)

<center>图 1： 创建 GA_Dash 蓝图类</center>

将技能独立为单独的 GA 资产后，我不仅实现了代码的解耦，还可以通过配置参数来决定该技能在多人网络环境下的运行表现。这也是 GAS 框架最强大的优势之一。

![](/images/dash2.png)

<center>图 2： GA_Dash 的核心标签与网络参数配置</center>

在 `GA_Dash` 的类默认设置 (Class Defaults) 中，有三个极其关键的配置，这也是多人动作游戏开发中的核心考点：

1. Ability Tags (标签驱动)：赋予其 `GameplayAbility.Movement.Dash` 标签。GAS 倡导"标签驱动"思想，后续我们可以通过判断角色身上是否有该标签，来替代传统的布尔值判断，彻底告别硬编码。
2. Instancing Policy (实例化策略)：设置为 `Instanced Per Actor` (每个 Actor 实例化)。因为冲刺技能在释放过程中需要保存一些内部状态，非实例化策略无法存储状态，因此必须为每个角色单独实例化一个 GA 对象。
3. Net Execution Policy (网络执行策略)：设置为 `Local Predicted` (本地预测)。这是多人动作游戏保证手感的灵魂。它允许客户端立刻在本地执行冲刺表现，而不需要等待服务器的 RPC 确认，实现客户端的"零延迟"体验。
## 2. 冲刺事件图表：提交逻辑与异步 Task

进入 `GA_Dash` 的事件图表，首先需要处理技能的触发。在 GAS 框架中，一个规范的技能生命周期通常包含：激活 (Activate)、提交消耗 (Cost)、执行异步行为 (Ability Task) 以及结束技能 (End Ability)。

![](/images/dash3.png)

<center>图 3： GA_Dash 的完整事件图表逻辑</center>

这张图表包含了 GAS 动作开发中的几个核心规范：

1. 消耗与冷却的精准控制：技能通过 `Event ActivateAbility` 激活后，第一时间调用 `CommitAbilityCost` 扣除玩家体力（Cost）。而冷却（Cooldown）则挂载在位移结束的分支上通过 `CommitAbilityCooldown` 提交。
2. 利用 Ability Task 处理异步位移：冲刺是一个持续的过程。这里摒弃了传统的 Timeline 或 Tick 位移，使用了 GAS 内置的异步任务节点 `Apply Root Motion Constant Force`。它能给角色根组件施加一个持续 0.3 秒、强度为 2000 的推力，并在结束时利用 `Clamp Velocity on Finish` 限制最终速度，保证冲刺结束时的手感平滑，不会因为惯性滑步。
3. 表现与逻辑的彻底解耦：`Apply Root Motion` 节点右上方的异步触发引脚，连接了 `Add GameplayCue To Owner` 节点。我没有在这里生成任何粒子特效或播放音效，而是仅仅发送了一个 `Gameplaycue.Dash.Fast` 的标签给系统。所有相关的视听表现，都交由客户端的 Gameplay Cue (GC) 去监听该标签并独立执行。这完美贯彻了"逻辑与表现分离"的 MVC 架构思想。
4. 规范闭环：当 Task 的 `On Finish` (任务完成) 触发时，必须调用 `End Ability` 来清理状态机，规范地结束该技能。
## 3. 获取物理化身：蓝图与 C++ 底层联动

在冲刺技能中，Task 节点需要知道应该对哪个目标施加物理推力。为了获取正确的方向，我使用了 `Get Avatar Actor from Actor Info` 节点，从系统信息表中把角色的物理化身提取出来单独使用。

![](/images/dash4.png)

<center>图 4： 提取物理化身节点</center>

这个节点之所以能成功获取到当前的实体角色，完全依赖于第一章角色初始化阶段我在 C++ 中写入的底层配置。

![](/images/dash5.png)

<center>图 5： 相关代码</center>

在 GAS 系统的底层架构里，一个角色被分成了两个核心部分：

1. Owner: 负责管理数据的"大脑"，比如判断技能冷却好了没、蓝量够不够。
2. Avatar: 负责在游戏里跑跳、播放动画、被物理碰撞推着走的"肉体"。

我在 C++ 阶段分别在服务端的 `PossessedBy` 和客户端的 `OnRep_PlayerState` 中调用了 `InitAbilityActorInfo(this, this)` 函数。传入的两个 `this` 就相当于在系统双端登记表上写明，我既是管理数据的大脑，也是负责挨打和跑跳的肉体。

冲刺的 Task 节点本身没有目标识别能力，它不知道要推谁。因此它必须通过这个蓝图节点去查阅 C++ 写的登记表，拿到登记好的肉体角色，再获取它的前向向量，最终把推力精准地作用在这个肉体上。如果当时没有写那段 C++ 代码做初始化，蓝图节点就会查不到肉体，冲刺技能便会因为找不到受力对象而当场报错失效。
## 4. 技能的赋予与激活

在 GAS 框架中，技能的使用遵循一个严格的两步流程：先赋予，再激活。我首先在角色初始化完成后的逻辑中，调用了 Give Ability 节点，将 GA_Dash 这个类注册到角色的 Ability System Component 中。这相当于把技能真正装进了角色的技能栏里。

![](/images/dash6.png)

<center>图 6： 赋予角色冲刺能力</center>

技能装备好之后，我就可以在角色的事件图表中处理玩家的输入逻辑了。

![](/images/dash7.png)

<center>图 7： 通过按键激活冲刺技能</center>

当玩家按下 Q 键时，我调用了 Try Activate Ability by Class 节点，并指定了之前赋予的 GA_Dash 类。此时系统会自动去角色的技能栏里查找是否拥有这个技能，如果有且满足冷却和消耗条件，就会真正触发前面写的冲刺事件图表逻辑。节点上的 Allow Remote Activation 选项被勾选，这保证了客户端按下按键时，可以通过网络通知服务器同步激活该技能。
## 5. 表现解耦：Gameplay Cue 接管视听效果

在传统的开发流程里，播放特效和音效的代码通常和位移逻辑混杂在一起。但在本套 GAS 框架中，`GC_Dash` 里只需要重写一个核心函数即可完成所有工作。

其核心机制是标签监听。我首先在它的详情面板中，将这个蓝图类与 `Gameplaycue.Dash.Fast` 标签进行了绑定。

![](/images/dash8.png)

<center>图 8： 绑定对应的 Gameplay Cue 标签</center>

回忆一下前面在冲刺的 GA 蓝图里，我在 Task 节点上方单独调用了 Add GameplayCue 节点来发送这个标签，并在技能结束时自动移除它。这就意味着，系统会自动根据这个标签的添加和移除，来触发 `GC_Dash` 里的 `HandleGameplayCue` 函数。

![](/images/dash9.png)

<center>图 9： HandleGameplayCue 的完整表现逻辑</center>

在这个核心函数中，我使用了 `Switch on EGameplayCueEvent` 节点。这个节点极其强大，它能根据标签所处的生命周期，将表现逻辑完美拆分：

1. On Active 阶段：当冲刺刚开始，标签被添加时触发。我在这里生成了一个附着在根组件上的拖尾粒子特效，播放了风声破空的音效。同时，为了体现冲刺极快的视觉冲击力，我获取了角色的 Mesh 网格体并将其隐藏，模拟出一种瞬间闪现的视觉感。
2. Removed 阶段：当 Task 任务完成，GA 调用 End Ability 导致标签被移除时触发。此时我播放了技能结束的音效，在角色当前落地位置生成了一个结尾爆点特效，并将隐藏的角色模型重新显示出来。

通过这种方式，冲刺的物理逻辑和视听表现彻底分家。将来美术同学如果需要替换特效或调整表现逻辑，只需要打开这个 GC 蓝图即可，完全没有破坏核心技能逻辑的风险。这种高度解耦的架构，也是大型多人游戏项目必备的底层设计。
