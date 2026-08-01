# 小队信息 PartyHelper

命名空间：`PromeRotation.Helpers`

用于获取小队成员并查询成员数据。

## GetParty

方法：

```csharp
PartyHelper.GetParty()
```

作用：按UI中的小队顺序获取当前全部小队成员，包括本地玩家。

返回值：`List<IBattleChara>`。无法读取时返回空列表。

说明：ARR回放期间会自动改用小队 UI 数据，与UI性质相同，会自动调用两个，使用本方法即可。

## GetSortedParty

方法：

```csharp
PartyHelper.GetSortedParty()
```

参数含义：无。

作用：获取经过队伍优先级系统排序的全部小队成员。

返回值：`IReadOnlyList<IBattleChara>`。返回当前排序结果。

## GetPartyMember

方法：

```csharp
PartyHelper.GetPartyMember(index)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `index` | `int` | 小队索引。`0` 为本地玩家，`1` 至 `7` 为第二至第八位。 |

作用：按小队顺序获取指定成员。

返回值：`IBattleChara?`。索引越界或成员不存在时返回 `null`。

## GetTargetTypeByIndex

方法：

```csharp
PartyHelper.GetTargetTypeByIndex(index)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `index` | `int` | 小队索引。 |

作用：将小队索引转换为动作目标类型。

返回值：`ActionTargetType`。索引 `0` 返回 `Self`，索引 `1` 至 `7` 返回对应小队位置；其他索引返回 `Self`。

## GetHpPercent

方法：

```csharp
PartyHelper.GetHpPercent(battleChara)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `battleChara` | `IBattleChara?` | 要查询的战斗角色。 |

作用：获取战斗角色的当前生命值百分比。

返回值：`float`。范围为 `0` 至 `100`；对象不可用或最大生命值为 `0` 时返回 `0`。

## GetMpPercent

方法：

```csharp
PartyHelper.GetMpPercent(battleChara)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `battleChara` | `IBattleChara?` | 要查询的战斗角色。 |

作用：获取玩家角色的当前魔力值百分比。

返回值：`float`。范围为 `0` 至 `100`；对象不是玩家角色或最大魔力值为 `0` 时返回 `0`。

## GetPartyMemberHpPercent

方法：

```csharp
PartyHelper.GetPartyMemberHpPercent(index)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `index` | `int` | 小队索引。 |

作用：获取指定小队成员的生命值百分比。

返回值：`float`。范围为 `0` 至 `100`；成员不存在时返回 `0`。

## GetPartyMemberMpPercent

方法：

```csharp
PartyHelper.GetPartyMemberMpPercent(index)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `index` | `int` | 小队索引。 |

作用：获取指定小队成员的魔力值百分比。

返回值：`float`。范围为 `0` 至 `100`；成员不存在时返回 `0`。

## FindPartyMemberWithStatus

方法：

```csharp
PartyHelper.FindPartyMemberWithStatus(statusId, includeSelf)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `statusId` | `uint` | 状态效果 ID。 |
| `includeSelf` | `bool` | 是否包含本地玩家。可省略；默认为 `true`。 |

作用：按小队顺序查找第一个拥有指定状态的成员，并返回其动作目标类型。

返回值：`ActionTargetType?`。没有匹配成员时返回 `null`。

## FindPartyMemberCharWithStatus

方法：

```csharp
PartyHelper.FindPartyMemberCharWithStatus(statusId, includeSelf)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `statusId` | `uint` | 状态效果 ID。 |
| `includeSelf` | `bool` | 是否包含本地玩家。可省略；默认为 `true`。 |

作用：按小队顺序查找第一个拥有指定状态的成员。

返回值：`IBattleChara?`。没有匹配成员时返回 `null`。

## GetLowestHpPartyMember

方法：

```csharp
PartyHelper.GetLowestHpPartyMember()
```

参数含义：无。

作用：获取除本地玩家外生命值百分比最低的存活小队成员。

返回值：`ActionTargetType`。队伍中只有本地玩家时返回 `Self`。

说明：队伍存在其他成员但没有存活队友时，当前实现返回 `PartyMember2`。
