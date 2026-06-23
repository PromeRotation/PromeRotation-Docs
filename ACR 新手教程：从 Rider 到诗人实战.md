# PromeRotation ACR 新手教程：从 Rider 到诗人实战

**适用版本**：本文基于当前仓库（2026-06-23）的 PromeRotation 外部 ACR 接口编写。

**读者画像**：你接触过编程（可能写过 Python / JS / Java），但**没写过 C#**，也**没写过游戏 ACR**。本文会带你从 Rider 新建项目开始，手把手写出一个能用的诗人 ACR。

---

## 你能做出什么

跟着本文一步步走，你会得到一个**完全自己写的诗人 ACR**，具备以下功能：

| 功能 | 说明 | 章节 |
|------|------|------|
| **打 1（BurstShot）** | 进战后自动使用爆发射击 | 第2章 🏆 |
| **触发 2（RefulgentArrow）** | BurstShot 有概率给 `HawksEye` buff，此时自动打辉煌箭 | 第3章 🏆 |
| **Resolver 模式重构** | 拆成多个小文件，用标准 Resolver 结构 | 第4章 |
| **Heartbreak 充能技能** | GCD 中间自动插入碎心箭，理解充能数和 oGCD 窗口 | 第5章 🏆 |
| **最小起手** | 开怪后自动执行 `1 → 旅神歌 → 常规 1/触发2` | 第6章 🏆 |
| **自定义歌轴 + 唱歌 QT** | 设置面板改歌序/时长，QT 开关控制自动唱歌 | 第7章 🏆 |
| **无目标切歌** | 不选中敌人时也能切歌（跑路途中） | 第8章 🏆 |
| **自动大地神 + 大地神 Hotkey** | 检测队友治疗 buff 自动放大地神，加手动热键按钮 | 第9章 🏆 |
| **Timeline 职业节点** | 进阶：让 Timeline 编辑器能控制歌序和时长 | 第10章 |

**每个章节都以「dotnet build → 进游戏验证」作为里程碑**。每完成一章你都能看到游戏里多了一个新功能。

---

## 0. 先理解 ACR 的加载规则

PromeRotation 的外部 ACR（Auto Combat Rotation）本质上是一个 **C# Class Library（类库）**，编译成 `.dll` 后放到指定目录，PromeRotation 就会在游戏里加载它。

几个**最容易被卡住的规则**：

1. **主类必须是 `public`、非抽象、有 `public` 无参构造函数**，并实现 `IRotation` 接口。
2. **必须标注四参数的元数据特性**：
   ```csharp
   [RotationMetadata((uint)Job.BRD, "显示名称", "作者名", "版本号")]
   ```
3. **输出目录名必须和 `RotationMetadata` 的 "作者名" 完全一致**。例如作者叫 `MyBard`，输出目录就必须叫 `MyBard`，DLL 名字建议叫 `MyBard.dll`。
4. **同一个职业同一个作者不能重复注册**。开发时只保留一个同名目录。
5. **输出路径示例**：
   ```
   %APPDATA%\XIVLauncherCN\pluginConfigs\PromeRotation\ACR\MyBard\MyBard.dll
   ```

本文里的 `MyBard` 就是作者名，也是目录名，也是 DLL 前缀。你可以换成自己的名字，但**三个地方必须保持一致**。

---

## 1. 用 Rider 新建项目并配置

### 1.1 新建 Class Library 项目

1. 打开 **JetBrains Rider**。
2. 点击 `New Solution`（或 `File → New → Solution`）。
3. 左侧选 `Class Library`（不是 Console App，不是 ASP.NET）。
4. Language 选 `C#`。
5. Solution Name 填 `MyBard`。
6. Location 填一个你习惯的代码目录，例如 `I:\repos\MyBard`。
7. Target Framework：选 `.NET 10.0`（如果列表里有 `net10.0-windows` 就直接选它；如果没有先选 `.NET 10.0`，后面手动改 `.csproj`）。
8. 勾选 `Place solution and project in the same directory`（不要多一层目录）。
9. 点击 **Create**。

创建后你会看到文件树里有一个 `MyBard.csproj` 和一个自动生成的 `Class1.cs`。**删掉 `Class1.cs`**。

### 1.2 配置 .csproj 文件

双击打开 `MyBard.csproj`，替换成以下内容：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    <CopyLocalLockFileAssemblies>false</CopyLocalLockFileAssemblies>

    <RootNamespace>MyBard</RootNamespace>

    <!-- ⚠️ 下面这些路径你要改成自己的 -->
    <DalamudReferenceRoot Condition="'$(DalamudReferenceRoot)' == ''">
      C:\Users\你的用户名\AppData\Roaming\XIVLauncherCN\addon\Hooks\dev
    </DalamudReferenceRoot>

    <PromeRotationReferenceRoot Condition="'$(PromeRotationReferenceRoot)' == ''">
      I:\repos\PromeRotation-1.0\PromeRotation\bin\x64\Debug
    </PromeRotationReferenceRoot>

    <!-- ⚠️ 输出目录最后一级 MyBard 必须和 [RotationMetadata] 的作者一致 -->
    <OutputPath>
      C:\Users\你的用户名\AppData\Roaming\XIVLauncherCN\pluginConfigs\PromeRotation\ACR\MyBard
    </OutputPath>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
  </PropertyGroup>

  <!-- 排除编译产物 -->
  <ItemGroup>
    <Compile Remove="**\bin\**;**\obj\**" />
    <EmbeddedResource Remove="**\bin\**;**\obj\**" />
    <None Remove="**\bin\**;**\obj\**" />
  </ItemGroup>

  <!-- 引用 PromeRotation 和 Dalamud 的 DLL -->
  <ItemGroup>
    <Reference Include="Dalamud">
      <HintPath>$(DalamudReferenceRoot)\Dalamud.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="Dalamud.Bindings.ImGui">
      <HintPath>$(DalamudReferenceRoot)\Dalamud.Bindings.ImGui.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="FFXIVClientStructs">
      <HintPath>$(DalamudReferenceRoot)\FFXIVClientStructs.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="Lumina">
      <HintPath>$(DalamudReferenceRoot)\Lumina.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="Lumina.Excel">
      <HintPath>$(DalamudReferenceRoot)\Lumina.Excel.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="ECommons">
      <HintPath>$(PromeRotationReferenceRoot)\ECommons.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="PromeRotation">
      <HintPath>$(PromeRotationReferenceRoot)\PromeRotation.dll</HintPath>
      <Private>false</Private>
    </Reference>
  </ItemGroup>
</Project>
```

> **为什么要改成自己的路径？**
>
> - `DalamudReferenceRoot`：游戏框架的 DLL 目录，每个人的 Windows 用户名不一样。
> - `PromeRotationReferenceRoot`：PromeRotation 的编译输出，如果仓库位置不同也要改。
> - `OutputPath`：ACR 安装目录，DLL 会复制到这里让游戏加载。

### 1.3 试构建

在 Rider 底部打开 Terminal（按 `Alt+F12`），输入：

```powershell
dotnet build
```

如果看到 `Build succeeded`，恭喜，环境通了。如果报找不到 DLL，回去检查路径。

> **🎯 小技巧**：打开 Windows 资源管理器，确认 `C:\Users\你的用户名\AppData\Roaming\XIVLauncherCN\addon\Hooks\dev` 下面真的有 `Dalamud.dll`。如果没有，搜一下你的 `Dalamud.dll` 在哪。

---

## 2. 🏆 里程碑 1：打出第一发 BurstShot

**目标**：项目能被 PromeRotation 加载，进战后角色自动打 `BurstShot`（爆发射击）。

**建议用 70 级以上诗人测试**；等级不够的话把技能 ID 换成你能用的 GCD。

### 2.1 理解 ACR 是怎么工作的

每一帧（约 60 次/秒），PromeRotation 依次调用你的三个方法：

```
NextAlways()   → 立即执行（无视 GCD）
NextGcd()      → GCD 转好时调用
NextOffGcd()   → GCD 转着时调用
```

想让角色自动打 BurstShot，只要在 `NextGcd()` 里返回一个指向 BurstShot 的 `PAction` 就行。

### 2.2 写第一个文件

在项目根目录新建 `BardRotation.cs`，粘贴以下代码：

```csharp
using ECommons.ExcelServices;
using PromeRotation.Data;
using PromeRotation.Managers;
using PromeRotation.Rotation;
using Core = PromeRotation.Core.Core;

namespace MyBard;

// 告诉 PromeRotation：这是诗人（BRD）的 ACR
[RotationMetadata((uint)Job.BRD, "新手诗人", "MyBard", "0.1.0.0")]
public sealed class BardRotation : IRotation
{
    // BurstShot（爆发射击）技能 ID。70+ 诗人默认 GCD。
    private const uint BurstShot = 16495;

    // 事件处理器——先给一个空实现
    private readonly IRotationEventHandler _eventHandler =
        new EmptyRotationEventHandler();

    // QT 列表——暂时没有任何开关
    public static IReadOnlyDictionary<string, bool> QtList { get; } =
        new Dictionary<string, bool>();

