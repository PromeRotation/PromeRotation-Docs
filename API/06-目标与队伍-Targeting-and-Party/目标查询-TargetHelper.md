# 目标查询 TargetHelper

命名空间：`PromeRotation.Helpers`

用于查询目标、敌人数量、AOE 命中数量和 NPC。

## GetTargetPositional

方法：

```csharp
TargetHelper.GetTargetPositional()
```

参数含义：无。

作用：判断本地玩家位于当前目标的前方、侧面或后方。

返回值：`Positional`。没有本地玩家或当前目标时返回 `Positional.None`。

## HasPositionalRequirement

方法：

```csharp
TargetHelper.HasPositionalRequirement(battleChara)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `battleChara` | `IBattleChara` | 要检查的战斗角色。 |

作用：判断敌人是否存在身位要求。

返回值：`bool`。存在身位要求时返回 `true`，否则返回 `false`。

说明：非敌对目标、全方向目标或拥有状态 `3808` 的目标返回 `false`。

## EnemyIn5m

方法：

```csharp
TargetHelper.EnemyIn5m()
```

参数含义：无。

作用：统计本地玩家周围 5 米内可攻击且存活的敌人数量。

返回值：`uint`。本地玩家不可用时返回 `0`。

## EnemyInRange

方法：

```csharp
TargetHelper.EnemyInRange(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的目标圈边缘距离，单位为米。 |

作用：统计指定距离内可攻击且存活的敌人数量。

返回值：`uint`。本地玩家不可用时返回 `0`。

## EnemyInRangeTarget

方法：

```csharp
TargetHelper.EnemyInRangeTarget(target, range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `target` | `IBattleChara?` | 距离计算的中心目标。 |
| `range` | `float` | 与中心目标的目标圈边缘距离，单位为米。 |

作用：统计指定目标周围可攻击且存活的敌人数量。

返回值：`uint`。中心目标为 `null` 时返回 `0`。

## GetMostCanTargetObjects

方法：

```csharp
TargetHelper.GetMostCanTargetObjects(actionId, count, angle)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `actionId` | `uint` | AOE 技能 ID。 |
| `count` | `int` | 要求命中的最少敌人数。 |
| `angle` | `float` | 扇形角度，单位为度。可省略；默认为 `120`。 |

作用：根据技能的圆形、矩形或扇形范围选择命中敌人最多的目标。

返回值：`IBattleChara?`。没有满足命中数量的目标时返回 `null`。

说明：不会计算玩家角色、死亡目标、不可攻击目标或具有无敌状态的目标。

## GetEnemyCountInsideCircle

方法：

```csharp
TargetHelper.GetEnemyCountInsideCircle(target, radius)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `target` | `IBattleChara` | 圆形范围的中心目标。 |
| `radius` | `float` | 圆形半径，单位为米。 |

作用：统计圆形范围内的敌人数量。

返回值：`int`。返回符合条件的敌人数量。

## GetEnemyCountInsideRect

方法：

```csharp
TargetHelper.GetEnemyCountInsideRect(me, target, length, width)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `me` | `IBattleChara` | 矩形范围的起点。 |
| `target` | `IBattleChara` | 矩形范围的朝向目标。 |
| `length` | `float` | 矩形长度，单位为米。 |
| `width` | `float` | 矩形宽度，单位为米。 |

作用：统计从起点朝目标方向延伸的矩形范围内的敌人数量。

返回值：`int`。返回符合条件的敌人数量。

## GetEnemyCountInsideSector

方法：

```csharp
TargetHelper.GetEnemyCountInsideSector(me, target, sectorRadius, sectorAngle)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `me` | `IBattleChara` | 扇形范围的起点。 |
| `target` | `IBattleChara` | 扇形范围的朝向目标。 |
| `sectorRadius` | `float` | 扇形半径，单位为米。 |
| `sectorAngle` | `float` | 扇形角度，单位为度。 |

作用：统计从起点朝目标方向展开的扇形范围内的敌人数量。

返回值：`int`。返回符合条件的敌人数量。

## TargetHasInvincibilityStatus

方法：

```csharp
TargetHelper.TargetHasInvincibilityStatus(target)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `target` | `IBattleChara` | 要检查的目标。 |

作用：判断目标是否拥有无敌状态。

返回值：`bool`。拥有任一无敌状态时返回 `true`，否则返回 `false`。

## FindNpcByBaseId

方法：

```csharp
TargetHelper.FindNpcByBaseId(baseId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `baseId` | `uint` | NPC BaseId。 |

作用：查找第一个具有指定 BaseId 的战斗角色。

返回值：`IBattleChara?`。没有匹配对象时返回 `null`。

说明：不要求对象可选中；BaseId 也等同于旧版的 DataId。

## FindTargetableNpc

方法：

```csharp
TargetHelper.FindTargetableNpc(dataId, name, nearest)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `dataId` | `uint` | 要匹配的 ID。传入 `0` 时不限制 ID。 |
| `name` | `string?` | NPC 的完整名称。不区分大小写；留空时不限制名称。 |
| `nearest` | `bool` | 是否返回距离本地玩家最近的匹配对象。 |

作用：按 ID 和名称查找可选中的 NPC。

返回值：`IBattleChara?`。没有匹配对象时返回 `null`。

说明：当前实现将 `dataId` 与 NPC 的 `BaseId` 比较；旧版的 DataId 等同于 BaseId。

## FindNpcsByBaseId

方法：

```csharp
TargetHelper.FindNpcsByBaseId(baseId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `baseId` | `uint` | NPC BaseId。 |

作用：查找全部具有指定 BaseId 的战斗角色。

返回值：`List<IBattleChara>`。没有匹配对象时返回空列表。

说明：不要求对象可选中；BaseId 也等同于旧版的 DataId。。

## IsAllBossUntargetable

方法：

```csharp
TargetHelper.IsAllBossUntargetable()
```

参数含义：无。

作用：判断本地玩家周围 100 米内是否不存在可攻击且存活的敌人。

返回值：`bool`。不存在符合条件的敌人时返回 `true`，否则返回 `false`。

说明：当前实现不区分普通敌人和首领。

## Positional

| 值 | 含义 |
| --- | --- |
| `None` | 无法确定身位。 |
| `Front` | 目标前方。 |
| `Flank` | 目标侧面。 |
| `Rear` | 目标后方。 |

返回目标相对坐标的 `GetStancePoint()` 与 `GetStancePoint_Center()` 属于空间与移动 API。
