# 游戏状态 GameData

命名空间：`PromeRotation.Core`

用于读取游戏环境、战斗状态与移动状态。

## 属性

| 属性 | 类型 | 含义 |
| --- | --- | --- |
| `GameData.WillMove` | `bool` | 是否即将移动。 |
| `GameData.IsBoundByDuty` | `bool` | 是否在副本区域内。 |
| `GameData.TerritoryId` | `uint` | 当前区域 ID。 |
| `GameData.Weather` | `uint` | 当前天气 ID。 |

## IsInCombat

方法：

```csharp
GameData.IsInCombat()
```

参数含义：无。

作用：判断本地玩家是否处于战斗状态。

返回值：`bool`。处于战斗状态时返回 `true`，否则返回 `false`。

## IsInPvPArea

方法：

```csharp
GameData.IsInPvPArea()
```

参数含义：无。

作用：判断当前是否处于 PvP 区域。

返回值：`bool`。处于 PvP 区域时返回 `true`，否则返回 `false`。

## IsTargetInCombat

方法：

```csharp
GameData.IsTargetInCombat()
```

参数含义：无。

作用：判断当前目标自身是否处于战斗状态。

返回值：`bool`。目标处于战斗状态时返回 `true`；没有目标或无法读取时返回 `false`。

说明：读取的是目标的战斗状态，可在本地玩家尚未获得仇恨时判断队友是否已开战。

## IsInBossRoom

方法：

```csharp
GameData.IsInBossRoom()
```

参数含义：无。

作用：判断本地玩家是否处于Boss房。

返回值：`bool`。判断为Boss房时返回 `true`，否则返回 `false`。

说明：此方法为实验性 API，可能出现误判。无法读取状态时返回 `false`。

## GetCountdown

方法：

```csharp
GameData.GetCountdown()
```

参数含义：无。

作用：获取战斗倒计时的剩余时间。

返回值：`float`。返回剩余秒数；没有倒计时或无法读取时返回 `0`。

## TryGetCountdown

方法：

```csharp
GameData.TryGetCountdown(out countdown)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `countdown` | `out float` | 剩余秒数；没有倒计时时为 `0`。 |

作用：尝试读取战斗倒计时。

返回值：`bool`。读取成功时返回 `true`（包括没有倒计时的情况）；底层读取异常时返回 `false`。

说明：需要区分“没有倒计时”和“读取失败”时使用；只需要秒数时用 `GetCountdown()`。

## IsPlayerOccupied

方法：

```csharp
GameData.IsPlayerOccupied()
```

参数含义：无。

作用：判断本地玩家是否处于无法正常执行操作的占用状态。

返回值：`bool`。处于占用状态或本地玩家不可用时返回 `true`，否则返回 `false`。

## GetCurrentMeleeRange

方法：

```csharp
GameData.GetCurrentMeleeRange()
```

参数含义：无。

作用：获取当前近战攻击距离。

返回值：`float`。通常返回 `3`；启用长臂猿时返回 `6`。

## GetCurrentAttackRange

方法：

```csharp
GameData.GetCurrentAttackRange(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 基础攻击距离。 |

作用：根据当前攻击距离设置计算实际攻击距离。

返回值：`float`。未启用长臂猿时返回原值；启用时返回 `range + 3`。

## IsIn120

方法：

```csharp
GameData.IsIn120()
```

参数含义：无。

作用：判断本地玩家是否拥有任一两分钟爆发状态，包括药品状态。

返回值：`bool`。存在匹配状态时返回 `true`，否则返回 `false`。

## IsInPure120

方法：

```csharp
GameData.IsInPure120()
```

参数含义：无。

作用：判断本地玩家是否拥有任一两分钟爆发状态，不包括药品状态。

返回值：`bool`。存在匹配状态时返回 `true`，否则返回 `false`。

## HasEnemyCastingActionInRange

方法：

```csharp
GameData.HasEnemyCastingActionInRange(actionIds, range, remainingMilliseconds)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `actionIds` | `IReadOnlySet<uint>` | 要匹配的敌方咏唱技能 ID。 |
| `range` | `float` | 以本地玩家为中心的目标圈边缘距离。 |
| `remainingMilliseconds` | `int` | 咏唱剩余时间上限，单位为毫秒；小于或等于 `0` 时不限制。 |

作用：判断指定距离内是否有敌人正在咏唱匹配的技能。

返回值：`bool`。存在符合条件的敌方咏唱时返回 `true`，否则返回 `false`。