    // 起手列表——暂时没有
    public static IReadOnlyDictionary<string, Type> Openers { get; } =
        new Dictionary<string, Type>();

    public PAction? NextAlways() => null;

    // ★★★ 核心方法：GCD 决策 ★★★
    public PAction? NextGcd()
    {
        // 没有目标就不打
        if (Core.Target == null)
            return null;

        return new PAction(BurstShot, ActionType.Gcd, ActionTargetType.Target);
    }

    public PAction? NextOffGcd() => null;
    public IOpener? GetOpener() => null;
    public IRotationEventHandler GetEventHandler() => _eventHandler;

    public void DrawSettings() { }
    public void DrawQTs() { }

    // ★★★ Debug 面板：显示"为什么能/不能" ★★★
    public void UpdateDebugStatus()
    {
        RotationManager.GcdSolverStatus.Clear();
        RotationManager.OffGcdSolverStatus.Clear();
        RotationManager.AlwaysSolverStatus.Clear();

        RotationManager.GcdSolverStatus.Add(new SolverStatus
        {
            Name = "BurstShot",
            Success = Core.Target != null,
            Message = Core.Target == null ? "无目标" : "准备打 BurstShot"
        });
    }

    private sealed class EmptyRotationEventHandler : IRotationEventHandler
    {
        public void OnUpdate() { }
        public void OnOutOfBattleUpdate() { }
        public void OnBattleStarted() { }
        public void OnBattleUpdate() { }
        public void OnNoTarget() { }
        public void OnBattleEnded() { }
        public void OnTerritoryChanged(ushort territoryId) { }
    }
}
```

### 2.3 构建 + 进游戏验证

```powershell
dotnet build
```

**进游戏验证清单：**

| 步骤                           | 期望结果 |
|------------------------------|---------|
| 1. 打开 PromeRotation，切诗人      | 列表中显示 `新手诗人（MyBard / 0.1.0.0）` |
| 2. 选中木人、开怪               | 角色自动打 **BurstShot** |
| 3. 打开 PromeRotation Debug 面板 | GCD 状态显示 `BurstShot / Success=true` |

> **🎉 里程碑达成！** 你的第一行 C# 代码已经在游戏里跑起来了。这是写 ACR 最重要的第一步——你能控制一个 GCD 了。

---

## 3. 🏆 里程碑 2：打完 1 触发 2

**目标**：BurstShot 有概率给 `HawksEye` buff，拿到 buff 时自动打 `RefulgentArrow`（辉煌箭，技能 ID `7409`）。


### 3.1 逻辑分析

```
我身上有 HawksEye（buff ID 3861）或 Barrage（buff ID 128）？
  ├── 是 → 打 RefulgentArrow（2）
  └── 否 → 打 BurstShot（1）
```

### 3.2 改造 BardRotation.cs

**先在文件顶部加一个 using，再改两个方法**：`NextGcd()` 和 `UpdateDebugStatus()`。

在 `using PromeRotation.Rotation;` 后面加上：

```csharp
using PromeRotation.Extensions;
```

在 `BardRotation` 类的常量区加上：

```csharp
private const uint RefulgentArrow = 7409;
private const ushort HawksEye = 3861;
private const ushort Barrage = 128;
```

把 `NextGcd()` 替换成：

```csharp
public PAction? NextGcd()
{
    if (Core.Target == null)
        return null;

    var me = Core.Me;

    // 先检查 HawksEye，再检查 Barrage，然后落回基础 GCD。
    if (me != null &&
        me.Level >= 70 &&
        (me.HasStatus(HawksEye) || me.HasStatus(Barrage)))
    {
        return new PAction(RefulgentArrow, ActionType.Gcd, ActionTargetType.Target);
    }

    return new PAction(BurstShot, ActionType.Gcd, ActionTargetType.Target);
}
```

把 `UpdateDebugStatus()` 替换成：

```csharp
public void UpdateDebugStatus()
{
    RotationManager.GcdSolverStatus.Clear();
    RotationManager.OffGcdSolverStatus.Clear();
    RotationManager.AlwaysSolverStatus.Clear();

    var me = Core.Me;
    var canTwo = me != null &&
                 me.Level >= 70 &&
                 (me.HasStatus(HawksEye) || me.HasStatus(Barrage));

    RotationManager.GcdSolverStatus.Add(new SolverStatus
    {
        Name = canTwo ? "RefulgentArrow" : "BurstShot",
        Success = Core.Target != null,
        Message = Core.Target == null ? "无目标"
            : canTwo ? "有 buff → 打 2：RefulgentArrow"
                     : "无 buff → 打 1：BurstShot"
    });
}
```

### 3.3 构建 + 验证

```powershell
dotnet build
```

**进游戏验证：**

1. 打木人，没有 `HawksEye` 时一直打 **BurstShot**（1）。
2. BurstShot 触发 `HawksEye` 后（角色身上出现眼睛图标），**下一发 GCD 自动变成 RefulgentArrow**（2）。
3. 打完 RefulgentArrow buff 消失，下一发回到 BurstShot。
4. Debug 面板能看到状态从 `BurstShot` 切换到 `RefulgentArrow`。

> **🎉 里程碑达成！** 你已经学会了 ACR 最核心的思维模式：**读 Buff → 做决策 → 打不同技能**。复杂 ACR 里的大多数 Resolver 都建立在同样的思路上。

---

## 4. 用 Resolver 模式重构（结构不变，只看懂就够）

现在你只有两个技能，一个文件够用。但后面要加唱歌、大地神、设置面板……全塞在一个文件里会越来越乱。

常见做法是：把每个决策点拆成独立的 `IDecisionResolver`，在 `BardRotation` 的构造函数里按优先级注册。

```csharp
// 示例：BardRotation 构造函数里按优先级注册 Resolver
public BardRotation()
{
    _gcdResolvers.Add(new BardRefulgentArrowMaxGcd());  // 纷乱箭过期前→强制辉煌箭
    _gcdResolvers.Add(new BaseGcd());                    // 基础 GCD

    _offGcdResolvers.Add(new BardNaturesMinneOffGcd());  // 大地神
    _offGcdResolvers.Add(new BardSongAbility());         // 唱歌
    // ... 还有十几个
}
```

每个 Resolver 就两个方法：

```csharp
public interface IDecisionResolver
{
    CheckResult Check();      // "我能打吗？" → 成功/失败 + 原因
    PAction GetAction();      // "打什么？"   → 返回技能
}
```

**这一章里，我们要做三件事**：

1. 把技能 ID 和 Buff ID 拆到 `BRDSkill.cs` 和 `BRDBuff.cs` 里集中管理——之后改技能 ID 只需要改一处。
2. 把打1+触发2的逻辑做成 `BaseGcd.cs` Resolver，同时用 `.GetAdjustedActionId()` 处理低等级技能替换——拿低等级诗人测试也能自动适配。
3. `BardRotation.cs` 变简洁——只负责注册 Resolver 和转发调用。

**这一章不做新功能**，只是改变代码的组织方式。改完后功能和第3章完全一样（但低等级也能跑了）。

### 4.1 新建目录结构

```text
MyBard/
  BardRotation.cs          ← 重写：变简洁
  GlobalUsings.cs          ← 新增：全局 using
  Bard/
    Data/
      BRDSkill.cs          ← 新增：所有技能 ID 集中管理
      BRDBuff.cs           ← 新增：所有 Buff ID 集中管理
    Action/
      Gcd/
        BaseGcd.cs         ← 新增：从 BardRotation.cs 里拆出来的 GCD 逻辑
```

### 4.2 GlobalUsings.cs

在项目根目录新建 `GlobalUsings.cs`：

```csharp
// 全局 using —— 写一次，整个项目的 .cs 文件都能直接用
// 之后你在每个文件里写 Core.Me 就相当于 PromeRotation.Core.Core.Me
global using Core = PromeRotation.Core.Core;
global using Updaters = PromeRotation.Updaters;
```

> 💡 `global using` 是 C# 10 的语法。不熟悉 C# 没关系——你只需要知道，写了这个文件之后，其他 .cs 文件里可以直接写 `Core.Me` / `Core.Target`。

### 4.3 Bard/Data/BRDSkill.cs

新建 `Bard/Data/BRDSkill.cs`，把技能 ID 全部集中到这里：

```csharp
// 所有技能 ID 集中管理
// 好处：代码里写 BRDSkill.BurstShot 比写 16495 好懂一百倍
namespace MyBard.Bard.Data;

public class BRDSkill
{
    public const uint BurstShot = 16495;        // 爆发射击 —— 打1
    public const uint RefulgentArrow = 7409;    // 辉煌箭 —— 触发2
}
```

> 💡 后续章节加新技能时，只需要在这个文件里加一行 `public const uint` 就行了。

---

### 4.4 Bard/Data/BRDBuff.cs

新建 `Bard/Data/BRDBuff.cs`，把 Buff ID 集中到这里：

```csharp
// 所有 Buff ID 集中管理
namespace MyBard.Bard.Data;

