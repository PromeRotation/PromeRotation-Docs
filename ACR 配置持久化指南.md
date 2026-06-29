# PromeRotation ACR / 插件 配置持久化指南

**适用版本**：需 PromeRotation 含「配置持久化框架」的版本（`PromeRotation/Config/`，对应 PR #30 合并后的版本）。早于此版本的 PromeRotation 没有本功能。

---

## 你能得到什么

给配置字段贴一个 `[Persist]` 标注，框架就帮你：

| 能力 | 说明 |
|------|------|
| **自动保存** | 用户改了设置，框架自动落盘，你不用写 Save |
| **自动回填** | 下次进游戏，设置自动读回来，你不用写 Load |
| **换版本不丢** | PromeRotation 升级、你的 ACR 改版本号/改显示名，配置都还在 |
| **ACR / 插件统一** | 两者用完全一样的写法 |

---

## 三步接入

### 第 1 步：要保存的字段贴 `[Persist]`

```csharp
using PromeRotation.Config;

public sealed class MySettings
{
    [Persist] public int AoeEnemyCount = 3;     // 要保存 → 贴 [Persist]
    [Persist] public bool AutoPotion = false;
    [Persist] public int Mode = 0;

    public int RuntimeCounter;                   // 运行时临时变量 → 不贴 → 不保存
}
```

规则：
- **只对 `public` 字段/属性生效**。贴在非 public 成员上会被 JSON 静默丢弃 —— 框架检测到会在日志告警并跳过，提醒你改成 public。
- 属性需要 public 的 get 和 set。
- 字段类型要能 JSON 序列化（int / bool / float / string / 枚举 / 简单类 / List / Dictionary 等）。

### 第 2 步：ACR / 插件实现 `IPromeConfigurable`

```csharp
public class MyRotation : IRotation, IPromeConfigurable
{
    private MySettings _cfg = new();

    // 框架调用此方法拿到你的配置对象，之后自动回填 + 自动保存
    public object? GetPersistState() => _cfg;
}
```

插件一样：让你的 `IPromePlugin` 类同时实现 `IPromeConfigurable` 即可。

> 这个接口是**可选**的：不实现就和以前完全一样，不影响现有 ACR / 插件。

### 第 3 步：直接用配置

```csharp
if (敌人数量 >= _cfg.AoeEnemyCount) { /* 切 AOE */ }
```

- **加载时**：`_cfg` 里贴了 `[Persist]` 的字段已经被框架从存档回填好了。
- **运行时**：你改了 `_cfg` 的值，框架会通过低频轮询发现并自动保存（秒级）。

**你不需要写任何 Load / Save。**

---

## 进阶

### 想立即保存（不等轮询）

默认的自动保存是秒级轮询，通常够用。如果你想「点一下按钮立刻存」，调：

```csharp
PromeConfig.SaveNow(_cfg);   // 立即落盘，无需自己持有 store
```

### 多个配置类 / 分类型保存

一个 ACR 可以有多个配置对象，它们各自占一段，**分开保存互不覆盖**：

```csharp
var store = PromeConfig.ForAcr(author, jobId);
store.Save(qtConfig);          // 写入 "QtConfig" 段
store.Save(thresholdConfig);   // 写入 "ThresholdConfig" 段，不影响 QtConfig
```

只保存其中一类时，另一类原样保留。

---

## 配置存在哪

按**稳定身份**定位（和版本号、显示名、dll 文件名都无关，所以换版本/改名都不丢）：

```
pluginConfigs/PromeRotation/Configs/
├── acr/<Author>/<JobId>/config.json     # ACR：用 RotationMetadata 的 Author + JobId
└── plugin/<Id>/config.json              # 插件：用 PromePlugin 的 Id
```

`config.json` 内按 **段（section）** 组织，默认段名 = 配置类名，**只包含贴了 `[Persist]` 的成员**：

```json
{
  "MySettings": {
    "AoeEnemyCount": 3,
    "AutoPotion": false,
    "Mode": 0
  }
}
```

---

## ⚠️ 如果你的 ACR / 插件做混淆

`[Persist]` 成员名会作为 JSON 的 key。若你用 obfuscar 等工具混淆，**务必 Skip 掉配置类的字段/属性名**，否则成员被改名后 key 变化，用户已存的配置就读不回来了。

```xml
<SkipField type="你的命名空间.MySettings" name="*" />
<SkipProperty type="你的命名空间.MySettings" name="*" />
```

---

## 常见问题

**Q：我以前自己写了 Load/Save 到固定路径，要怎么迁移？**
A：接入后**只走框架**，不要再保留自己的写死路径读写（否则两份配置会互相覆盖）。把旧文件的数据搬到框架新位置一次，然后删掉自己的 Load/Save，改成 `GetPersistState() => _cfg`。

**Q：贴了 `[Persist]` 但没保存？**
A：检查是不是写在了非 public 字段上（看日志有没有告警）；以及该字段类型能否 JSON 序列化。

**Q：会不会和别的 ACR 配置冲突？**
A：不会。每个 ACR 按 `Author + JobId`、每个插件按 `Id` 独立分目录。
