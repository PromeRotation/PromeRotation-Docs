# 队伍优先级 PartyPriority

命名空间：`PromeRotation.Helpers`、`PromeRotation.Managers.PartyPriority`

用于按固定职能位置读取或设置小队优先级顺序。

## PartyPrioritySlot

| 值 | 含义 |
| --- | --- |
| `MT` | MT |
| `ST` | ST |
| `H1` | H1 |
| `H2` | H2 |
| `D1` | D1 |
| `D2` | D2 |
| `D3` | D3 |
| `D4` | D4 |

四人队使用 `MT`、`H1`、`D1`、`D2`；八人队使用全部位置。

## GetSortedPartyMember

方法：

```csharp
PartyHelper.GetSortedPartyMember(slot)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `slot` | `PartyPrioritySlot` | 要查询的职能位置。 |

作用：获取队伍优先级顺序中指定位置的成员。

返回值：`IBattleChara?`。位置不存在或没有对应成员时返回 `null`。

## SetSortedParty

方法：

```csharp
PartyPriorityManager.SetSortedParty(party)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `party` | `IReadOnlyList<IBattleChara>` | 按目标优先级排列的小队成员。 |

作用：使用传入顺序替换当前队伍优先级顺序。

返回值：无。

说明：此方法修改宿主共享的队伍优先级。完整排序结果通过 `PartyHelper.GetSortedParty()` 获取。