public class BRDBuff
{
    public const ushort HawksEye = 3861;  // 鹰眼 —— 打1触发2的关键
    public const ushort Barrage = 128;    // 纷乱箭 —— 也会触发2
}
```

---

### 4.5 Bard/Action/Gcd/BaseGcd.cs（核心）

新建 `Bard/Action/Gcd/BaseGcd.cs`：

```csharp
using PromeRotation.Data;
using PromeRotation.Extensions;  // GetAdjustedActionId() 需要这个 using
using PromeRotation.Resolvers;
using MyBard.Bard.Data;

namespace MyBard.Bard.Action.Gcd;

// 基础 GCD Resolver：判断打 1 还是触发打 2
public class BaseGcd : IDecisionResolver
{
    public CheckResult Check()
    {
        if (Core.Target == null)
            return new CheckResult(false, "无目标");

        var me = Core.Me;
        if (me == null)
            return new CheckResult(false, "本地玩家为空");

        // 有触发 buff 就打 2
        if (me.Level >= 70 &&
            (me.HasStatus(BRDBuff.HawksEye) || me.HasStatus(BRDBuff.Barrage)))
        {
            return new CheckResult(true,
                $"打2：RefulgentArrow (HawksEye={me.HasStatus(BRDBuff.HawksEye)})");
        }

        return new CheckResult(true, "打1：BurstShot");
    }

    public PAction GetAction()
    {
        var me = Core.Me;
        if (me != null &&  // me.Level >= 70 &&  使用了 GetAdjustedActionId() 后，我们无需再手动检查玩家等级了
            (me.HasStatus(BRDBuff.HawksEye) || me.HasStatus(BRDBuff.Barrage)))
        {
            return new PAction(
                BRDSkill.RefulgentArrow.GetAdjustedActionId(),
                ActionType.Gcd, ActionTargetType.Target);
        }

        // ★★★ GetAdjustedActionId() 自动处理低等级替换 ★★★
        // BurstShot(16495) 在 70 级以下 → HeavyShot(97)
        // 这样你拿低等级诗人测，ACR 也能正常工作
        return new PAction(
            BRDSkill.BurstShot.GetAdjustedActionId(),
            ActionType.Gcd, ActionTargetType.Target);
    }
}
```

---

### 4.6 简化 BardRotation.cs

把旧 `BardRotation.cs` 的内容整个替换成：

```csharp
using ECommons.ExcelServices;
using MyBard.Bard.Action.Gcd;
using PromeRotation.Data;
using PromeRotation.Managers;
using PromeRotation.Resolvers;
using PromeRotation.Rotation;

namespace MyBard;

[RotationMetadata((uint)Job.BRD, "新手诗人", "MyBard", "0.1.0.0")]
public sealed class BardRotation : IRotation
{
    // ★★★ Resolver 列表：优先级 = 注册顺序 ★★★
    private readonly List<IDecisionResolver> _gcdResolvers = new();
    private readonly List<IDecisionResolver> _offGcdResolvers = new();

    public BardRotation()
    {
        // GCD：目前只有基础 GCD（打1/触发2）
        _gcdResolvers.Add(new BaseGcd());

        // oGCD：暂无，后面章节再加
    }

    public static IReadOnlyDictionary<string, bool> QtList { get; } =
        new Dictionary<string, bool>();

    public static IReadOnlyDictionary<string, Type> Openers { get; } =
        new Dictionary<string, Type>();

    private readonly IRotationEventHandler _eventHandler =
        new EmptyRotationEventHandler();

    public PAction? NextAlways() => null;

    public PAction? NextGcd()
    {
        // 按优先级逐个问：你能打吗？
        // 第一个 Check() 成功的 resolver 就执行它
        foreach (var resolver in _gcdResolvers)
        {
            if (resolver.Check().Success)
                return resolver.GetAction();
        }
        return null;
    }

    public PAction? NextOffGcd() => null;
    public IOpener? GetOpener() => null;
    public IRotationEventHandler GetEventHandler() => _eventHandler;
    public void DrawSettings() { }
    public void DrawQTs() { }

    public void UpdateDebugStatus()
    {
        RotationManager.GcdSolverStatus.Clear();
        RotationManager.OffGcdSolverStatus.Clear();
        RotationManager.AlwaysSolverStatus.Clear();

        foreach (var r in _gcdResolvers)
        {
            var result = r.Check();
            RotationManager.GcdSolverStatus.Add(new SolverStatus
            {
                Name = r.GetType().Name,
                Success = result.Success,
                Message = result.Message
            });
        }
    }

    private sealed class EmptyRotationEventHandler : IRotationEventHandler
    {
        public void OnUpdate() { }
        public void OnOutOfBattleUpdate() { }
        public void OnBattleStarted() { }
        public void OnBattleUpdate() { }
        public void OnNoTarget() { }
        public void OnBattleEnded() { }
        public void OnTerritoryChanged(ushort territoryId) { }
    }
}
```

### 4.7 构建 + 验证

```powershell
dotnet build
```

**进游戏验证：功能和第3章完全一样——打1+触发2正常运作。** Debug 面板里的 Resolver 名字不再是硬写的字符串，而变成了 `BaseGcd`（类的名字）。

> **构建通过了？** 很好！结构变了功能不变，说明拆成功了。后期加功能只需要新建 Resolver 文件并 `_gcdResolvers.Add(...)` 一行就行。

---

## 5. 🏆 里程碑 3：Heartbreak 充能技能

**目标**：在两个 GCD 中间自动插入 `Heartbreak`（碎心箭）。如果角色等级不够，示例会退回到 `Bloodletter`（失血箭），让你也能学习“充能技能”的写法。

这一章开始接触 oGCD。你要记住三个点：

1. oGCD 只能在 GCD 转圈期间插入，不能贴着 GCD 转好时硬塞。
2. 充能技能要看 `GetActionCharges()`，不是只看冷却。
3. Debug reason 要告诉新人“是没目标、窗口不足、没解锁、没层数，还是准备释放”。

### 5.1 本章要新建/修改的文件

```text
MyBard/
  Bard/
    Data/
      BRDSkill.cs                    ← 追加 Bloodletter + HeartBreak
    Action/
      OffGcd/
        HeartBreakAbility.cs         ← 新增：充能技能 Resolver
  BardRotation.cs                    ← 注册 oGCD Resolver
```

### 5.2 往 BRDSkill.cs 追加技能

在 `BRDSkill.cs` 里加上：

```csharp
public const uint Bloodletter = 110;       // 失血箭：低等级充能能力技
public const uint HeartBreak = 36975;      // 碎心箭：高等级替换后的充能能力技
```

### 5.3 Bard/Action/OffGcd/HeartBreakAbility.cs

新建 `Bard/Action/OffGcd/HeartBreakAbility.cs`：

```csharp
using MyBard.Bard.Data;
using PromeRotation.Data;
using PromeRotation.Extensions;
using PromeRotation.Helpers;
using PromeRotation.Resolvers;

namespace MyBard.Bard.Action.OffGcd;

// Heartbreak / Bloodletter 教学版：只演示“充能技能如何检查并插入”
public class HeartBreakAbility : IDecisionResolver
{
    public CheckResult Check()
    {
        if (Core.Target == null)
            return new CheckResult(false, "当前无目标");

        var actionId = GetUsableActionId();
        if (actionId == 0)
            return new CheckResult(false, "碎心箭/失血箭未解锁");

        var gcdRemainMs = ActionHelper.GetGcdRemain() * 1000f;
        if (gcdRemainMs <= 650f)
            return new CheckResult(false, $"GCD窗口不足:{gcdRemainMs:F0}ms");

        var charges = actionId.GetActionCharges();
        if (charges < 1f)
            return new CheckResult(false, $"充能不足:{charges:F1}层");

        return new CheckResult(true, $"充能可用:{charges:F1}层");
    }

    public PAction GetAction()
    {
        return new PAction(GetUsableActionId(), ActionType.OffGcd,
            ActionTargetType.Target);
    }

    private static uint GetUsableActionId()
    {
        // 高等级时 HeartBreak 可能由游戏自动替换；低等级时退回 Bloodletter。
        var adjusted = BRDSkill.HeartBreak.GetAdjustedActionId();
        if (adjusted != 0 && adjusted.IsActionAvailableByLevelAndQuest())
            return adjusted;

        return BRDSkill.Bloodletter.IsActionAvailableByLevelAndQuest()
            ? BRDSkill.Bloodletter
            : 0;
    }
}
```

> **为什么要用 `GetActionCharges()`？**
>
> 充能技能不是简单的“冷却好了/没好”。它可能有 0、1、2、3 层。`charges >= 1` 表示至少有一层可以用；后面做完整循环时，还会进一步考虑“满层防溢出”“团辅期保留”等更复杂策略。

### 5.4 改造 BardRotation.cs——注册 oGCD Resolver

先在 `BardRotation.cs` 顶部补上：

```csharp
using MyBard.Bard.Action.OffGcd;
```

构造函数里，在 GCD resolver 后面注册 Heartbreak：

```csharp
    public BardRotation()
    {
        // GCD：基础 GCD（打1/触发2）
        _gcdResolvers.Add(new BaseGcd());

        // oGCD：充能技能
        _offGcdResolvers.Add(new HeartBreakAbility());
    }
