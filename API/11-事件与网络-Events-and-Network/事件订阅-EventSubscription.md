# 事件订阅 EventSubscription

命名空间：`PromeRotation.LogSystem` / `PromeRotation.Hosting`

订阅宿主公开的战斗与环境事件。所有事件按 ACT/Overlay 日志行的最终语义归类，共 32 个公开事件。

## 注意事项

- 事件中的**包内位置**（如 `EffectPosition`）直接来自网络包，可视为服务器判定的位置。
- 事件中的**位置快照**（字段说明标注"快照"）来自本地客户端对象表内存，是本地缓存值，不一定准确。
- 在强位置要求的逻辑中（如判定站位），优先使用包内位置，避免使用客户端快照。

## 术语

`Object` / `Actor` / `Entity` / `Mob` / `Combatant` 均表示游戏中具有战斗数据的实体（玩家、Boss、木人、召唤物等），本文统一译为**实体**。

## 订阅方式

订阅返回 `IDisposable`，调用 `Dispose` 即退订；事件在 Dalamud Framework 线程发布。

- **脚本快捷方法**：`ctx.OnStartCast(...)`、`ctx.OnStatusAdd(...)` 等，每条事件下方列出
- **强类型订阅**：`Events.Subscribe<LogSystemCastStartEvent>(...)`（`Events` 为 `ILogSystemEventSource`）
- **按名称订阅**：`ctx.Subscribe("CastStart", ...)`

所有事件继承公共字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `Timestamp` | `DateTime` | 本地捕获时间（不是服务器权威时间） |
| `Kind` | `LogSystemEventKind` | 事件种类 |
| `Raw` | `object?` | 原始载荷，不保证跨版本稳定，勿依赖 |

## 目录

