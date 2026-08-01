# 状态效果 StatusHelper

命名空间：`PromeRotation.Helpers`

用于查询战斗角色身上的状态效果。

## HasStatus

方法：

```csharp
StatusHelper.HasStatus(battleChara, statusId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `battleChara` | `IBattleChara` | 要查询的战斗角色。 |
| `statusId` | `uint` | 状态效果 ID。 |

作用：判断战斗角色是否拥有指定状态效果。

返回值：`bool`。存在指定状态效果时返回 `true`，否则返回 `false`。

## GetStatusStack

方法：

```csharp
StatusHelper.GetStatusStack(battleChara, statusId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `battleChara` | `IBattleChara` | 要查询的战斗角色。 |
| `statusId` | `uint` | 状态效果 ID。 |

作用：获取指定状态效果的层数。

返回值：`int`。返回状态层数；状态不存在时返回 `0`。

## GetStatusLeftTime

方法：

```csharp
StatusHelper.GetStatusLeftTime(battleChara, statusId)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `battleChara` | `IBattleChara` | 要查询的战斗角色。 |
| `statusId` | `uint` | 状态效果 ID。 |

作用：获取指定状态效果的剩余时间。

返回值：`float`。返回剩余秒数；状态不存在时返回 `0`。

说明：负数剩余时间会转换为对应的正数。

战斗角色也可通过[战斗角色扩展 BattleCharaExtensions](./战斗角色扩展-BattleCharaExtensions.md)调用这些查询。