```

把 `NextOffGcd()` 从 `return null` 改成：

```csharp
public PAction? NextOffGcd()
{
    foreach (var resolver in _offGcdResolvers)
    {
        if (resolver.Check().Success)
            return resolver.GetAction();
    }
    return null;
}
```

在 `UpdateDebugStatus()` 里，GCD resolver 循环后面再加 oGCD resolver 的 Debug：

```csharp
foreach (var r in _offGcdResolvers)
{
    var result = r.Check();
    RotationManager.OffGcdSolverStatus.Add(new SolverStatus
    {
        Name = r.GetType().Name,
        Success = result.Success,
        Message = result.Message
    });
}
```

### 5.5 构建 + 验证

```powershell
dotnet build
```

**进游戏验证：**

1. 选中木人进战，确认打1+触发2仍然正常。
2. 等 `Bloodletter` / `Heartbreak` 至少有 1 层。
3. 观察角色会在 GCD 中间自动插入充能技能。
4. Debug 面板 oGCD 区能看到 `HeartBreakAbility`：
   - 没目标：`当前无目标`
   - GCD 快转好：`GCD窗口不足`
   - 没层数：`充能不足`
   - 可释放：`充能可用`

> **🎉 里程碑达成！** 你已经学会了第一个 oGCD：读充能层数 → 判断插入窗口 → 返回能力技。

---

## 6. 🏆 里程碑 4：最小起手

**目标**：开怪后自动执行一个非常短的起手：

```text
BurstShot → TheWanderersMinuet → 回到常规循环打 BurstShot/RefulgentArrow
```

这里故意不把第三个 GCD 写死在起手队列里。原因是 `IOpener.InCombatSequence` 是一串预先排好的动作，不能等第一发 GCD 打完后再动态判断是否触发 `HawksEye`。正确做法是：起手只排 `1 → 旅神歌`，队列结束后让常规 `BaseGcd` 接管下一发；如果第一发触发了 `HawksEye`，下一发自然就是 `RefulgentArrow`，否则就是 `BurstShot`。

### 6.1 本章要新建/修改的文件

```text
MyBard/
  Bard/
    Data/
      BRDSkill.cs                 ← 追加 TheWanderersMinuet
    Opener/
      SimpleBardOpener.cs         ← 新增：最小起手
  BardRotation.cs                 ← 注册起手列表并返回起手
```

### 6.2 往 BRDSkill.cs 追加旅神歌

```csharp
public const uint TheWanderersMinuet = 3559;   // 旅神歌
```

> 后面自动唱歌章节还会加入贤者歌和军神歌。这里先只加旅神歌，因为最小起手只需要这一首。

### 6.3 Bard/Opener/SimpleBardOpener.cs

新建 `Bard/Opener/SimpleBardOpener.cs`：

```csharp
using MyBard.Bard.Data;
using PromeRotation.Data;
using PromeRotation.Extensions;
using PromeRotation.Rotation;

namespace MyBard.Bard.Opener;

public class SimpleBardOpener : IOpener
{
    public string OpenerName => "最小起手：1-旅神-常规1/触发2";

    public void InitializeCountdown(CountDownHandler countdownHandler)
    {
        // 本教程先不做倒计时预读，只做进战后的最小起手。
    }

    public List<PAction> InCombatSequence => new()
    {
        BaseGcd(),
        new PAction(BRDSkill.TheWanderersMinuet, ActionType.OffGcd,
            ActionTargetType.Target)
    };

    private static PAction BaseGcd()
    {
        var me = Core.Me;
        if (me != null &&
            me.Level >= 70 &&
            (me.HasStatus(BRDBuff.HawksEye) || me.HasStatus(BRDBuff.Barrage)))
        {
            return new PAction(BRDSkill.RefulgentArrow.GetAdjustedActionId(),
                ActionType.Gcd, ActionTargetType.Target)
            {
                RequiresVerification = true
            };
        }

        return new PAction(BRDSkill.BurstShot.GetAdjustedActionId(),
            ActionType.Gcd, ActionTargetType.Target)
        {
            RequiresVerification = true
        };
    }
}
```

> **为什么 GCD 要设置 `RequiresVerification = true`？**
>
> 起手是一个动作队列。GCD 技能需要确认确实打出去了，队列才继续往下走；否则网络延迟或队列窗口问题可能让后面的旅神歌过早排进去。

### 6.4 改造 BardRotation.cs——注册起手

先补 using：

```csharp
using MyBard.Bard.Opener;
```

把 `Openers` 改成：

```csharp
public static IReadOnlyDictionary<string, Type> Openers { get; } =
    new Dictionary<string, Type>
    {
        { "最小起手：1-旅神-常规1/触发2", typeof(SimpleBardOpener) }
    };
```

把 `GetOpener()` 改成：

```csharp
public IOpener? GetOpener() => new SimpleBardOpener();
```

### 6.5 构建 + 验证

```powershell
dotnet build
```

**进游戏验证：**

1. 选中木人。
2. 确认 ACR 开启，并且当前没有正在执行的旧队列。
3. 开怪进入战斗。
4. 观察起手顺序：第一发 GCD → 旅神歌。
5. 起手队列结束后，常规 `BaseGcd` 接管下一发 GCD。
6. 如果第一发 GCD 触发 `HawksEye`，下一发应该变成 `RefulgentArrow`；没触发则继续 `BurstShot`。

> **🎉 里程碑达成！** 你已经写出了第一个起手。它很短，但已经包含了起手队列最重要的概念：GCD、oGCD、动作顺序、以及复用现有 GCD 决策。

---

## 7. 🏆 里程碑 5：自定义歌轴 + 唱歌 QT

**目标**：给 ACR 加上自动唱歌功能，并且：
- 在设置面板里可以调整三首歌的顺序和时长（歌轴）
- 用一个 QT 开关控制自动唱歌的开关

### 7.1 本章要新建/修改的文件

**从前面章节继承的文件（已存在，不需新建）**：

- `Bard/Data/BRDSkill.cs` → 本章需追加：贤者歌、军神歌、大地神
- `Bard/Data/BRDBuff.cs` → 本章无需追加（治疗 buff 在后面章节才加）

**本章新建**：

```text
MyBard/
  Bard/
    Data/
      BRDQt.cs               ← QT key 名称集中管理
      BardSettings.cs        ← 运行期设置（歌序、时长）
    Action/
      OffGcd/
        SongAbility.cs       ← 唱歌决策 Resolver（薄壳，委托给 BardSongHelper）
    BardSongHelper.cs        ← ★★★ 新增：切歌判断的核心逻辑 ★★★
    UI/
      BardSongUI.cs          ← 歌轴设置面板（ImGui 绘制）
```

### 7.2 往 BRDSkill.cs 追加新技能

在 `BRDSkill.cs` 里补全诗人教程用到的技能。此时文件应该长这样：

```csharp
namespace MyBard.Bard.Data;

public class BRDSkill
{
    // ---- 第4章已有的 ----
    public const uint BurstShot = 16495;
    public const uint RefulgentArrow = 7409;

    // ---- 第5章已有的 ----
    public const uint Bloodletter = 110;
    public const uint HeartBreak = 36975;

    // ---- 第6章已有的 ----
    public const uint TheWanderersMinuet = 3559;

    // ---- 本章新增 ----
    public const uint MagesBallad = 114;           // 贤者歌
    public const uint ArmysPaeon = 116;            // 军神歌
    public const uint NaturesMinne = 7408;         // 大地神（第9章用到）
}
```

---

### 7.3 BRDQt.cs

```csharp
// QT = Quick Toggle（快速开关）。在 PromeRotation 面板里展示为开关按钮
namespace MyBard.Bard.Data;

public class BRDQt
{
    public static string Song = "唱歌";
}
```

### 7.4 BardSettings.cs

```csharp
using Dalamud.Game.ClientState.JobGauge.Enums;

namespace MyBard.Bard.Data;

// 单例：全局只有一份运行期设置。
// 这一版教程先不做文件持久化；重启游戏后会回到默认值。
public class BardSettings
{
    public static BardSettings Instance { get; } = new();

    // 歌轴顺序（默认：旅神→贤者→军神）
    public Song FirstSong { get; set; } = Song.WanderersMinuet;
    public Song SecondSong { get; set; } = Song.MagesBallad;
    public Song ThirdSong { get; set; } = Song.ArmysPaeon;

    // 每首歌持续时长（单位：秒）
    public float WandererSongDuration { get; set; } = 42.6f;
    public float MageSongDuration { get; set; } = 39.2f;
    public float ArmySongDuration { get; set; } = 39.0f;

    public void ResetToDefault()
    {
        FirstSong = Song.WanderersMinuet;
        SecondSong = Song.MagesBallad;
        ThirdSong = Song.ArmysPaeon;
    }
}
```

### 7.5 BardSongHelper.cs（核心：切歌判断逻辑）

这是**整个自动唱歌的脑子**。`SongAbility`（oGCD Resolver）和后面的`BardNoTargetSongSwitcher`（无目标切歌）都调用这里。

这是教学版核心切歌模型：只保留 QT、ACR 状态、目标/GCD 窗口、歌序、歌时长和技能就绪检查。

```csharp
using Dalamud.Game.ClientState.JobGauge.Enums;
using MyBard.Bard.Data;
using PromeRotation.Data;
using PromeRotation.Extensions;
using PromeRotation.Helpers;

