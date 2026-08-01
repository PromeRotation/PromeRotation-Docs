# 本地玩家 Core

命名空间：`PromeRotation.Core`

用于获取当前本地玩家。

## Me

属性：

```csharp
Core.Me
```

参数含义：无。

作用：获取当前本地玩家的战斗角色对象。

返回值：`IBattleChara?`。本地玩家不可用时返回 `null`。

说明：角色状态查询可使用[战斗角色扩展 BattleCharaExtensions](./战斗角色扩展-BattleCharaExtensions.md)。`Core.Target` 与 `Core.SetTarget()` 属于目标选择 API。
