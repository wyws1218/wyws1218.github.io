---
title: 🤖 基于马尔可夫算法的 AI 预测敌人
date: 2026-01-01 00:00:00
tags: [UE5, C++, 游戏开发, AI, GAS, 作品集]
categories: [UE 技术, AI]
---

## 项目背景

传统魂类 Boss 的智能，本质是一张写死的"距离—状态"查表：玩家喝药它就丢火球，玩家举剑它就举盾。这种 0 帧读指令式的条件反射，缺乏单局内的短期记忆，玩家一旦摸清固定规律，就能靠背板无伤"逃课"。

这个项目想让 Boss 换一种活法：**在短时间内实时统计玩家的出招习惯，预判你"即将做"的动作，并在你的前摇还没开始时就提前打出克制**。核心是用一阶马尔可夫转移矩阵记录连招频次，靠 GAS 的 GameplayTag 采集数据，用嵌套 TMap 把"习惯学习"降维成 O(1) 的查表操作，全程白盒可控，不依赖神经网络，也不吃 CPU。

![](/images/markov-demo.jpg)

<center>图 1： 魂类 AI RPG 核心系统界面原型（Demo Boss：霜巨人）</center>

## 1. 为什么是马尔可夫，而不是神经网络

要让 AI 变聪明，学术界的第一反应是深度强化学习。但实时动作游戏要求 60 帧运行，每帧逻辑只有约 16 毫秒，挂一个神经网络不仅算力开销大、会掉帧，而且它是个"黑盒"——设计师根本无法控制 Boss 突然做出什么诡异举动，破坏关卡体验。

马尔可夫假设给了一条更轻的路：**当前状态只取决于有限个先前状态**。放到战斗里，就是"玩家下一招"只和"玩家上一招"强相关。我用有限的玩家输入信息，去预测下一次最可能的输入，得到的是一个粗略但足够用的概率近似——代价极低，却刚好够 Boss 看穿你的连招惯性。这也正是格斗游戏（如《杀手本能》的 Shadow AI）采用 N-Gram / 马尔可夫矩阵统计玩家习惯的思路。

## 2. 数据采集：用 GameplayTag 定义攻防词汇表

预测的前提是把"动作"变成机器能统计的离散符号。我没有去监听底层按键，而是直接复用 GAS 的 GameplayTag——玩家和 Boss 的每一个战斗动作都对应一个标签。

![](/images/markov-tags.png)

<center>图 2： Combat 标签层级：Player 四招与 Boss 四个克制招一一对应</center>

标签分成两组，构成严格的克制关系：

| 玩家动作 | Boss 克制 | 备注 |
| --- | --- | --- |
| `Combat.Player.Light` (轻击) | `Combat.Boss.CounterLight` (格挡+怒吼) | 先只做反击一套 |
| `Combat.Player.Heavy` (重击) | `Combat.Boss.CounterHeavy` (闪避+平A反击) | Boss 根据预测反击 |
| `Combat.Player.Dodge` (闪避) | `Combat.Boss.CounterDodge` (追踪技能) | 预测成功玩家扣血 |
| `Combat.Player.Block` (格挡) | `Combat.Boss.CounterBlock` (突刺，抓前摇) | 扣血只为展示预测算法 |

因为原项目本身就是 GAS 架构，玩家的轻击、重击等操作本来就会激活对应的 Gameplay Ability。我只需要在 GA 激活的那一瞬间，把动作标签广播给 Boss 身上的"大脑"，让它记一笔。

![](/images/markov-broadcast.png)

<center>图 3： 玩家 GA 蓝图：激活能力的瞬间调用 RecordPlayerAction，把 Combat.Player.Light 传给 Boss</center>

## 3. 核心：O(1) 的二维转移矩阵

Boss 的"大脑"是一个继承自 `UActorComponent` 的 C++ 组件 `UCombatPredictor`。它的核心数据结构是一个嵌套 TMap——外层 Key 是"玩家上一个动作"，内层 Key 是"玩家下一个动作"，内层 Value 是这个连招组合出现的次数。