namespace MyBard.Bard;

public static class BardSongHelper
{
    // 统一入口：是否该切歌了？
    // allowNoTarget=true 时跳过目标和 GCD 窗口检查（allowNoTarget 给无目标切歌用）
    public static bool ShouldSwitchSong(out uint songActionId, out string reason,
        bool allowNoTarget = false)
    {
        songActionId = 0;
        reason = string.Empty;

        // 1. QT 检查
        if (!PromeSettings.Instance.GetQt(BRDQt.Song))
        {
            reason = "唱歌QT关闭";
            return false;
        }

        // 2. ACR 状态检查
        if (PromeSettings.Instance.EnableAcr != AcrState.On)
        {
            reason = "ACR未开启";
            return false;
        }

        // 3. 玩家状态检查
        var me = Core.Me;
        if (me == null || me.IsDead)
        {
            reason = "玩家不可用";
            return false;
        }

        // 4. 目标检查（allowNoTarget=true 时跳过，给无目标切歌用）
        if (!allowNoTarget && Core.Target == null)
        {
            reason = "当前无目标";
            return false;
        }

        // 5. GCD 窗口检查（allowNoTarget=true 时跳过）
        if (!allowNoTarget)
        {
            var gcdRemainMs = ActionHelper.GetGcdRemain() * 1000f;
            if (gcdRemainMs <= 550f)
            {
                reason = $"GCD窗口不足:{gcdRemainMs:F0}ms";
                return false;
            }
        }

        // 6. 读当前歌曲状态
        var settings = BardSettings.Instance;
        var currentSong = JobGaugeHelper.BRD.GetCurrentSong;
        var songTimerMs = JobGaugeHelper.BRD.GetCurrentSongTimer;

        // 7. 当前歌曲在歌轴里的位置
        int currentIdx = -1;
        if (currentSong == settings.FirstSong) currentIdx = 0;
        else if (currentSong == settings.SecondSong) currentIdx = 1;
        else if (currentSong == settings.ThirdSong) currentIdx = 2;

        if (currentIdx >= 0)
        {
            // 有歌 → 检查是否到切歌时长
            var duration = GetSongDuration(currentSong);
            var thresholdMs = 45000f - duration * 1000f;

            if (songTimerMs > thresholdMs)
            {
                reason = $"{Name(currentSong)}未到时长:{songTimerMs:F0}>{thresholdMs:F0}ms";
                return false;
            }

            // 下一首歌
            var nextSong = SongAt(settings, (currentIdx + 1) % 3);
            if (!IsSongReady(nextSong))
            {
                reason = $"{Name(nextSong)}冷却未就绪";
                return false;
            }

            songActionId = SpellId(nextSong);
            reason = $"切{Name(nextSong)}";
            return true;
        }

        // 8. 没唱歌 → 起第一首
        if (IsSongReady(settings.FirstSong))
        {
            songActionId = SpellId(settings.FirstSong);
            reason = $"起{Name(settings.FirstSong)}";
            return true;
        }

        reason = "无歌曲且第一首未就绪";
        return false;
    }

    // ---- 辅助方法 ----

    private static Song SongAt(BardSettings s, int idx) => idx switch
    {
        0 => s.FirstSong, 1 => s.SecondSong, 2 => s.ThirdSong,
        _ => Song.None
    };

    public static float GetSongDuration(Song song)
    {
        return song switch
        {
            Song.WanderersMinuet => BardSettings.Instance.WandererSongDuration,
            Song.MagesBallad => BardSettings.Instance.MageSongDuration,
            Song.ArmysPaeon => BardSettings.Instance.ArmySongDuration,
            _ => 0
        };
    }

    private static uint SpellId(Song song) => song switch
    {
        Song.WanderersMinuet => BRDSkill.TheWanderersMinuet,
        Song.MagesBallad => BRDSkill.MagesBallad,
        Song.ArmysPaeon => BRDSkill.ArmysPaeon,
        _ => 0
    };

    private static bool IsSongReady(Song song)
    {
        var id = SpellId(song);
        if (id == 0)
            return false;

        return id.IsActionAvailableByLevelAndQuest() &&
               id.GetActionCooldown() <= 0.05f;
    }

    private static string Name(Song song) => song switch
    {
        Song.WanderersMinuet => "旅神",
        Song.MagesBallad => "贤者",
        Song.ArmysPaeon => "军神",
        _ => "无"
    };
}
```

> **🎯 关键设计**：`allowNoTarget` 参数让同一份逻辑同时服务有目标（oGCD Resolver）和无目标（后续会讲到）两种场景。

---

### 7.6 Bard/Action/OffGcd/SongAbility.cs（薄壳）

现在 `SongAbility` 变得非常薄——只管调用 `BardSongHelper` 返回结果：

```csharp
using MyBard.Bard.Data;
using PromeRotation.Data;
using PromeRotation.Resolvers;

namespace MyBard.Bard.Action.OffGcd;

// 唱歌 oGCD Resolver — 薄壳，实际判断委托给 BardSongHelper
public class SongAbility : IDecisionResolver
{
    private uint _songActionId;

    public CheckResult Check()
    {
        if (!BardSongHelper.ShouldSwitchSong(out _songActionId, out var reason))
            return new CheckResult(false, reason);

        return new CheckResult(true, reason);
    }

    public PAction GetAction()
    {
        return new PAction(_songActionId, ActionType.OffGcd,
            ActionTargetType.Target);
    }
}
```

### 7.7 Bard/UI/BardSongUI.cs

```csharp
using Dalamud.Bindings.ImGui;
using Dalamud.Game.ClientState.JobGauge.Enums;
using MyBard.Bard.Data;

namespace MyBard.Bard.UI;

// 歌轴设置面板——在 PromeRotation 的设置标签页里画 UI
internal static class BardSongUI
{
    public static void DrawSongSettings()
    {
        var s = BardSettings.Instance;

        var first = s.FirstSong;
        if (DrawCombo("第一首", ref first)) s.FirstSong = first;

        var second = s.SecondSong;
        if (DrawCombo("第二首", ref second)) s.SecondSong = second;

        var third = s.ThirdSong;
        if (DrawCombo("第三首", ref third)) s.ThirdSong = third;

        if (ImGui.Button("重置默认歌序"))
            s.ResetToDefault();

        var w = s.WandererSongDuration;
        if (ImGui.SliderFloat("旅神歌时长", ref w, 3f, 45f, "%.1f秒"))
            s.WandererSongDuration = w;

        var m = s.MageSongDuration;
        if (ImGui.SliderFloat("贤者歌时长", ref m, 3f, 45f, "%.1f秒"))
            s.MageSongDuration = m;

        var a = s.ArmySongDuration;
        if (ImGui.SliderFloat("军神歌时长", ref a, 3f, 45f, "%.1f秒"))
            s.ArmySongDuration = a;
    }

    private static bool DrawCombo(string label, ref Song song)
    {
        if (!ImGui.BeginCombo(label, DisplayName(song))) return false;
        var changed = false;
        foreach (var c in new[] { Song.WanderersMinuet, Song.MagesBallad, Song.ArmysPaeon })
        {
            if (ImGui.Selectable(DisplayName(c), song == c)) { song = c; changed = true; }
            if (song == c) ImGui.SetItemDefaultFocus();
        }
        ImGui.EndCombo();
        return changed;
    }

    private static string DisplayName(Song s) => s switch
    {
        Song.WanderersMinuet => "旅神",
        Song.MagesBallad => "贤者",
        Song.ArmysPaeon => "军神",
        _ => "无"
    };
}
```

### 7.8 改造 BardRotation.cs——注册 QT、Resolver、设置面板

把 `BardRotation.cs` 的内容替换成：

```csharp
using Dalamud.Bindings.ImGui;
using ECommons.ExcelServices;
using MyBard.Bard.Action.Gcd;
using MyBard.Bard.Action.OffGcd;
using MyBard.Bard.Data;
using MyBard.Bard.Opener;
using MyBard.Bard.UI;
using PromeRotation.Data;
using PromeRotation.Managers;
using PromeRotation.Resolvers;
using PromeRotation.Rotation;

namespace MyBard;

[RotationMetadata((uint)Job.BRD, "新手诗人", "MyBard", "0.1.0.0")]
public sealed class BardRotation : IRotation
{
    private readonly List<IDecisionResolver> _gcdResolvers = new();
    private readonly List<IDecisionResolver> _offGcdResolvers = new();

    public BardRotation()
    {
        // oGCD：唱歌 → 充能技能
        _offGcdResolvers.Add(new SongAbility());
        _offGcdResolvers.Add(new HeartBreakAbility());

        // GCD：基础 GCD（打1/触发2）
        _gcdResolvers.Add(new BaseGcd());

        // ★★★ 注册 QT（必须！否则 SetQt/GetQt 无效）★★★
        foreach (var (name, def) in QtList)
            PromeSettings.Instance.AddQt(name, def);
    }