- [咏唱与技能](#cast-group)
  - [CastStart](#caststart)
  - [ActionEffect](#actioneffect)
  - [CancelCast](#cancelcast)
  - [InterruptCast](#interruptcast)
- [状态](#status-group)
  - [StatusAdd](#statusadd)
  - [StatusRemove](#statusremove)
- [点名与标记](#marker-group)
  - [TargetIcon](#targeticon)
  - [Marker](#marker)
  - [Waymark](#waymark)
- [连线](#tether-group)
  - [Tether](#tether)
  - [TetherCancel](#tethercancel)
- [战斗状态与副本文本](#battle-group)
  - [BattleStarted](#battlestarted)
  - [BattleEnded](#battleended)
  - [CountdownStarted](#countdownstarted)
  - [NpcYell](#npcyell)
  - [InstanceContentText](#instancecontenttext)
  - [MapEffect](#mapeffect)
  - [DirectorUpdate](#directorupdate)
- [实体](#entity-group)
  - [ObjectCreate](#objectcreate)
  - [ObjectRemove](#objectremove)
  - [Targetable](#targetable)
  - [SetObjPos](#setobjpos)
- [ActorControl 扩展](#actor-control-group)
  - [PlayActionTimeline](#playactiontimeline)
  - [TargetVfx](#targetvfx)
  - [EObjAnimation](#eobjanimation)
  - [EObjSetState](#eobjsetstate)
  - [Knockback](#knockback)
  - [MorelogCompat](#morelogcompat)
- [场景与其他](#scene-group)
  - [Territory](#territory)
  - [ChangeMap](#changemap)
  - [WeatherChanged](#weatherchanged)
  - [ChatLog](#chatlog)
  - [TimelineExecution](#timelineexecution)

---

<a id="cast-group"></a>
## 咏唱与技能

<a id="caststart"></a>
### CastStart

类型：`LogSystemCastStartEvent`（`LogSystemEventKind.CastStart`）

订阅：`ctx.OnStartCast(...)` 或 `Events.Subscribe<LogSystemCastStartEvent>(...)`

作用：实体开始咏唱技能时触发。咏唱成功后会触发技能生效事件，被取消或打断则触发取消咏唱事件。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `ActionId` | `ushort` | 咏唱动作 ID |
| `CastTime` | `float` | 咏唱总时长（秒） |
| `SourceId` / `TargetId` | `ulong` | 咏唱实体 / 咏唱目标实体实例 ID（地面技能目标可为 0） |
| `CanInterrupt` | `byte` | 可打断标记（0/1 原始值，不是 bool） |
| `EffectPosition` | `Vector3` | 包内效果位置（服务器判定） |
| `SourcePosition` / `TargetPosition` | `Vector3?` | 咏唱实体 / 目标实体位置快照（本地内存，实体缺失时为 null） |
| `SourceRotation` / `TargetRotation` | `float?` | 咏唱实体 / 目标实体朝向快照（弧度，南为 0；包内回退值非空） |

注意：`EffectPosition` 与 `TargetPosition` 是两个字段；位置以包内值为准，见[注意事项](#注意事项)。

<a id="actioneffect"></a>
### ActionEffect

类型：`LogSystemActionEffectEvent`（`LogSystemEventKind.ActionEffect`）

订阅：`ctx.OnActionEffect(...)` 或 `Events.Subscribe<LogSystemActionEffectEvent>(...)`

作用：技能生效时触发。单体技能命中一个目标，群体技能命中多目标或 0 目标；AOE 按目标拆成多条，同批共享 `GlobalSequence`。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `GlobalSequence` | `uint` | 本次动作的全局序号，同一次 AOE 的所有目标共享 |
| `TargetIndex` | `int` | 当前目标在目标数组中的下标（0 起始） |
| `ActionId` | `uint` | 动作 ID |
| `SourceId` / `TargetId` | `ulong` | 来源实体 / 命中实体实例 ID（目标为 0 时不发布） |
| `EffectPosition` | `Vector3?` | 包内动作位置（服务器判定） |
| `SourcePosition` / `TargetPosition` | `Vector3?` | 来源实体 / 命中实体位置快照 |
| `SourceRotation` / `TargetRotation` | `float?` | 来源实体 / 命中实体朝向快照 |

注意：仅凭 `ActionId` 和坐标无法恢复 effect array（伤害/治疗数值不公开）。

<a id="cancelcast"></a>
### CancelCast

类型：`LogSystemCancelCastEvent`（`LogSystemEventKind.CancelCast`）

订阅：`ctx.OnCancelCast(...)` 或 `Events.Subscribe<LogSystemCancelCastEvent>(...)`

作用：咏唱因主动取消或移动而中断时触发（ActorControl 0x0F）。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 取消咏唱的实体实例 ID |
| `ActionId` | `uint` | 被取消动作 ID |
| `SourcePosition` | `Vector3?` | 实体位置快照 |

注意：与打断咏唱是不同事件（不同 category），迁移触发器时需核对。

<a id="interruptcast"></a>
### InterruptCast

类型：`LogSystemInterruptCastEvent`（`LogSystemEventKind.InterruptCast`）

订阅：`ctx.OnInterruptCast(...)` 或 `Events.Subscribe<LogSystemInterruptCastEvent>(...)`

作用：咏唱被打断时触发（ActorControl 0x5F1）。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 被打断的实体实例 ID |
| `ActionId` | `uint` | 被打断动作 ID |
| `SourcePosition` | `Vector3?` | 实体位置快照 |

注意：ACT 的 0x17 行把取消与打断合并为一条，本事件将其拆开，迁移时不能仅按事件名等同。

---

<a id="status-group"></a>
## 状态

<a id="statusadd"></a>
### StatusAdd

类型：`LogSystemStatusAddEvent`（`LogSystemEventKind.StatusAdd`）

订阅：`ctx.OnStatusAdd(...)` 或 `Events.Subscribe<LogSystemStatusAddEvent>(...)`

作用：实体获得状态（增益或减益）时触发；状态刷新也触发并标记 `IsRefresh`。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `StatusId` | `uint` | 状态 ID |
| `SourceId` / `TargetId` | `ulong` | 状态来源实体 / 持有者实例 ID |
| `Duration` | `float` | 剩余时长（秒，绝对值） |
| `StackCount` | `byte` | 层数（低位参数，并非所有状态都表示层数） |
| `IsRefresh` | `bool` | 是否判为刷新 |
| `SourcePosition` / `TargetPosition` | `Vector3?` | 位置快照 |

注意：持续时间不能保证精确，个别情况已计时一部分；层数可能表示其他信息（如忍术组合），需与游戏数据核对。

<a id="statusremove"></a>
### StatusRemove

类型：`LogSystemStatusRemoveEvent`（`LogSystemEventKind.StatusRemove`）

订阅：`ctx.OnStatusRemove(...)` 或 `Events.Subscribe<LogSystemStatusRemoveEvent>(...)`

作用：状态移除时触发；被覆盖的状态和实体死亡移除的状态也会触发。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `StatusId` | `uint` | 被移除状态 ID |
| `SourceId` / `TargetId` | `ulong` | 最后状态来源实体 / 持有者实例 ID |
| `Duration` | `float` | 最后剩余时长（秒） |
| `StackCount` | `byte` | 最后层数 |
| `SourcePosition` / `TargetPosition` | `Vector3?` | 位置快照 |

注意：实体离开对象表导致状态不可见时也可能触发，不能自动解释为驱散。

---

<a id="marker-group"></a>
## 点名与标记

<a id="targeticon"></a>
### TargetIcon

类型：`LogSystemTargetIconEvent`（`LogSystemEventKind.TargetIcon`）

订阅：`ctx.OnTargetIcon(...)` 或 `Events.Subscribe<LogSystemTargetIconEvent>(...)`

作用：实体头顶出现点名标记时触发。标记 ID 跨副本一致地表示标记的视觉效果，新副本可能有按实例扰码的偏移。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `TargetId` | `ulong` | 标记目标实体实例 ID |
| `MarkerId` | `uint` | 标记 ID |
| `SourceId` | `ulong` | ActorControl 来源实体（不一定是标记目标） |

注意：未验证当前版本扰码规则前不要硬编码历史减法；标记何时消失没有公开事件。

<a id="marker"></a>
### Marker

类型：`LogSystemMarkerEvent`（`LogSystemEventKind.Marker`）

订阅：`ctx.OnMarker(...)` 或 `Events.Subscribe<LogSystemMarkerEvent>(...)`

作用：实体头顶的玩家标点（攻击/锁链/禁止/形状等）被添加或移除时触发。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `TargetId` | `ulong` | 被标记实体实例 ID |
| `MarkerId` | `uint` | 场上标记 ID（Prome 已 +1，与原始 category 值不同） |
| `SourceId` | `ulong` | 设置标记的来源实体 |

注意：与场地标点 `Waymark` 完全不同；插件本地标点不产生网络数据。

<a id="waymark"></a>
### Waymark

类型：`LogSystemWaymarkEvent`（`LogSystemEventKind.Waymark`）

订阅：`ctx.OnLogEvent<LogSystemWaymarkEvent>(...)` 或 `Events.Subscribe<LogSystemWaymarkEvent>(...)`

作用：场地标点（A/B/C/D/1/2/3/4）被添加或移除时触发。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `MarkerId` | `byte` | 标点槽（0..7，对应 A/B/C/D/1/2/3/4） |
| `Active` | `bool` | 是否启用（移除时为 false） |
| `Position` | `Vector3` | 标点位置（inactive 时不可当新坐标） |

注意：由本地标点状态差分产生，不是网络行的直接复刻；启动时已有标点构成初始状态。

---

<a id="tether-group"></a>
## 连线

<a id="tether"></a>
### Tether

类型：`LogSystemTetherEvent`（`LogSystemEventKind.Tether`）

订阅：`ctx.OnTether(...)` 或 `Events.Subscribe<LogSystemTetherEvent>(...)`

作用：敌我之间的连线建立时触发。连线 ID 跨副本一致地表示连线的视觉样式，对应 Channeling 表。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `TetherId` | `uint` | 连线种类 ID |
| `SourceId` / `TargetId` | `ulong` | 连线两端实体实例 ID |
| `Category` | `ActorControlId` | ActorControl category（此处 0x23，保留诊断值） |
| `SourcePosition` / `TargetPosition` | `Vector3?` | 两端位置快照 |

注意：玩家间技能连线（龙视、护盾等）不产生此事件；部分机制会周期性重复施加连线，导致重复行。

<a id="tethercancel"></a>
### TetherCancel

类型：`LogSystemTetherCancelEvent`（`LogSystemEventKind.TetherCancel`）

订阅：`ctx.OnTetherCancel(...)` 或 `Events.Subscribe<LogSystemTetherCancelEvent>(...)`

作用：连线解除时触发（ACT 无对应日志行，由本宿主独立公开）。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `TetherId` | `uint` | 连线种类 ID |
| `SourceId` / `TargetId` | `ulong` | 连线两端实体实例 ID |
| `Category` | `ActorControlId` | ActorControl category（此处 0x2F） |
| `SourcePosition` / `TargetPosition` | `Vector3?` | 两端位置快照 |

注意：不能假定取消包一定复述建立时的目标与类型。

---

<a id="battle-group"></a>
## 战斗状态与副本文本

<a id="battlestarted"></a>
### BattleStarted

类型：`LogSystemBattleStartedEvent`（`LogSystemEventKind.BattleStarted`）

订阅：`ctx.OnBattleStarted(...)` 或 `Events.Subscribe<LogSystemBattleStartedEvent>(...)`

作用：本地玩家进入战斗状态时触发（本地战斗标志上升沿）。

无事件专属字段。

注意：这是本地玩家的战斗标志，不等于副本统一开战，不携带敌人或遭遇信息。

<a id="battleended"></a>
### BattleEnded

类型：`LogSystemBattleEndedEvent`（`LogSystemEventKind.BattleEnded`）

订阅：`ctx.OnBattleEnded(...)` 或 `Events.Subscribe<LogSystemBattleEndedEvent>(...)`

作用：本地玩家脱离战斗状态时触发（本地战斗标志下降沿）。

无事件专属字段。

注意：死亡、脱战、场景切换都可能使标志下降，不要自动解释为副本胜利。

<a id="countdownstarted"></a>
### CountdownStarted

类型：`LogSystemCountdownStartedEvent`（`LogSystemEventKind.CountdownStarted`）

订阅：`ctx.OnCountdownStarted(...)` 或 `Events.Subscribe<LogSystemCountdownStartedEvent>(...)`

作用：尝试开始倒计时时触发。"战斗开始"提示不对应任何日志行，需要检测战斗开始请用 `BattleStarted`。

无事件专属字段。

注意：不公开发起者、初始秒数、剩余秒数，也不发布倒计时取消事件。

<a id="npcyell"></a>
### NpcYell

类型：`LogSystemNpcYellEvent`（`LogSystemEventKind.NpcYell`）

订阅：`ctx.OnNpcYell(...)` 或 `Events.Subscribe<LogSystemNpcYellEvent>(...)`

作用：NPC 以头顶气泡形式喊出台词时触发。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `NpcYellId` | `ushort` | 台词文本行 ID（对应 NpcYell 表，稳定判断优先用它） |
| `NameId` | `uint` | NPC 名称行 ID（对应 BNpcName 表，不是 DataId） |
| `SourceId` | `ulong` | 发言 NPC 实例 ID |

注意：事件不直接携带最终本地化文本，需按 sheet 和语言解析。

<a id="instancecontenttext"></a>
### InstanceContentText

类型：`LogSystemInstanceContentTextEvent`（`LogSystemEventKind.InstanceContentText`）

订阅：`ctx.OnLogEvent<LogSystemInstanceContentTextEvent>(...)` 或 `Events.Subscribe<LogSystemInstanceContentTextEvent>(...)`

作用：副本在屏幕中央显示战斗台词（InstanceContentTextData）时触发。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `RowID` | `uint` | InstanceContentTextData 表行 ID（稳定判断优先用它） |
| `Text` | `string` | 解析后的台词文本（含 RSV 替换）；无法解析时为空字符串 |

注意：`Text` 随客户端语言变化，触发器应优先用 `RowID`。

<a id="mapeffect"></a>
### MapEffect

类型：`LogSystemMapEffectEvent`（`LogSystemEventKind.MapEffect`）

订阅：`ctx.OnMapEffect(...)` 或 `Events.Subscribe<LogSystemMapEffectEvent>(...)`

作用：副本实例中产生场地特效时触发。场地特效不依赖任何实体或技能，部分机制在判定前只有此事件可判断；同一处特效在生成、激活、消失时各触发一条。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `Position` | `uint` | 特效位置索引（对应内存中 128 槽位数组，不是三维坐标） |
| `Param1` / `Param2` | `ushort` | 特效开关标志位（每位置 16 通道），可按机制需要组合 |

注意：`flags`/`location` 的含义随副本不同，务必限制区域，否则会在其他副本中意外触发。

<a id="directorupdate"></a>
### DirectorUpdate

类型：`LogSystemDirectorUpdateEvent`（`LogSystemEventKind.DirectorUpdate`）

订阅：`ctx.OnDirectorUpdate(...)` 或 `Events.Subscribe<LogSystemDirectorUpdateEvent>(...)`

作用：副本演出控制（ActorControlSelf 的 DirectorUpdate 类别）时触发，如 BGM 切换、部分过场动画、护盾开关、淡入淡出。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `Instance` | `uint` | 副本实例/内容标识（前两字节更新类型，后两字节 InstanceContentType） |
| `Command` | `uint` | Director 命令 |
| `Param1`…`Param4` | `uint` | 命令参数 |

注意：参数必须按具体 `Instance`/`Command` 解读，不能脱离命令给参数统一语义。

---

<a id="entity-group"></a>
## 实体

<a id="objectcreate"></a>
### ObjectCreate

类型：`LogSystemObjectCreateEvent`（`LogSystemEventKind.ObjectCreate`）

订阅：`ctx.OnCreateObject(...)` 或 `Events.Subscribe<LogSystemObjectCreateEvent>(...)`

作用：新实体被添加到玩家客户端时触发。距离缩短、同屏实体数下降导致实体出现时也会触发。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 实体实例 ID（实体主键） |
| `DataId` | `uint` | 实体数据 ID |
| `NpcNameId` | `uint` | NPC 名称行 ID（非 NPC 可为 0） |
| `Name` | `string` | 观察时名称（本地化） |
| `ObjectKind` | `byte` | 实体类别原值 |
| `OwnerId` | `ulong` | 所有者实例 ID（召唤物为召唤者，无所有者常为 0） |
| `SourcePosition` | `Vector3` | 位置快照 |
| `SourceRotation` | `float` | 朝向快照 |

注意：这是"本客户端首次观察"，不保证等于网络生成时刻。

<a id="objectremove"></a>
### ObjectRemove

类型：`LogSystemObjectRemoveEvent`（`LogSystemEventKind.ObjectRemove`）

订阅：`ctx.OnLogEvent<LogSystemObjectRemoveEvent>(...)` 或 `Events.Subscribe<LogSystemObjectRemoveEvent>(...)`

作用：实体从玩家客户端被移除时触发。切换地图、死亡、同屏实体过多、距离过远都会移除。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 离表实体实例 ID（稳定识别字段） |
| `DataId` | `uint` | 最后数据 ID |
| `ObjectKind` | `byte` | 最后实体类别原值 |
| `SourcePosition` / `SourceRotation` | `Vector3` / `float` | 最后观察位置 / 朝向（不是移除包数据） |

注意：实体暂时离开对象表也会表现为移除，不要推导死亡。

<a id="targetable"></a>
### Targetable

类型：`LogSystemTargetableEvent`（`LogSystemEventKind.Targetable`）

订阅：`ctx.OnLogEvent<LogSystemTargetableEvent>(...)` 或 `Events.Subscribe<LogSystemTargetableEvent>(...)`

作用：实体名称牌的显示/隐藏切换时触发（`IsTargetable`），可用来判断实体何时可选中。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 实体实例 ID |
| `IsTargetable` | `bool` | 当前可选中状态 |
| `DataId` | `uint` | 实体数据 ID |
| `ObjectKind` | `byte` | 实体类别原值 |
| `SourcePosition` / `SourceRotation` | `Vector3?` / `float?` | 位置 / 朝向快照 |

注意：名称显示、可见性与可选中并非总是同一状态。

<a id="setobjpos"></a>
### SetObjPos

类型：`LogSystemSetObjPosEvent`（`LogSystemEventKind.SetObjPos`）

订阅：`ctx.OnSetObjPos(...)` 或 `Events.Subscribe<LogSystemSetObjPosEvent>(...)`

作用：实体被直接设置到指定位置时触发（无需移动过程），如绝本运动会中大量不可选中的分身实体。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 实体实例 ID |
| `Position` | `Vector3` | 观察时位置 |
| `Rotation` | `float` | 观察时朝向 |
| `DataId` | `uint` | 观察时数据 ID |

注意：由特定 ActorControl category 触发的位置快照，不是连续移动流，同一实体的普通移动不会持续触发。

---

<a id="actor-control-group"></a>
## ActorControl 扩展

以下事件由 ActorControl 网络包拆分而来，参数语义需在具体副本上下文解释。

<a id="playactiontimeline"></a>
### PlayActionTimeline

类型：`LogSystemPlayActionTimelineEvent`（`LogSystemEventKind.PlayActionTimeline`）

订阅：`ctx.OnPlayActionTimeline(...)` 或 `Events.Subscribe<LogSystemPlayActionTimelineEvent>(...)`

作用：实体播放动作时间轴命令时触发（ActorControl 0x197），如跳跃、落地动画。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 播放时间轴的实体实例 ID |
| `TimelineId` | `uint` | 时间轴 ID |
| `SourcePosition` / `SourceRotation` | `Vector3?` / `float?` | 位置 / 朝向快照 |

注意：这是播放命令，不保证动画已完成。

<a id="targetvfx"></a>
### TargetVfx

类型：`LogSystemTargetVfxEvent`（`LogSystemEventKind.TargetVfx`）

订阅：`ctx.OnTargetVfx(...)` 或 `Events.Subscribe<LogSystemTargetVfxEvent>(...)`

作用：目标特效触发时（ActorControl 0xB8）。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | ActorControl 来源实体（常为特效宿主） |
| `VfxId` | `uint` | 目标特效 ID |
| `SourcePosition` / `SourceRotation` | `Vector3?` / `float?` | 位置 / 朝向快照 |

注意：不公开目标 ID 参数，需要目标关联的逻辑不能仅依赖此事件。

<a id="eobjanimation"></a>
### EObjAnimation

类型：`LogSystemEObjAnimationEvent`（`LogSystemEventKind.EObjAnimation`）

订阅：`ctx.OnEObjAnimation(...)` 或 `Events.Subscribe<LogSystemEObjAnimationEvent>(...)`

作用：事件物件（EObj）播放动画时触发（ActorControl 0x19D）。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 事件物件实例 ID |
| `Param1` / `Param2` | `ushort` | 动画参数（需两者共同解释） |
| `SourcePosition` / `SourceRotation` | `Vector3?` / `float?` | 位置 / 朝向快照 |

注意：参数不是通用动画 ID/状态枚举，需要副本样本解释。

<a id="eobjsetstate"></a>
### EObjSetState

类型：`LogSystemEObjSetStateEvent`（`LogSystemEventKind.EObjSetState`）

订阅：`ctx.OnEObjSetState(...)` 或 `Events.Subscribe<LogSystemEObjSetStateEvent>(...)`

作用：事件物件状态切换时触发（ActorControl 0x199）。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 事件物件实例 ID |
| `State` | `ushort` | 事件物件状态 |
| `SourcePosition` / `SourceRotation` | `Vector3?` / `float?` | 位置 / 朝向快照 |

注意：状态值只能在具体物件/副本上下文解释，不得当成统一枚举。

<a id="knockback"></a>
### Knockback

类型：`LogSystemKnockbackEvent`（`LogSystemEventKind.Knockback`）

订阅：`ctx.OnLogEvent<LogSystemKnockbackEvent>(...)` 或 `Events.Subscribe<LogSystemKnockbackEvent>(...)`

作用：实体开始实际执行强制位移（击退/拉近）时触发（ActorControl 0xE1）。它不是对未来机制的预测。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | 关联实体（通常视作被移动实体） |
| `Rotation` | `float` | 移动方向（弧度，由 param1 解码） |
| `Distance` | `float` | 强制移动距离（yalm，由 param2 解码） |
| `Speed` | `float` | 移动速度参数（原值保留在 `Raw`） |
| `SourcePosition` | `Vector3?` | 事件发布时实体位置快照 |

注意：免疫、拉近以及特殊位移是否产生相同事件需实战样本确认；它不替代技能生效中的击退效果类型。

<a id="morelogcompat"></a>
### MorelogCompat

类型：`LogSystemMorelogCompatEvent`（`LogSystemEventKind.MorelogCompat`）

订阅：`ctx.OnMorelogCompat(...)` 或 `Events.Subscribe<LogSystemMorelogCompatEvent>(...)`

作用：一组未拆分为专用事件的 ActorControl category 的兜底记录。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `SourceId` | `ulong` | ActorControl 来源实体实例 ID |
| `CategoryId` | `uint` | ActorControl category（需按 category 解释） |
| `DataId` | `uint` | 观察时数据 ID |
| `SourcePosition` / `SourceRotation` | `Vector3?` / `float?` | 位置 / 朝向快照 |

注意：`Raw` 中的其余参数不属于稳定契约；业务代码应优先订阅已拆出的专用事件。

---

<a id="scene-group"></a>
## 场景与其他

<a id="territory"></a>
### Territory

类型：`LogSystemTerritoryEvent`（`LogSystemEventKind.Territory`）

订阅：`ctx.OnTerritoryChanged(...)` 或 `Events.Subscribe<LogSystemTerritoryEvent>(...)`

作用：切换区域时触发。ACT 启动后开启游戏也会触发，很适合用于初始化。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `TerritoryId` | `uint` | 区域特征 ID（不是地图 ID） |
| `TerritoryName` | `string` | 区域名称（本地化，受语言影响） |

注意：区域切换和地图切换是不同事实；同一区域内可发生 `ChangeMap`。

<a id="changemap"></a>
### ChangeMap

类型：`LogSystemChangeMapEvent`（`LogSystemEventKind.ChangeMap`）

订阅：`ctx.OnLogEvent<LogSystemChangeMapEvent>(...)` 或 `Events.Subscribe<LogSystemChangeMapEvent>(...)`

作用：切换地图时触发，包括切换区域或同一区域内切换子区域。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `MapId` | `uint` | 地图 ID（不是 Territory ID） |
| `RegionName` | `string` | 地域名（如"萨纳兰"） |
| `PlaceName` | `string` | 区域名 |
| `SubName` | `string` | 子区域名（无子区域时为空） |

注意：本地化名称不适合做稳定判断键。

<a id="weatherchanged"></a>
### WeatherChanged

类型：`LogSystemWeatherChangedEvent`（`LogSystemEventKind.WeatherChanged`）

订阅：`ctx.OnWeatherChanged(...)` 或 `Events.Subscribe<LogSystemWeatherChangedEvent>(...)`

作用：本地天气变化时触发。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `WeatherId` | `uint` | 当前天气 ID（需查 Weather sheet 取名称） |

注意：只表达本客户端观察到的变化；初始化天气只建基线。

<a id="chatlog"></a>
### ChatLog

类型：`LogSystemChatLogEvent`（`LogSystemEventKind.ChatLog`）

订阅：`ctx.OnChatLog(...)` 或 `Events.Subscribe<LogSystemChatLogEvent>(...)`

作用：游戏消息框中的系统消息触发（对应 ACT 0x00 行）。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `Sender` | `string` | 发送者显示文本（聊天/台词类消息，其他子类型多为空） |
| `Message` | `string` | 消息显示文本 |
| `LogKind` | `string` | 聊天频道/日志类别文本 |

注意：不同语言客户端文本不同、格式不一致、措辞模糊且随时可能被调整，触发器应尽量避免使用此事件；副本台词优先用 `NpcYell`，机制信息优先用其他事件。

<a id="timelineexecution"></a>
### TimelineExecution

类型：`LogSystemTimelineExecutionEvent`（`LogSystemEventKind.TimelineExecution`）

订阅：`ctx.OnLogEvent<LogSystemTimelineExecutionEvent>(...)` 或 `Events.Subscribe<LogSystemTimelineExecutionEvent>(...)`

作用：时间轴条件或行为执行状态变化时触发（宿主内部事件）。

字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `NodeId` / `NodeName` | `int` / `string` | 时间轴节点 ID / 名称 |
| `ItemKind` | `TimelineExecutionItemKind` | 执行项类型（条件/行为） |
| `ItemIndex` | `int` | 执行项下标 |
| `ItemType` | `string` | 执行项类型名 |
| `State` | `TimelineExecutionState` | 执行状态（等待/成功/失败） |
| `Message` | `string` | 执行消息 |

注意：面向宿主时间轴调试与执行跟踪，一般业务无需订阅。