```cpp
UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class ARPG_API UCombatPredictor : public UActorComponent
{
    GENERATED_BODY()

public:
    /*
     * O(1) 嵌套哈希矩阵。
     * 外层 Key: 玩家上一个动作
     * 内层 Key: 玩家下一个动作
     * 内层 Value: 该连招出现的次数
     */
    TMap<FGameplayTag, TMap<FGameplayTag, int32>> TransitionMatrix;

    /* 记录玩家上一次使用的动作，作为下一次预测查表时的依据 */
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "AI Brain")
    FGameplayTag LastPlayerAction;
};
```

选 TMap 的理由很实在：它底层是哈希表，寻址复杂度是 O(1)。玩家每打出一招，我只要 `FindOrAdd` 两层、给计数 +1 即可，完全不需要遍历历史序列。这就是"把复杂的习惯学习降维成查表"的关键——极低算力下拿到媲美高级 AI 的拟真决策，而且过程 100% 结构化、可断点调试。

**记录玩家动作**：只有当"上一招"和"当前招"都合法时才构成一个连招组合（开局第一招没有前驱，跳过），然后累加频次并更新指针。

```cpp
void UCombatPredictor::RecordPlayerAction(FGameplayTag NewAction)
{
    // 确保这不是开局第一招（第一招没有"上一个动作"，无法构成连招组合）
    if (LastPlayerAction.IsValid() && NewAction.IsValid())
    {
        // O(1) 寻址：取出"上一个动作"对应的内层字典
        TMap<FGameplayTag, int32>& InnerMap = TransitionMatrix.FindOrAdd(LastPlayerAction);
        // O(1) 寻址：把"当前动作"的频次 +1
        int32& Count = InnerMap.FindOrAdd(NewAction);
        Count++;
    }

    // 更新"上一个动作"指针，为下次查表做准备
    LastPlayerAction = NewAction;
    bHasUnprocessedAction = true;   // 标记有未处理的新情报（见第 5 节）
}
```

## 4. 预测与克制：查表取最高频，决策树下发

Boss 需要出手时调用 `GetCounterAction`：抽出"玩家刚打出的那一招"对应的历史统计表，遍历内层字典（最多 4 个状态，开销近乎为 0）找出频次最高的那一招，就是预测结果；再经一个决策树映射到对应的克制标签。

```cpp
// O(1) 查表：取出玩家刚打出的动作对应的"历史习惯统计表"
TMap<FGameplayTag, int32>* InnerMap = TransitionMatrix.Find(LastPlayerAction);
if (!InnerMap || InnerMap->Num() == 0) return FGameplayTag::EmptyTag;

// 遍历内层字典，找出频次最高的那一招
FGameplayTag PredictedAction = FGameplayTag::EmptyTag;
int32 MaxCount = -1;
for (const TPair<FGameplayTag, int32>& Pair : *InnerMap)
{
    if (Pair.Value > MaxCount) { MaxCount = Pair.Value; PredictedAction = Pair.Key; }
}

// 决策树：根据预测动作下发克制标签
FGameplayTag BossCounter = FGameplayTag::EmptyTag;
if (PredictedAction.MatchesTag(FGameplayTag::RequestGameplayTag("Combat.Player.Light")))
    BossCounter = FGameplayTag::RequestGameplayTag("Combat.Boss.CounterLight");
// ... Heavy / Dodge / Block 同理
```

举个具体例子：玩家刚打出第一招 `Light`，AI 立刻查矩阵中 `[Light]` 转移到下一招的概率。如果算出 `[Light] -> [Heavy]` 高达 90%，那么在 Light 刚结束、Heavy 前摇还没开始的瞬间，Boss 就抢先启动克制 Heavy 的动作——这就是"看穿意图"，而不是等你打出来再反应。

这套预测大脑挂在 Boss 蓝图的组件列表里，和 ASC、Motion Warping 等组件平级。

![](/images/markov-component.png)

<center>图 4： CombatPredictor 组件挂载在 Boss 身上</center>