    // ★★★ QT 列表——唱歌 ★★★
    public static IReadOnlyDictionary<string, bool> QtList { get; } =
        new Dictionary<string, bool>
        {
            { BRDQt.Song, true },  // 唱歌：默认开
        };

    public static IReadOnlyDictionary<string, Type> Openers { get; } =
        new Dictionary<string, Type>
        {
            { "最小起手：1-旅神-常规1/触发2", typeof(SimpleBardOpener) }
        };

    private readonly IRotationEventHandler _eventHandler =
        new EmptyRotationEventHandler();

    public PAction? NextAlways() => null;

    public PAction? NextGcd()
    {
        foreach (var r in _gcdResolvers)
        {
            if (r.Check().Success) return r.GetAction();
        }
        return null;
    }

    public PAction? NextOffGcd()
    {
        foreach (var r in _offGcdResolvers)
        {
            if (r.Check().Success) return r.GetAction();
        }
        return null;
    }

    public IOpener? GetOpener() => new SimpleBardOpener();
    public IRotationEventHandler GetEventHandler() => _eventHandler;

    // ★★★ 设置面板——带「歌轴」标签页 ★★★
    public void DrawSettings()
    {
        if (!ImGui.BeginTabBar("Settings##MyBard"))
            return;

        if (ImGui.BeginTabItem("歌轴"))
        {
            BardSongUI.DrawSongSettings();
            ImGui.EndTabItem();
        }

        ImGui.EndTabBar();
    }

    public void DrawQTs() { }

    public void UpdateDebugStatus()
    {
        RotationManager.GcdSolverStatus.Clear();
        RotationManager.OffGcdSolverStatus.Clear();
        RotationManager.AlwaysSolverStatus.Clear();

        foreach (var r in _gcdResolvers)
        {
            var result = r.Check();
            RotationManager.GcdSolverStatus.Add(new SolverStatus
            {
                Name = r.GetType().Name,
                Success = result.Success,
                Message = result.Message
            });
        }

        foreach (var r in _offGcdResolvers)
        {
            var result = r.Check();
            RotationManager.OffGcdSolverStatus.Add(new SolverStatus
            {
                Name = r.GetType().Name,
                Success = result.Success,
                Message = result.Message
            });
        }
    }

    private sealed class EmptyRotationEventHandler : IRotationEventHandler
    {
        public void OnUpdate() { }
        public void OnOutOfBattleUpdate() { }
        public void OnBattleStarted() { }
        public void OnBattleUpdate() { }
        public void OnNoTarget() { }
        public void OnBattleEnded() { }
        public void OnTerritoryChanged(ushort territoryId) { }
    }
}
```

### 7.9 构建 + 验证

```powershell
dotnet build
```

**进游戏验证清单：**

| 步骤 | 期望 |
|------|------|
| 1. 进战，确认打1+触发2仍然正常 | BurstShot ↔ RefulgentArrow |
| 2. 打开 PromeRotation 设置 →「歌轴」标签页 | 看到歌序下拉框 + 时长滑块 |
| 3. 把歌序改成「贤者→军神→旅神」 | 歌曲按新顺序循环 |
| 4. 把旅神时长从 42.6 拖到 20.0 | 旅神只唱 20 秒就切 |
| 5. 点「重置默认歌序」 | 恢复旅→贤→军 |
| 6. 打开 PromeRotation QT 面板 | 看到「唱歌」开关（默认开） |
| 7. 关掉唱歌 QT | ACR 不再自动唱歌，但打1+触发2正常 |
| 8. 再打开唱歌 QT | 歌曲自动恢复 |
| 9. Debug 面板的 oGCD 区 | 能看到 `SongAbility` 行的成功/失败原因 |

> **🎉 里程碑达成！** 你的 ACR 现在：
>
> - ✅ 有完整的设置面板，可以调歌轴
> - ✅ 有 QT 开关，可以一键开关自动唱歌
> - ✅ 打1+触发2不受影响

---

## 8. 🏆 里程碑 6：无目标切歌

**目标**：不选中敌人时（例如跑路途中），ACR 仍然会自动切歌。

### 8.1 理解实现方式

这里用 `IRotationEventHandler.OnBattleUpdate()` 做一个独立检查：
1. 当前无目标？
2. ACR 已开启？
3. 需要切歌？

如果都满足，就用 `ActionUpdater.UseAction4Receiver()` 直接发送技能指令——不经过 NextGcd/NextOffGcd 管道。

### 8.2 新建 Bard/BardNoTargetSongSwitcher.cs

```csharp
using Dalamud.Game.ClientState.JobGauge.Enums;
using MyBard.Bard.Data;
using PromeRotation.Data;
using PromeRotation.Helpers;
using PromeRotation.Managers;

namespace MyBard.Bard;

// 无目标时自动切歌（不经过 NextOffGcd 管道）
public static class BardNoTargetSongSwitcher
{
    private const int ThrottleMs = 1000;      // 节流：每秒最多尝试1次
    private static long _lastAttemptTime;
    private static bool _lastStatus;
    private static string _lastMessage = "尚未检查";

    public static void TrySwitch()
    {
        if (!CanAttempt(out var blockedReason))
        {
            SetStatus(false, blockedReason);
            return;
        }

        // 全权交给 BardSongHelper，allowNoTarget=true 跳过目标/GCD窗口检查
        if (!BardSongHelper.ShouldSwitchSong(out var actionId, out var reason, allowNoTarget: true))
        {
            SetStatus(false, reason);
            return;
        }

        _lastAttemptTime = Environment.TickCount64;
        Updaters.ActionUpdater.UseAction4Receiver(actionId, ActionType.OffGcd, 0);
        SetStatus(true, $"无目标切歌完成");
    }

    public static void MarkOutOfBattle()
    {
        SetStatus(false, "非战斗状态");
    }

    public static SolverStatus GetStatus()
    {
        return new SolverStatus
        {
            Name = "NoTargetSongSwitch",
            Success = _lastStatus,
            Message = _lastMessage
        };
    }

    private static void SetStatus(bool success, string message)
    {
        _lastStatus = success;
        _lastMessage = message;
    }

    private static bool CanAttempt(out string reason)
    {
        if (PromeSettings.Instance.EnableAcr != AcrState.On)
        {
            reason = "ACR未开启";
            return false;
        }

        var me = Core.Me;
        if (me == null)
        {
            reason = "本地玩家为空";
            return false;
        }

        if (me.IsDead)
        {
            reason = "玩家死亡";
            return false;
        }

        if (Core.Target != null)
        {
            reason = "有目标，交给SongAbility";
            return false;
        }

        if (Environment.TickCount64 - _lastAttemptTime < ThrottleMs)
        {
            reason = "无目标切歌节流中";
            return false;
        }

        reason = string.Empty;
        return true;
    }
}
```

### 8.3 改造 BardRotation.cs——注册事件和 Debug

**这次只改两处**：

1. 事件处理器从 `EmptyRotationEventHandler` 换成有实际逻辑的 `BardRotationEventHandler`。
2. `UpdateDebugStatus()` 里加上 `NoTargetSongSwitch` 的展示。

修改 `BardRotation.cs`：

先在文件顶部补一行 using：

```csharp
using MyBard.Bard;
```

找到 `private sealed class EmptyRotationEventHandler`，把它替换成：

```csharp
    // ★★★ 事件处理器——触发无目标切歌 ★★★
    private sealed class BardRotationEventHandler : IRotationEventHandler
    {
        public void OnUpdate() { }

        public void OnOutOfBattleUpdate()
        {
            BardNoTargetSongSwitcher.MarkOutOfBattle();
        }

        public void OnBattleStarted() { }

        // ★★★ 每帧战斗更新时，尝试无目标切歌 ★★★
        public void OnBattleUpdate()
        {
            BardNoTargetSongSwitcher.TrySwitch();
        }

        public void OnNoTarget() { }
        public void OnBattleEnded() { }
        public void OnTerritoryChanged(ushort territoryId) { }
    }
```

然后改字段初始化那行，从 `new EmptyRotationEventHandler()` 改成 `new BardRotationEventHandler()`：

```csharp
    private readonly IRotationEventHandler _eventHandler =
        new BardRotationEventHandler();  // ← 改了这里
```

在 `UpdateDebugStatus()` 的末尾（`RotationManager.AlwaysSolverStatus.Clear();` 那几行之后）加上：

```csharp
    // 加上无目标切歌的状态
    RotationManager.OffGcdSolverStatus.Add(
        BardNoTargetSongSwitcher.GetStatus());
