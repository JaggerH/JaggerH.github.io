---
title: 套利执行模型：从重复下单 Bug 到 Order Gate 机制
created_time: 2026-02-08T00:00:00.000Z
last_edited_time: 2026-02-08T00:00:00.000Z
tags:
  - LEAN
  - QuantConnect
  - Trading
  - CSharp
  - Architecture
date: '2026-02-08'
layout: single
---

一次 Rebalance 取消订单失败引发的系统性思考：异步市价单的中间态如何导致重复下单，以及如何用一行检查从根源上解决这个问题。

<!--more-->

## 一、问题现场

### ProcessTarget 执行流程

ArbitrageExecutionModel 的核心执行入口是 `ProcessTarget`，采用三层决策树：

```csharp
private void ProcessTarget(AQCAlgorithm algorithm, IArbitragePortfolioTarget target)
{
    if (!ValidateTargetPreconditions(algorithm, target)) return;

    // Rebalance (highest priority)
    var rebalanceDecision = ShouldRebalance(algorithm, target);
    if (rebalanceDecision.ShouldRebalance)
    {
        Rebalance(algorithm, target, rebalanceDecision);
        return;
    }

    // Sweep
    var sweepDecision = ShouldSweep(algorithm, target);
    if (sweepDecision.ShouldSweep)
    {
        Sweep(algorithm, target, sweepDecision);
        return;
    }

    ExecuteRegularOrder(algorithm, target);
}
```

这个方法每 100ms 被执行循环调用一次。在异步市价单（`Asynchronous=true`）模式下，订单发出后不会阻塞等待结果，执行循环继续运行。

### Rebalance 取消 New 订单失败

第一个告警：Rebalance 在运行前会取消跟 Target 相关的订单，执行时报了内部错误——不能取消 `OrderStatus == New` 的订单。

回顾 `PlaceOrder` 的生命周期：

1. 创建订单，`OrderStatus == New`
2. `PlaceOrder` 将订单加到 Transaction 队列
3. 内部校验通过后提交给券商
4. 在券商返回结果之前，`OrderStatus == New`

LEAN 内部判定不能取消 `New` 状态的订单是正确逻辑——这个状态的订单还没有被提交到券商，取消操作没有意义。

### LTCUSDT 重复下单复盘

之前注意到同一状态下存在多笔订单的情况，但不能确认是否有重复下单。直到一次边缘情况确认了这个 bug：

LTCUSDT 交易对的订单接近了结，券商端 Crypto 持仓数量 **0.01**，报错试图卖出 0.02，超过持仓数量，被内拒。复盘流程：

1. `GridPosition` 持仓数量 0.03
2. 生成下单意图，计划从 0.03 → 0.01，下单数量 -0.02
3. 异步提交订单，还没有收到前一笔的返回
4. 再次生成相同的下单意图，从 0.03 → 0.01，下单数量 -0.02
5. 前一笔执行结果返回，第二笔报错：试图裸空，数量超限

### 防御性修复的局限

对上述问题做了防御性修复（commit `cdd5bb341`）：在 `CalculateConstrainedQuantities` 中对 `UnorderedQuantity` 加上 `OpenOrder` 未成交数量的扣减。

```csharp
// Deduct in-flight orders (submitted but not yet filled) to prevent duplicate orders
var (inFlight1, inFlight2) = TradingPairManager.GetOpenOrderRemainQuantities(_algorithm, target);
unorderedQty1 -= inFlight1;
unorderedQty2 -= inFlight2;
```

这个修复只覆盖了 Regular 路径的 `CalculateConstrainedQuantities`，不是系统性解决方案。Rebalance 路径、Sweep 路径都没有类似保护。这是一个**系统性的问题**——无论是 Regular、Rebalance 还是 Sweep，都没有考虑在途订单的检测和监控。

## 二、根因分析：异步市价单的中间态

### PlaceOrder 生命周期

```
[New] ──(HandleSubmitOrderRequest → SetOrder)──> [Submitted] ──(fill event)──> [Filled]
                                                        │
                                                        └─(cancel event)─> [Canceled]
                                                        └─(error)─> [Invalid]
```

关键窗口在 `New → Submitted` 之间：

- **`New`**：订单已创建，`OrderTicket` 已加入 `_openOrders` 集合，但后台处理线程还没有调用 `HandleSubmitOrderRequest`
- 此时 `ticket._order == null`，所以 `ticket.Status` 返回 `OrderStatus.New`
- `New` 状态的订单不能被取消（LEAN 正确拒绝）
- `New` 状态的订单不影响 `GridPosition`（没有 fill event）

### 100ms 执行循环与在途订单的冲突

```
时间线:
T+0ms    Execute → MarketOrder(async=true) → 订单进入队列 [New]
T+1ms    后台线程开始处理 → HandleSubmitOrderRequest
T+5ms    brokerage.PlaceOrder() → 等待 REST 响应
T+100ms  Execute 再次触发 → 读取 GridPosition（未更新）→ 计算相同的下单意图
T+150ms  第一笔 REST 响应回来 → [Submitted] → [Filled]
T+100ms  第二笔订单提交 → 重复下单!
```

