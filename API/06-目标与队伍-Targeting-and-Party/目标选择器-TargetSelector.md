# 目标选择器 TargetSelector

命名空间：`PromeRotation.Data`、`PromeRotation.TargetSelector`

用于切换宿主的自动目标选择模式。

## TargetSelectorMode

属性：

```csharp
PromeSettings.Instance.TargetSelectorMode
```

参数含义：无。

作用：读取或切换自动目标选择模式。

返回值：`SelectorModeType`。此属性可读写。

## SelectorModeType

| 值 | 含义 |
| --- | --- |
| `None` | 关闭自动目标选择。 |
| `Closest` | 选择距离最近的目标。 |
| `Farthest` | 选择距离最远的目标。 |
| `LowestHp` | 选择血量百分比最低的目标。 |
| `HighestHp` | 选择血量百分比最高的目标。 |
| `LowestHpIn3R` | 选择 3 米内血量百分比最低的目标。 |
| `HighestHpIn3R` | 选择 3 米内血量百分比最高的目标。 |
| `LowestHpIn6R` | 选择 6 米内血量百分比最低的目标。 |
| `HighestHpIn6R` | 选择 6 米内血量百分比最高的目标。 |

说明：距离按目标圈边缘计算。未指定距离的模式在 50 米内选择目标。