```

之后记得删掉空的 `EmptyRotationEventHandler` 类（它已经被 `BardRotationEventHandler` 取代了）。

### 8.4 构建 + 验证

```powershell
dotnet build
```

**进游戏验证：**

1. 确认唱歌 QT 开。
2. 进战，唱旅神歌。
3. 旅神歌快结束时（等约 40 秒），**按 Esc 取消目标**。
4. 观察：歌曲自动切换到贤者歌（即使没有目标）。
5. 打开 Debug 面板，看 oGCD 区最后一行 `NoTargetSongSwitch`：
   - 无目标且切歌成功：`Success=true / 无目标切歌完成`
   - 有目标时：`Success=false / 有目标，交给SongAbility`
6. 关掉唱歌 QT，无目标切歌也应该停掉：`Success=false / 唱歌QT关闭`

> **🎉 里程碑达成！** 你的 ACR 现在能在无目标时也自动维持歌曲循环了。这是在 boss 上天转场时的实用功能。

---

## 9. 🏆 里程碑 7：自动大地神 + 大地神 Hotkey

**目标**：
- 自动：检测队友有秘策（学者）/ 活化（贤者）/ 中间学派（占星）时，自动放大地神（NaturesMinne）
- Hotkey：在热键面板里有一个可点击的「大地神」按钮

### 9.1 本章要新建的文件

```text
MyBard/
  Bard/
    Action/
      OffGcd/
        AutoNatureMinne.cs    ← 新增：大地神自动决策
    UI/
      BardHotkeyUI.cs         ← 新增：热键面板
```

另外要在 `BRDBuff.cs` 和 `BRDQt.cs` 里加一行。

### 9.2 在 BRDBuff.cs 加上三个治疗 buff

```csharp
public class BRDBuff
{
    public const ushort HawksEye = 3861;
    public const ushort Barrage = 128;
    // ★★★ 新增：用于检测队友的治疗 buff ★★★
    public const ushort Recitation = 1896;    // 秘策（学者）
    public const ushort Zoe = 2611;           // 活化（贤者）
    public const ushort NeutralSect = 1892;   // 中间学派（占星）
}
```

### 9.3 在 BRDQt.cs 加上大地神 QT key

```csharp
public class BRDQt
{
    public static string Song = "唱歌";
    public static string NatureMinne = "自动大地神";  // ★★★ 新增
}
```

### 9.4 Bard/Action/OffGcd/AutoNatureMinne.cs

```csharp
using MyBard.Bard.Data;
using PromeRotation.Data;
using PromeRotation.Extensions;
using PromeRotation.Helpers;
using PromeRotation.Resolvers;

namespace MyBard.Bard.Action.OffGcd;

// 自动大地神：队友有秘策/活化/中间学派时，给自己放大地神提高受疗
public class AutoNatureMinne : IDecisionResolver
{
    public CheckResult Check()
    {
        if (!PromeSettings.Instance.GetQt(BRDQt.NatureMinne))
            return new CheckResult(false, "自动大地神QT关闭");

        if (BRDSkill.NaturesMinne.GetActionCooldown() > 0.05f)
            return new CheckResult(false,
                $"冷却中:{BRDSkill.NaturesMinne.GetActionCooldown() * 1000f:F0}ms");

        if (ActionHelper.GetGcdRemain() * 1000f <= 650f)
            return new CheckResult(false, "GCD窗口不足");

        var party = PartyHelper.GetUIParty();
        foreach (var member in party)
        {
            if (member.HasStatus(BRDBuff.Recitation) ||
                member.HasStatus(BRDBuff.Zoe) ||
                member.HasStatus(BRDBuff.NeutralSect))
            {
                return new CheckResult(true, $"大地神对齐 {member.Name}");
            }
        }

        return new CheckResult(false, "无可用治疗Buff对齐");
    }

    public PAction GetAction()
    {
        // 教学版先对自己使用，行为稳定、容易验证。
        // 如果你要精确给某个队友，需要再扩展目标选择逻辑。
        return new PAction(BRDSkill.NaturesMinne, ActionType.OffGcd,
            ActionTargetType.Self);
    }
}
```

### 9.5 Bard/UI/BardHotkeyUI.cs

```csharp
using MyBard.Bard.Data;
using PromeRotation;
using PromeRotation.Data;
using PromeRotation.UI.HotKey;

namespace MyBard.Bard.UI;

// 热键面板：在游戏里显示可点击按钮
internal static class BardHotkeyUI
{
    public static void SetupHotkeyPanel()
    {
        var panel = new HotkeyPanel(columns: 3);

        // 大地神按钮——点击就把大地神加入热键队列
        panel.AddHotkey("大地神",
            new PAction(BRDSkill.NaturesMinne, ActionType.OffGcd,
                ActionTargetType.Self));

        HotkeyManager.Instance.AddHotkeyPanel(panel);
    }
}
```

### 9.6 改造 BardRotation.cs——注册新 Resolver、QT、Hotkey

**改以下三处**：

1. 构造函数里加 `_offGcdResolvers.Add(new AutoNatureMinne());`
2. `QtList` 里加 `{ BRDQt.NatureMinne, true }`
3. 构造函数里加 `BardHotkeyUI.SetupHotkeyPanel();`

所以构造函数变成：

```csharp
    public BardRotation()
    {
        // oGCD：唱歌 → 大地神 → 充能技能
        _offGcdResolvers.Add(new SongAbility());
        _offGcdResolvers.Add(new AutoNatureMinne());
        _offGcdResolvers.Add(new HeartBreakAbility());

        // GCD：基础 GCD（打1/触发2）
        _gcdResolvers.Add(new BaseGcd());

        // 注册 QT
        foreach (var (name, def) in QtList)
            PromeSettings.Instance.AddQt(name, def);

        // 注册热键面板
        BardHotkeyUI.SetupHotkeyPanel();
    }
```

`QtList` 改成：

```csharp
    public static IReadOnlyDictionary<string, bool> QtList { get; } =
        new Dictionary<string, bool>
        {
            { BRDQt.Song, true },
            { BRDQt.NatureMinne, true },  // ← 新增
        };
```

### 9.7 构建 + 验证

```powershell
dotnet build
```

**进游戏验证清单：**

| 步骤 | 期望 |
|------|------|
| 1. 打开 PromeRotation QT 面板 | 看到「唱歌」「自动大地神」两个开关，都默认开 |
| 2. 组一个学者/占星/贤者队友进本 | — |
| 3. 队友下开秘策/活化/中间学派 | 你的角色立即自动放出大地神 |
| 4. Debug 面板 oGCD 区看 `AutoNatureMinne` | 等待时显示 `无可用治疗Buff对齐`，放时显示 `大地神对齐 玩家名` |
| 5. 关掉自动大地神 QT | 队友开治疗 buff 时不再放大地神 |
| 6. 打开 PromeRotation 热键面板 | 看到「大地神」按钮 |
| 7. 大地神可用时点击该按钮 | 大地神被加入执行队列，并在窗口内释放 |
| 8. 确认第7章（唱歌）和第8章（无目标切歌）不受影响 | 全部正常 |

> **🎉 所有里程碑全部达成！** 你的 ACR 现在具备完整的实用功能了。

---

## 10. 加 Timeline 职业节点（进阶）

**目标**：让 PromeRotation 的 Timeline 编辑器能通过节点控制歌序和时长。

> 这一步**不做新功能**，只是暴露第7章已有的设置给 Timeline 系统。做完后你可以在 Timeline 里写"某个机制后切换歌序"之类的自动化。

### 10.1 本章要新建的文件

```text
MyBard/
  Bard/
    Timeline/
      Actions/
        BardSongOrderAction.cs
        BardSongDurationAction.cs
    BardJobNodeProvider.cs
```

### 10.2 Bard/Timeline/Actions/BardSongOrderAction.cs

```csharp
using Dalamud.Game.ClientState.JobGauge.Enums;
using MyBard.Bard.Data;
using PromeRotation.Data;
using PromeRotation.Timeline.Core;

namespace MyBard.Bard.Timeline.Actions;

public sealed class BardSongOrderAction : IAction, ISerializableAction, IJobNodeDescriptor
{
    private int _firstSong = 1;   // 1=旅神 2=贤者 3=军神
    private int _secondSong = 2;
    private int _thirdSong = 3;

    public string NodeDisplayName => "Bard/歌曲顺序";

    public NodeParamInfo[] Params => new[]
    {
        new NodeParamInfo("first", "第一首歌", "1=旅神,2=贤者,3=军神",
            "int", new[] { ("1", "旅神"), ("2", "贤者"), ("3", "军神") }),
        new NodeParamInfo("second", "第二首歌", "1=旅神,2=贤者,3=军神",
            "int", new[] { ("1", "旅神"), ("2", "贤者"), ("3", "军神") }),
        new NodeParamInfo("third", "第三首歌", "1=旅神,2=贤者,3=军神",
            "int", new[] { ("1", "旅神"), ("2", "贤者"), ("3", "军神") }),
    };

    public string GetParam(string fieldName) => fieldName switch
    {
        "first" => _firstSong.ToString(),
        "second" => _secondSong.ToString(),
        "third" => _thirdSong.ToString(),
        _ => ""
    };

    public void SetParam(string fieldName, string value)
    {
        if (!int.TryParse(value, out var v) || v < 1 || v > 3) return;
        switch (fieldName)
        {
            case "first": _firstSong = v; break;
            case "second": _secondSong = v; break;
            case "third": _thirdSong = v; break;
        }
    }

    public void Execute()
    {
        if (_firstSong == _secondSong || _firstSong == _thirdSong || _secondSong == _thirdSong)
            return;

        var s = BardSettings.Instance;
        s.FirstSong = IntToSong(_firstSong);
        s.SecondSong = IntToSong(_secondSong);
        s.ThirdSong = IntToSong(_thirdSong);
    }