预测出的克制标签，在 Boss 蓝图的反击函数里被翻译成具体技能：

![](/images/markov-counter-bp.png)

<center>图 5： Boss 蓝图 ExecuteCounterAttack：按克制标签分发到对应反击招式</center>

## 5. 让预测接管行为树：自定义 BTTask

预测算出来了，还得让 Boss 真的动起来。我写了一个自定义任务节点 `BTTask_PredictiveCounter`，挂在行为树"近战攻击"分支下，与原有的 Melee Attack 序列并列。

![](/images/markov-bt.png)

<center>图 6： 行为树：自建的 BTTask_PredictiveCounter 节点</center>

节点内部逻辑很直接：拿到被控 Pawn，Cast 成 Boss，调用前面的 `GetCounterAction` / 反击函数，成功就 `Finish Execute(Success)`；如果转换失败则走另一条 Finish 分支，防止节点卡死。

![](/images/markov-bttask.png)

<center>图 7： BTTask_PredictiveCounter 事件图：Cast Boss → 执行反击 → Finish Execute</center>

## 6. 踩坑：Boss 对同一个动作无限反击

联调时出现一个典型 bug：**只要玩家停手不出招，Boss 就对着最后那一招无限循环反击**。

排查后定位到冲突本质：`LastPlayerAction` 这个指针需要被保留下来供后续数学计算，但行为树是循环 tick 的——它跑完一次反击任务，Boss 还在近战距离内，下一秒又跑回来问大脑"玩家最后做了什么"。因为玩家没再出招，指针依然停留在 `Combat.Player.Light`，于是 Boss 觉得"还没反击够"，又放了一次。

解决办法借鉴微信消息的"已读回执"（Intel Freshness）：情报一直在（保留给马尔可夫计算），但加一个"未读小红点"标志位 `bHasUnprocessedAction`。收到新动作时置 `true`，反击处理一次后立刻置 `false`。

```cpp
FGameplayTag UCombatPredictor::GetCounterAction()
{
    // 情报已被处理过（旧的），直接拒绝反击
    if (!bHasUnprocessedAction) return FGameplayTag::EmptyTag;
    // 新鲜情报，立刻"已读"，消费掉这次情报
    bHasUnprocessedAction = false;
    // ... 后续正常查表预测
}
```

这样一来，Boss 每次玩家出招只反击一次，指针依旧保留给下一次连招统计，两个需求互不打架。

## 7. 系统架构总览

整套系统分三层：表现与交互层跑在 UE5 客户端（GAS / AIModule / Motion Warping），决策与预测层是核心——AI 大脑组件负责监听标签、更新 TMap 矩阵，决策核心负责查表选出克制技能，再由 GAS 集成服务把克制标签通过打断行为树的方式下发给 Boss 执行。

![](/images/markov-arch.png)

<center>图 8： 系统架构：表现层 / 决策预测层 / GAS 集成服务</center>

Demo 的战斗场景基于 SketchFab 地形资源搭建，重新烘焙了导航网格，保证 Boss 的 EQS 游走和近战逼近正常工作。

![](/images/markov-arena.png)

<center>图 9： 战斗场景与导航网格</center>

## 8. 创新点与收敛表现

和传统方案比，这套 AI 的差异在三处：

1. **零态预测与习惯抓取**：传统魂类先手基于静态的距离/几何判断，这里引入玩家习惯统计，做的是带心理博弈的动态预判。
2. **序列转移的未来截断**：传统后手是 0 帧读指令，这里靠概率（如 `Light→Heavy` 达 70%）在玩家前置动作刚结束时就提前反击，实现真正的"看穿意图"。
3. **极短的收敛时间**：格斗游戏的影子 AI 往往需要多局训练，而这套矩阵能在单局内动态收敛权重，玩家几个回合就能直观感受到 Boss 在"进化"。

同时系统保留了降级容错：如果玩家毫无逻辑地乱按，各分支概率会趋于平均、置信度过低，此时自动退回传统的"查表条件反射"，避免被噪声数据带偏。
