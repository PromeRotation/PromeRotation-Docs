# 治疗目标 HealerHelper

命名空间：`PromeRotation.Helpers`

用于从小队中查询治疗或复活目标。距离均按目标圈边缘计算。

## GetDeathDurationSeconds

方法：

```csharp
HealerHelper.GetDeathDurationSeconds(member)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `member` | `IBattleChara` | 要查询的小队成员。 |

作用：获取成员当前持续死亡的时间。

返回值：`float`。返回死亡秒数；成员没有死亡记录时返回 `0`。

## ForceUpdateDeathRecords

方法：

```csharp
HealerHelper.ForceUpdateDeathRecords()
```

参数含义：无。

作用：立即更新小队成员的死亡时间记录。

返回值：无。

## ClearAllDeathRecords

方法：

```csharp
HealerHelper.ClearAllDeathRecords()
```

参数含义：无。

作用：清空全部死亡时间记录。

返回值：无。

## IsTank

方法：

```csharp
HealerHelper.IsTank(job)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `job` | `Job` | 要判断的职业。 |

作用：判断职业是否为坦克。

返回值：`bool`。职业为骑士、战士、暗黑骑士或绝枪战士时返回 `true`。

## IsHealer

方法：

```csharp
HealerHelper.IsHealer(job)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `job` | `Job` | 要判断的职业。 |

作用：判断职业是否为治疗。

返回值：`bool`。职业为白魔法师、学者、占星术士或贤者时返回 `true`。

## GetDeadPartyMemberWithPriority

方法：

```csharp
HealerHelper.GetDeadPartyMemberWithPriority(range, minDeadSeconds)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |
| `minDeadSeconds` | `float` | 要求持续死亡的最短时间，单位为秒。 |

作用：获取范围内优先复活的死亡小队成员。

返回值：`IBattleChara?`。没有符合条件的成员时返回 `null`。

说明：排除拥有状态 `148` 的成员。优先级依次为治疗、坦克、输出；同一职能优先选择更早死亡的成员。

## GetAllDeadPartyMembersSorted

方法：

```csharp
HealerHelper.GetAllDeadPartyMembersSorted(range, minDeadSeconds)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |
| `minDeadSeconds` | `float` | 要求持续死亡的最短时间，单位为秒。 |

作用：获取范围内符合复活条件的全部死亡小队成员，并按复活优先级排序。

返回值：`List<IBattleChara>`。没有符合条件的成员时返回空列表。

说明：排除拥有状态 `148` 的成员。优先级依次为治疗、坦克、输出；同一职能按死亡时间排序。

## HasDeadPartyMember

方法：

```csharp
HealerHelper.HasDeadPartyMember(range, minDeadSeconds)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |
| `minDeadSeconds` | `float` | 要求持续死亡的最短时间，单位为秒。 |

作用：判断范围内是否存在符合复活条件的死亡小队成员。

返回值：`bool`。存在时返回 `true`，否则返回 `false`。

## GetDeadPartyMemberCount

方法：

```csharp
HealerHelper.GetDeadPartyMemberCount(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |

作用：统计范围内没有状态 `148` 的死亡小队成员数量。

返回值：`int`。本地玩家不可用时返回 `0`。

## GetPartyMembersBelowHpInRange

方法：

```csharp
HealerHelper.GetPartyMembersBelowHpInRange(range, hpThreshold)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |
| `hpThreshold` | `float` | 生命值百分比阈值，范围为 `0` 至 `100`。 |

作用：获取范围内生命值百分比低于阈值的存活小队成员，包括本地玩家。

返回值：`List<IBattleChara>`。没有符合条件的成员时返回空列表。

## GetPartyMembersBelowHpCountInRange

方法：

```csharp
HealerHelper.GetPartyMembersBelowHpCountInRange(range, hpThreshold)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |
| `hpThreshold` | `float` | 生命值百分比阈值，范围为 `0` 至 `100`。 |

作用：统计范围内生命值百分比低于阈值的存活小队成员数量，包括本地玩家。

返回值：`int`。返回符合条件的成员数量。

## GetLowestHpTargetInRange

方法：

```csharp
HealerHelper.GetLowestHpTargetInRange(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |

作用：获取范围内生命值百分比最低的存活小队成员，包括本地玩家。

返回值：`IBattleChara?`。没有符合条件的成员时返回 `null`。

## GetLowestHpPercentInRange

方法：

```csharp
HealerHelper.GetLowestHpPercentInRange(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |

作用：获取范围内存活小队成员的最低生命值百分比。

返回值：`float`。范围为 `0` 至 `100`；没有符合条件的成员时返回 `100`。

## GetTanksBelowHpInRange

方法：

```csharp
HealerHelper.GetTanksBelowHpInRange(range, hpThreshold)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |
| `hpThreshold` | `float` | 生命值百分比阈值，范围为 `0` 至 `100`。 |

作用：统计范围内生命值百分比低于阈值的存活坦克数量。

返回值：`int`。返回符合条件的坦克数量。

## GetLowestHpTankInRange

方法：

```csharp
HealerHelper.GetLowestHpTankInRange(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |

作用：获取范围内生命值百分比最低的存活坦克。

返回值：`IBattleChara?`。没有符合条件的坦克时返回 `null`。

## NeedEmergencyHeal

方法：

```csharp
HealerHelper.NeedEmergencyHeal(range, emergencyThreshold)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |
| `emergencyThreshold` | `float` | 紧急治疗生命值阈值。可省略；默认为 `20`。 |

作用：判断范围内是否有存活小队成员的生命值百分比低于紧急治疗阈值。

返回值：`bool`。存在时返回 `true`，否则返回 `false`。

## GetAveragePartyHpInRange

方法：

```csharp
HealerHelper.GetAveragePartyHpInRange(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |

作用：计算范围内存活小队成员的平均生命值百分比，包括本地玩家。

返回值：`float`。没有符合条件的成员时返回 `100`。

## GetAlivePartyMemberCountInRange

方法：

```csharp
HealerHelper.GetAlivePartyMemberCountInRange(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |

作用：统计范围内的存活小队成员数量，包括本地玩家。

返回值：`int`。本地玩家不可用时返回 `0`。

## GetPartyMembersInRange

方法：

```csharp
HealerHelper.GetPartyMembersInRange(range)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `range` | `float` | 与本地玩家的最大距离，单位为米。 |

作用：获取范围内的存活小队成员，不包括本地玩家。

返回值：`List<IBattleChara>`。没有符合条件的成员时返回空列表。
