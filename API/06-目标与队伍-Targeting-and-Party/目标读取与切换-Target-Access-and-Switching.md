# 目标读取与切换 Target Access and Switching

命名空间：`PromeRotation.Core`

用于读取、设置或清除当前目标。

## Target

属性：

```csharp
Core.Target
```

参数含义：无。

作用：获取本地玩家当前选中的战斗角色。

返回值：`IBattleChara?`。当前目标不是战斗角色或没有目标时返回 `null`。

## Core.SetTarget

方法：

```csharp
Core.SetTarget(target)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `target` | `IBattleChara?` | 要设置的目标。传入 `null` 时清除目标。 |

作用：设置或清除当前目标。

返回值：无。

说明：本地玩家不可用时，无法设置非空目标。

## GameData.SetTarget

方法：

```csharp
GameData.SetTarget(target)
```

参数含义：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `target` | `IBattleChara` | 要设置的目标。 |

作用：将可选中且存活的战斗角色设为当前目标。

返回值：`bool`。设置成功时返回 `true`，否则返回 `false`。

说明：此方法不能清除目标。
