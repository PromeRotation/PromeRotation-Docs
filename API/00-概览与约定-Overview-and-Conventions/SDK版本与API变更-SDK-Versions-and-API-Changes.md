# SDK 版本与 API 变更 SDK Versions and API Changes

本页记录 `PromeRotation.SDK.API15` 每个版本对应的 PromeRotation 版本，以及公开 API 的增删。

## 0.1.0-preview.6

对应 PromeRotation `1.5.10.4`（PromeRotation-1.0 `main`）。上一版 `0.1.0-preview.5` 对应 2026-07 的 PromeRotation。

### 已文档化的新增成员

| 成员 | 文档 |
| --- | --- |
| `Core.MyPrLevel()` | [本地玩家 Core](../05-游戏数据与状态-Game-Data-and-State/本地玩家-Core.md#myprlevel) |
| `GameData.IsInPvPArea()` | [游戏状态 GameData](../05-游戏数据与状态-Game-Data-and-State/游戏状态-GameData.md#isinpvparea) |
| `GameData.IsTargetInCombat()` | [游戏状态 GameData](../05-游戏数据与状态-Game-Data-and-State/游戏状态-GameData.md#istargetincombat) |
| `GameData.TryGetCountdown(out float)` | [游戏状态 GameData](../05-游戏数据与状态-Game-Data-and-State/游戏状态-GameData.md#trygetcountdown) |
| `StatusHelper.GetStatusSourceEntityId` / `HasStatusWithSource` | [状态效果 StatusHelper](../05-游戏数据与状态-Game-Data-and-State/状态效果-StatusHelper.md#getstatussourceentityid) |
| `BattleCharaExtensions.GetStatusSourceEntityId` / `HasStatusWithSource` | [战斗角色扩展 BattleCharaExtensions](../05-游戏数据与状态-Game-Data-and-State/战斗角色扩展-BattleCharaExtensions.md#getstatussourceentityid) |
| `PartyHelper.GetSortedParty()` / `GetSortedPartyMember(PartyPrioritySlot)` | [队伍优先级 PartyPriority](../06-目标与队伍-Targeting-and-Party/队伍优先级-PartyPriority.md) |
| `TargetHelper.FindTargetableNpc` | [目标查询 TargetHelper](../06-目标与队伍-Targeting-and-Party/目标查询-TargetHelper.md#findtargetablenpc) |

### 新增的公开命名空间与类型（尚未文档化）

以下类型在 SDK 中可见，但还没有稳定性承诺，使用前请以源码为准：

- `PromeRotation.Spatial.Drawing`：`DrawManager`、`DrawHandle`、`DrawStyle`、`DrawBinding`、`DrawBindingBuilder` 等绘制 API，部分已在 [08-绘制](../08-绘制-Draw/) 中说明。
- `PromeRotation.Spatial.SafeZoneReborn.Solver`：新版安全区求解器（`SafeZoneSolver`、`SolveOptions`、`SafeZoneSolution`、`TimedWaypoint` 等）。
- `PromeRotation.Managers.PartyPriority`：`PartyPrioritySlot`、`PartyPriorityManager`。
- `PromeRotation.LogSystem`：`ILogSystemEventSource`，以及新事件 `LogSystemInstanceContentTextEvent`（见[事件订阅](../11-事件与网络-Events-and-Network/事件订阅-EventSubscription.md#instancecontenttext)）。
- `PromeRotation.Hosting.MechanicControl`。
- `PromeRotation.Helpers`：`DutyActionHelper`、`PVPHelper`。
- `PromeRotation.Updaters`：`ActionUpdaterRouter`、`ActionUpdaterMode`、`PvpActionUpdater`。
- `PromeRotation.UI.QuickToggles`：`QtDefinition`、`QtRegistry`。
- `PromeRotation.Timeline.Core` / `Actions` / `Conditions`：`ITimelineExecutionContext`、`TimelineJobCatalog`、`EnqueueSkillGroupAction`、`ShowHintAction`、`SelfJobCondition` 等。
- `PromeRotation.Timeline.Library`、`Prome.TimelineWire`：时间轴库（下载、上传、本地索引）及其传输格式，属于宿主内部实现，不建议外部直接依赖。

### 移除的公开类型

以下类型已从公开 API 中移除，引用它们的代码需要迁移：

| 类型 | 迁移方式 |
| --- | --- |
| `PromeRotation.LogSystem.LogSystemBridge`、`LogSystemHub` | 改用 `ILogSystemEventSource` 或脚本上下文的事件订阅，见[事件订阅](../11-事件与网络-Events-and-Network/事件订阅-EventSubscription.md)。 |
| `PromeRotation.Managers.CombatEventManager.Hooks.CreateObjectHook`、`KnockbackHook`、`ObjectEffectHook` | 改为订阅 `ObjectCreate`、`Knockback`、`EObjAnimation` 等事件。 |
| `PromeRotation.Managers.CombatEventManager.Parsers.*`（`ActionEffectParser`、`ActorControlParser`、`CreateObjectParser`、`NpcYellParser`、`StartCastParser`） | 解析器不再公开，使用对应的事件。 |
| `PromeRotation.Managers.CombatEventManager.Structs.*`（`EffectsEntry`、`Ender`、`FFXIVIpcObjectSpawn`、`NpcYellStruct`） | 原始包结构不再公开，使用事件中的字段。 |
| `PromeRotation.Managers.LogMessageManager.BattleTalk2` | 使用 `NpcYell` 或 `InstanceContentText` 事件。 |
| `PromeRotation.UI.LogConsole`、`PromeRotation.Windows.CombatEventLogWindow` | 宿主内部窗口，无替代。 |