    public ActionDto ToDto() => new()
    {
        Type = "bardsongorder",
        Params = new Dictionary<string, string>
        {
            ["first"] = _firstSong.ToString(),
            ["second"] = _secondSong.ToString(),
            ["third"] = _thirdSong.ToString()
        }
    };

    public static void Register(RotationNodeContext context)
    {
        ActionFactory.Register(context, "bardsongorder", dto =>
        {
            var a = new BardSongOrderAction();
            if (dto.Params == null) return a;
            if (dto.Params.TryGetValue("first", out var f) && int.TryParse(f, out var fi))
                a._firstSong = Math.Clamp(fi, 1, 3);
            if (dto.Params.TryGetValue("second", out var s) && int.TryParse(s, out var si))
                a._secondSong = Math.Clamp(si, 1, 3);
            if (dto.Params.TryGetValue("third", out var t) && int.TryParse(t, out var ti))
                a._thirdSong = Math.Clamp(ti, 1, 3);
            return a;
        });
    }

    private static Song IntToSong(int v) => v switch
    {
        1 => Song.WanderersMinuet,
        2 => Song.MagesBallad,
        3 => Song.ArmysPaeon,
        _ => Song.None
    };
}
```

### 10.3 Bard/Timeline/Actions/BardSongDurationAction.cs

```csharp
using MyBard.Bard.Data;
using PromeRotation.Timeline.Core;

namespace MyBard.Bard.Timeline.Actions;

public sealed class BardSongDurationAction : IAction, ISerializableAction, IJobNodeDescriptor
{
    private float _wanderer = 42.6f;
    private float _mage = 39.2f;
    private float _army = 39f;

    public string NodeDisplayName => "Bard/歌曲时长";

    public NodeParamInfo[] Params => new[]
    {
        new NodeParamInfo("wanderer", "旅神歌时长", "3-45秒", "float"),
        new NodeParamInfo("mage", "贤者歌时长", "3-45秒", "float"),
        new NodeParamInfo("army", "军神歌时长", "3-45秒", "float"),
    };

    public string GetParam(string fieldName) => fieldName switch
    {
        "wanderer" => _wanderer.ToString("F1"),
        "mage" => _mage.ToString("F1"),
        "army" => _army.ToString("F1"),
        _ => ""
    };

    public void SetParam(string fieldName, string value)
    {
        switch (fieldName)
        {
            case "wanderer" when float.TryParse(value, out var v):
                _wanderer = Math.Clamp(v, 3f, 45f); break;
            case "mage" when float.TryParse(value, out var v):
                _mage = Math.Clamp(v, 3f, 45f); break;
            case "army" when float.TryParse(value, out var v):
                _army = Math.Clamp(v, 3f, 45f); break;
        }
    }

    public void Execute()
    {
        var s = BardSettings.Instance;
        s.WandererSongDuration = _wanderer;
        s.MageSongDuration = _mage;
        s.ArmySongDuration = _army;
    }

    public ActionDto ToDto() => new()
    {
        Type = "bardsongduration",
        Params = new Dictionary<string, string>
        {
            ["wanderer"] = _wanderer.ToString("F1"),
            ["mage"] = _mage.ToString("F1"),
            ["army"] = _army.ToString("F1")
        }
    };

    public static void Register(RotationNodeContext context)
    {
        ActionFactory.Register(context, "bardsongduration", dto =>
        {
            var a = new BardSongDurationAction();
            if (dto.Params == null) return a;
            if (dto.Params.TryGetValue("wanderer", out var w) && float.TryParse(w, out var fw))
                a._wanderer = Math.Clamp(fw, 3f, 45f);
            if (dto.Params.TryGetValue("mage", out var m) && float.TryParse(m, out var fm))
                a._mage = Math.Clamp(fm, 3f, 45f);
            if (dto.Params.TryGetValue("army", out var r) && float.TryParse(r, out var fr))
                a._army = Math.Clamp(fr, 3f, 45f);
            return a;
        });
    }
}
```

### 10.4 Bard/BardJobNodeProvider.cs

```csharp
using MyBard.Bard.Timeline.Actions;
using PromeRotation.Timeline.Core;

namespace MyBard.Bard;

public sealed class BardJobNodeProvider : IJobNodeProvider
{
    public void RegisterNodes(RotationNodeContext context)
    {
        BardSongOrderAction.Register(context);
        BardSongDurationAction.Register(context);
    }

    public IReadOnlyList<(string DisplayName, string Description, Func<ICondition> Create)> GetConditionDescriptors()
        => Array.Empty<(string, string, Func<ICondition>)>();

    public IReadOnlyList<(string DisplayName, string Description, Func<IAction> Create)> GetActionDescriptors()
        => new (string, string, Func<IAction>)[]
        {
            ("Bard/歌曲顺序", "设置旅神/贤者/军神的循环顺序",
                () => new BardSongOrderAction()),
            ("Bard/歌曲时长", "设置三首歌各自持续多久后切歌",
                () => new BardSongDurationAction()),
        };
}
```

### 10.5 在 BardRotation.cs 里注册 NodeProvider

在 `BardRotation` 类的顶部，和其他静态属性一起加一行：

如果 `BardRotation.cs` 顶部还没有这些 using，先补上：

```csharp
using MyBard.Bard;
using PromeRotation.Timeline.Core;
```

```csharp
    // ★★★ 告诉 PromeRotation 你的 Timeline 节点 ★★★
    public static IJobNodeProvider? NodeProvider { get; } =
        new BardJobNodeProvider();
```

### 10.6 构建 + 验证

```powershell
dotnet build
```

**进游戏验证：**

1. 打开 PromeRotation 的 Timeline 编辑器。
2. 添加一个 Action Node，在节点列表里应该能看到 `Bard/歌曲顺序` 和 `Bard/歌曲时长`。
3. 用 `Bard/歌曲顺序` 节点设置一个非默认歌序，确认执行后游戏内歌序变化。

---

## 11. 最终测试顺序

完成全部功能后，按这个顺序逐个验证，不要一次测所有：

| # | 测试项 | 怎么做 | 期望 |
|---|--------|--------|------|
| 1 | 编译 | `dotnet build` | `Build succeeded` |
| 2 | 加载 | 进游戏看 ACR 列表 | 看到 `新手诗人（MyBard / 0.1.0.0）` |
| 3 | 打1 | 进战 | 自动打 BurstShot |
| 4 | 触发2 | 等 HawksEye buff | GCD 变 RefulgentArrow |
| 5 | Heartbreak | 等碎心箭/失血箭有充能 | GCD 中间自动插入 |
| 6 | 最小起手 | 重新开怪 | 起手执行 `1 → 旅神歌`，之后常规循环接管 |
| 7 | 唱歌 | 唱歌 QT 开 | 自动循环三首歌 |
| 8 | 设置歌轴 | 设置页改歌序 | 切歌顺序变化 |
| 9 | 关唱歌 QT | QT 面板关掉唱歌 | 不自动唱歌 |
| 10 | 无目标切歌 | 战斗中按 Esc 取消目标 | 歌曲仍然自动切 |
| 11 | 自动大地神 | 和学者组队，等秘策 | 自动放大地神 |
| 12 | 大地神 Hotkey | 点热键面板的大地神按钮 | 技能排入队列并释放 |
| 13 | Timeline 节点 | 在 Timeline 加歌曲顺序节点 | 执行后歌序变化 |

---

## 12. 常见报错与解决

| 错误 | 原因 | 解决 |
|------|------|------|
| 找不到 PromeRotation.dll | `.csproj` 的 HintPath 不对 | 检查 `PromeRotationReferenceRoot` 路径 |
| ACR 不加载 | 输出目录名 ≠ 作者名 | OutputPath 最后一级目录要和 `[RotationMetadata]` 一致 |
| 找不到 IRotation | 主类没实现接口或不是 public | 确认 `BardRotation : IRotation` 是 `public sealed class` |
| SetQt 没效果 | QT key 没注册 | 确认在构造函数里调了 `AddQt` |
| Timeline 没职业节点 | 没加 NodeProvider 属性 | 确认有 `public static IJobNodeProvider? NodeProvider` |
| 技能不释放但 Debug 显示成功 | 等级不够 / 技能 ID 错 / 窗口不足 | 检查等级解锁和 GCD 窗口 |
| 同一包两个 BRD Rotation | 拆文件后残留了旧文件 | 删掉旧的 `BardRotation.cs` |

---

## 13. 下一步学什么

非常推荐你研究一下 Github 的自动 CI 流程，能节省很多时间
| GitHub Release 自动构建发布 | `https://github.com/kanyeishere/PRACR/blob/main/docs/github-release.md` |

**核心原则回顾：**

1. **一个 Resolver 只管一个决策点**——复杂 ACR 通常会有很多 resolver，每个只做一件小事。
2. **优先级靠注册顺序表达**——`_gcdResolvers.Add()` 的顺序就是优先级。
3. **Check() 只判断并返回原因**——Debug 面板一看就知道为什么没打。
4. **GetAction() 只返回动作**——判断都在 Check() 里做完了。

按这个思路继续加功能，你的 ACR 会越来越完整，而且 Debug 永远清晰。
