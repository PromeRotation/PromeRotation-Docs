# API 整理上下文

## 本轮目标

先建立对外文档骨架和 API 能力地图，不在本轮确定最终命名空间、签名或兼容性承诺。后续编写具体内容时，以本文件作为第一版探索基线。

## 调研范围

### 宿主

- `PromeRotation/Plugins`：插件元数据、同步/异步生命周期、时间轴扩展注册。
- `PromeRotation.Scripting` 与 `PromeRotation/Scripting`：脚本契约、上下文实现和运行会话。
- `PromeRotation/Rotation`、`Resolvers`、`Data`、`Managers`、`Updaters`：ACR 契约、决策、技能描述、队列和执行。
- `PromeRotation/Hosting`、`LogSystem`、`Network`：所有权、事件总线、战斗事件与网络事件。
- `PromeRotation/Spatial`：绘制、几何、安全区、动态绑定与求解。
- `PromeRotation/Timeline`、`PureTimeline`：时间轴节点、条件、动作、变量和运行时。

### 外部参考项目

- `D:/.Dev/.PromePlugin`：PromeDraw、PromeSafeZoneModule、PromeReceiver 等插件的真实调用方式。
- `D:/.Dev/.PromeScripts`：持续脚本、条件/动作脚本、事件订阅、朝向控制和绘制脚本。
- `D:/.Dev/MirukuACR`：基于 `PromeRotation.SDK.API15` 的轮转、决策器、起手、QT、热键和职业时间轴节点。

## 三类开发者需要的宿主能力

### 做一个插件

最先需要的是可预测的加载/卸载生命周期、宿主服务访问、配置 UI、事件订阅和资源清理。现有插件还会直接使用 `Svc`、`Plugin.Instance`、静态 Manager、Hook 与宿主内部类型，说明未来 SDK 需要提供更稳定的服务入口和明确的高级/不安全能力边界。

### 写一个脚本

脚本主要依赖 `IScriptContext`、三种脚本生命周期、游戏对象查询、共享黑板、日志、战斗事件，以及目标/朝向/技能/绘制等受控操作。现有 `ScriptContextHostingExtensions` 已经形成事件订阅与受控异步任务的雏形，适合作为脚本 API 的统一入口。

### 写一个 ACR

ACR 需要轮转元数据与注册、生命周期回调、GCD/oGCD/Always 决策、技能和状态查询、目标解析、动作队列、起手、QT、热键、职业量谱，以及职业专属时间轴节点。MirukuACR 中对 `Core`、`Data`、`Extensions`、`Helpers`、`Managers`、`Updaters` 的大量直接引用，表明这些能力需要收敛到较小、稳定且可版本化的 SDK 表面。

## 已观察到的边界问题

1. 宿主内部存在多个事件入口（LogSystem、CombatEventManager、静态 EventManager 等），外部 API 应统一订阅模型和事件数据结构。
2. 查询与修改混杂在静态单例和 Manager 中；后续应区分只读状态、动作请求与高风险强制操作。
3. 插件、脚本和 ACR 都可能创建长期资源；SDK 应提供 owner/scope，并保证卸载时自动取消订阅、任务、绘制、窗口和热键。
4. 外部项目仍依赖 ECommons、Dalamud 原生类型和宿主 DLL。需要逐项决定哪些类型可以作为契约透传，哪些需要 PromeRotation 自己的 DTO/接口。
5. ACR 已使用预览版 NuGet SDK，而插件和脚本多为直接引用 DLL。文档需分别说明包引用、运行时装载和版本兼容策略。
6. `PromeRotation.Scripting` 的注释称上下文“只读”，但接口实际包含黑板写入及游戏控制；后续文档应改为“受控执行上下文”，并标注每项副作用。

## 后续编写每类 API 时的固定模板

1. 能力范围与非目标。
2. 稳定入口（接口、服务或上下文属性）。
3. 类型与成员索引。
4. 生命周期、线程与所有权。
5. 错误、取消、超时和释放语义。
6. 插件、脚本、ACR 的适用性矩阵。
7. 最小示例与常见误用。
8. API 版本、废弃和迁移说明。

## 暂不纳入稳定 API 的内容

- 插件扫描器、下载器、自更新器和内部注册表的管理实现。
- Opcode、Hook、内存修改和原始包结构等底层实现细节；若将来开放，应单独作为高级/实验性能力。
- 具体窗口实现、调试面板、内部缓存和仅供宿主自身调用的 Helper。
