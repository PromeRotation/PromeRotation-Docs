# 战斗角色扩展 BattleCharaExtensions

命名空间：`PromeRotation.Extensions`

为 `IBattleChara` 提供常用的状态、位置与攻击目标查询。

## HasStatus

方法：

```csharp
battleChara.HasStatus(statusId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `statusId` | `uint` | 状态效果 ID。 |

作用：判断战斗角色是否拥有指定状态效果。

返回值：`bool`。存在指定状态效果时返回 `true`，否则返回 `false`。

## GetStatusStackCount

方法：

```csharp
battleChara.GetStatusStackCount(statusId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `statusId` | `uint` | 状态效果 ID。 |

作用：获取指定状态效果的层数。

返回值：`int`。返回状态层数；状态不存在时返回 `0`。

## GetStatusLeftTime

方法：

```csharp
battleChara.GetStatusLeftTime(statusId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `statusId` | `uint` | 状态效果 ID。 |

作用：获取指定状态效果的剩余时间。

返回值：`float`。返回剩余秒数；状态不存在时返回 `0`。

说明：以上三个状态查询由 `StatusHelper` 提供具体实现。

## GetStatusSourceEntityId

方法：

```csharp
battleChara.GetStatusSourceEntityId(statusId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `statusId` | `uint` | 状态效果 ID。 |

作用：获取指定状态效果的施加者实体 ID。

返回值：`uint`。状态不存在时返回 `0`。

说明：等同于 [StatusHelper.GetStatusSourceEntityId](./状态效果-StatusHelper.md#getstatussourceentityid)。

## HasStatusWithSource

方法：

```csharp
battleChara.HasStatusWithSource(statusId, sourceId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `statusId` | `uint` | 状态效果 ID。 |
| `sourceId` | `uint` | 施加者的实体 ID。 |

作用：判断战斗角色是否拥有由指定实体施加的状态效果。

返回值：`bool`。存在匹配状态时返回 `true`，否则返回 `false`。

说明：等同于 [StatusHelper.HasStatusWithSource](./状态效果-StatusHelper.md#hasstatuswithsource)。

## DistanceToMe

方法：

```csharp
battleChara.DistanceToMe()
```

参数含义：无。

作用：计算本地玩家与当前目标之间的目标圈边缘距离。

返回值：`float`。本地玩家或当前目标不可用时返回 `0`。

说明：当前实现不使用调用该方法的 `battleChara`，而是读取 `Core.Me` 与 `Core.Target`。目标圈重叠时可能返回负数。

## PositionXY

方法：

```csharp
battleChara.PositionXY()
```

参数含义：无。

作用：获取战斗角色在游戏平面上的位置。

返回值：`Vector2`。`X` 对应三维坐标的 `X`，`Y` 对应三维坐标的 `Z`。

## IsPlayer

方法：

```csharp
battleChara.IsPlayer()
```

参数含义：无。

作用：判断战斗角色是否为玩家角色。

返回值：`bool`。是玩家角色时返回 `true`，否则返回 `false`。

## IsEnemy

方法：

```csharp
battleChara.IsEnemy()
```

参数含义：无。

作用：判断战斗角色是否可作为攻击技能的目标。

返回值：`bool`。可作为攻击目标时返回 `true`；对象为 `null` 或不可攻击时返回 `false`。