在 T+100ms 时，第一笔订单还处于 `New` 或 `Submitted` 状态，`GridPosition` 完全未更新。执行循环基于陈旧的仓位状态计算出相同的下单意图。

### 为什么 New 状态是最危险的窗口

| 状态 | GridPosition 已更新? | 能被取消? | in-flight 扣减有效? |
|------|---------------------|----------|-------------------|
| New | 否 | 否 | 部分（只覆盖 Regular 路径） |
| Submitted | 否 | 是 | 是 |
| Filled | 是 | N/A | N/A |

`New` 状态是所有防御机制的盲区：
- `GridPosition` 未更新，所以 `ShouldRebalance()` 读到的数据不可靠
- 不能取消，所以 Rebalance 的 "先取消后重下" 策略失败
- in-flight 扣减只覆盖了 Regular 路径的 `CalculateConstrainedQuantities`

## 三、Order Gate：一行代码的系统性修复

### 设计：检查 Symbol 级别的 New 订单

在 `ProcessTarget()` 入口处，`ValidateTargetPreconditions` 之后，插入一行检查：

```csharp
private void ProcessTarget(AQCAlgorithm algorithm, IArbitragePortfolioTarget target)
{
    if (!ValidateTargetPreconditions(algorithm, target)) return;

    // Order Gate: block execution while orders are in New status (not yet submitted to brokerage)
    if (algorithm.Transactions.GetOpenOrderTickets(
            t => (t.Symbol == target.Leg1Symbol || t.Symbol == target.Leg2Symbol)
                 && t.Status == OrderStatus.New).Any())
    {
        return;
    }

    // ... Rebalance → Sweep → Regular decision tree unchanged ...
}
```

这个检查在所有路径（Rebalance、Sweep、Regular）之前执行，是真正的系统性修复。

### 为什么只检查 OrderStatus.New

- **`New`** 是订单创建后、提交给券商之前的状态，此时 `GridPosition` 完全未更新
- **`Submitted`** 以后的在途订单已经被 commit `cdd5bb341` 的 in-flight 扣减覆盖
- 只阻塞最危险的窗口，不会过度限制执行频率

### 为什么不需要超时机制

- REST 接口一定会返回结果（成功或失败），`New` 状态不会永久停留
- 定时对账机制保证不漏单
- 如果 REST 请求异常超时，LEAN 的 `BrokerageTransactionHandler` 有内置的超时处理

### 为什么按 Symbol 而不是按 Tag 过滤

同一交易对的不同 Target 的市价单都会影响市场状况（订单簿变化、做市商反应）。Symbol 级别的阻塞更安全，避免多个 Target 同时向同一交易对发单。

### 与 in-flight 扣减的互补关系

| 防护层 | 覆盖范围 | 作用时机 |
|--------|---------|---------|
| Order Gate | 所有路径（Rebalance、Sweep、Regular） | `New` 状态 |
| In-flight 扣减 | Regular 路径的 CalculateConstrainedQuantities | `Submitted` 状态 |

两层防护互补：
1. **Order Gate** 拦截 `New` 状态的窗口期——此时不应该做任何执行决策
2. **In-flight 扣减** 处理 `Submitted` 状态——订单已提交但未成交，通过扣减避免超量下单

### Phase 2 LimitOrder 兼容性

当前只检查 `New` 状态。未来如果引入 LimitOrder：
- LimitOrder 提交后迅速变为 `Submitted`，不会长时间阻塞在 `New`
- 如果需要区分 MarketOrder vs LimitOrder 的阻塞策略，只需增加 `&& t.OrderType == OrderType.Market` 条件

## 四、Phase 2 路线图

Order Gate 解决了当前阶段最紧迫的重复下单问题。下一阶段需要考虑更复杂的执行管理：

### 时间区间管理：限价单超时升级

从双腿 MarketOrder 转向单腿 LimitOrder（甚至双腿 LimitOrder）后，需要管理订单的生命周期：
- 限价单超时未成交 → 升级为市价单（或取消重下）
- 时间区间内的价格变化检测 → 决定是否修改限价
- 部分成交后的策略：等待、补单、或取消

### 双腿误差管理

当前的 Rebalance 机制只处理"一腿成交、另一腿未成交"的情况。更精细的误差管理需要：
- 不仅是慢腿追快腿，还要考虑追单成本
- 双腿成交价格偏离预期时的处理策略
- 误差累积的监控和告警阈值

### 错误追踪机制

当前的错误处理比较被动。需要建立主动的错误追踪：
- 交易所拒单原因分类和计数
- 风控触发的自动降级（减少下单频率或数量）
- FOK 失败的重试策略
- 保证金不足时的仓位调整方案

### `ExecutionState` 重构方向

最终目标是将 `ProcessTarget` 的无状态决策树重构为有状态的执行状态机：

```
IDLE → PLACING → SUBMITTED → PARTIAL_FILL → FILLED
                    │                │
                    └── TIMEOUT ─────┘── REBALANCE
```

每个 Target 维护自己的 `ExecutionState`，执行循环根据状态决定下一步操作，而不是每次都从零开始计算。这将从根本上消除"基于陈旧状态重复计算"的问题。
