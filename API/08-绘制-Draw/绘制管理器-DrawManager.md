# DrawManager 绘制管理器

命名空间：`PromeRotation.Spatial.Drawing`

调用入口：`Plugin.Instance.DrawManager`

作用：管理绘制元素，包括添加、更新和移除。

## Add

方法：

```csharp
Plugin.Instance.DrawManager.Add(geometry, style, durationMs, startDelayMs, rendererType)
Plugin.Instance.DrawManager.Add(id, geometry, style, durationMs, startDelayMs, rendererType)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `id` | `string` | 绘制元素 ID。省略或留空时自动生成；相同 ID 会替换原绘制元素。 |
| `geometry` | `IGeometryShape` | 要绘制的几何图形。 |
| `style` | `DrawStyle?` | 绘制样式。可省略；默认为 `DrawStyle.Default`。 |
| `durationMs` | `float` | 显示时长，单位为毫秒。可省略；`0` 表示不自动过期。 |
| `startDelayMs` | `float` | 开始显示延迟时间，单位为毫秒。可省略；默认为 `0`。 |
| `rendererType` | `RendererType` | 渲染器类型。可省略；默认为 `Default`。 |

作用：添加一个静态绘制元素。

返回值：`DrawHandle`。可忽略；需要通过句柄更新或移除元素时保存。

## AddBound

方法：

```csharp
Plugin.Instance.DrawManager.AddBound(geometry, binding, style, durationMs, startDelayMs, rendererType)
Plugin.Instance.DrawManager.AddBound(id, geometry, binding, style, durationMs, startDelayMs, rendererType)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `id` | `string` | 绘制元素 ID。省略或留空时自动生成；相同 ID 会替换原绘制元素。 |
| `geometry` | `IGeometryShape` | 几何图形及初始尺寸。 |
| `binding` | `DrawBinding` | 绘制元素的位置、朝向或尺寸绑定。 |
| `style` | `DrawStyle?` | 绘制样式。可省略；默认为 `DrawStyle.Default`。 |
| `durationMs` | `float` | 显示时长，单位为毫秒。可省略；`0` 表示不自动过期。 |
| `startDelayMs` | `float` | 开始显示延迟时间，单位为毫秒。可省略；默认为 `0`。 |
| `rendererType` | `RendererType` | 渲染器类型。可省略；默认为 `Default`。 |

作用：添加一个随绑定目标动态变化的绘制元素。

返回值：`DrawHandle`。可忽略；需要通过句柄更新或移除元素时保存。

说明：支持绑定 `CircleGeometry`、`DonutGeometry`、`FanGeometry`、`RectangleGeometry`、`RayRectangleGeometry` 和 `LocalPolygonGeometry`。`WorldPolygonGeometry` 不支持绑定。

## AddTether

方法：

```csharp
Plugin.Instance.DrawManager.AddTether(start, end, style, width, endRadius, durationMs, startDelayMs, rendererType)
Plugin.Instance.DrawManager.AddTether(id, start, end, style, width, endRadius, durationMs, startDelayMs, rendererType)
```

起点与终点支持以下组合：

| 起点 | 终点 |
| --- | --- |
| `uint startEntityId` | `uint endEntityId` |
| `uint startEntityId` | `Vector3 endPosition` |
| `Vector3 startPosition` | `uint endEntityId` |
| `Vector3 startPosition` | `Vector3 endPosition` |
| `IGameObject start` | `IGameObject end` |
| `IGameObject start` | `Vector3 endPosition` |
| `Vector3 startPosition` | `IGameObject end` |

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `id` | `string` | 绘制元素 ID。省略或留空时自动生成；相同 ID 会替换原绘制元素。 |
| `start` | `uint`、`IGameObject` 或 `Vector3` | 连线起点。实体会动态更新位置；坐标保持固定。 |
| `end` | `uint`、`IGameObject` 或 `Vector3` | 连线终点。实体会动态更新位置；坐标保持固定。 |
| `style` | `DrawStyle?` | 连线样式。可省略；默认为 `DrawStyle.Default`。 |
| `width` | `float` | 连线宽度。可省略；默认为 `0.05`，必须大于 `0`。 |
| `endRadius` | `float` | 终点圆半径。可省略；默认为 `0.25`，`0` 表示不绘制终点圆。 |
| `durationMs` | `float` | 显示时长，单位为毫秒。可省略；`0` 表示不自动过期。 |
| `startDelayMs` | `float` | 开始显示延迟时间，单位为毫秒。可省略；默认为 `0`。 |
| `rendererType` | `RendererType` | 渲染器类型。可省略；`Default` 对连线等同于 `ImGui`。 |

作用：添加一个连接实体或世界坐标的连线绘制元素。

返回值：`DrawHandle`。可忽略；需要通过句柄更新或移除元素时保存。

说明：实体不存在时，连线暂不显示。

## Update

方法：

```csharp
Plugin.Instance.DrawManager.Update(handle, geometry, style, binding, durationMs, startDelayMs, rendererType)
Plugin.Instance.DrawManager.Update(id, geometry, style, binding, durationMs, startDelayMs, rendererType)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `handle` | `DrawHandle` | 要更新的绘制元素句柄。 |
| `id` | `string` | 要更新的绘制元素 ID。 |
| `geometry` | `IGeometryShape?` | 新几何图形。`null` 表示不修改。 |
| `style` | `DrawStyle?` | 新样式。`null` 表示不修改。 |
| `binding` | `DrawBinding?` | 新绑定。`null` 表示不修改。 |
| `durationMs` | `float?` | 新显示时长，并从更新时重新计时。`null` 表示不修改。 |
| `startDelayMs` | `float?` | 新的开始显示延迟时间。设置时必须同时设置 `durationMs`。 |
| `rendererType` | `RendererType?` | 新渲染器类型。`null` 表示不修改。 |

作用：更新一个已有绘制元素。

返回值：`bool`。更新成功为 `true`；绘制元素不存在、没有修改项或参数不受支持时为 `false`。

说明：Tether 不能通过此方法修改端点、宽度或终点圆半径。

## Remove

方法：

```csharp
Plugin.Instance.DrawManager.Remove(handle)
Plugin.Instance.DrawManager.Remove(id)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `handle` | `DrawHandle` | 要移除的绘制元素句柄。 |
| `id` | `string` | 要移除的绘制元素 ID。 |

作用：移除一个绘制元素。

返回值：`bool`。成功移除为 `true`；绘制元素不存在为 `false`。

## RemoveByRegex

方法：

```csharp
Plugin.Instance.DrawManager.RemoveByRegex(pattern)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `pattern` | `string` | 用于匹配绘制元素 ID 的正则表达式。 |

作用：移除所有 ID 匹配表达式的绘制元素。

返回值：`int`。被移除的元素数量。

## Clear

方法：

```csharp
Plugin.Instance.DrawManager.Clear()
```

参数：无。

作用：移除全部绘制元素。

返回值：无。

说明：战斗结束或切换区域时也会自动清空。

## 相关类型

| 类型 | 含义 |
| --- | --- |
| `IGeometryShape` | 可绘制几何图形的公共接口。 |
| `DrawBinding` | 绘制元素的动态绑定规则。 |
| `DrawStyle` | 填充颜色、描边颜色和描边粗细。 |
| `DrawStyle.Default` | 默认白色样式。 |
| `DrawStyle.Danger` | 危险区域样式。 |
| `DrawStyle.Safe` | 安全区域样式。 |
| `DrawHandle` | 绘制元素句柄。详见 [绘制句柄 DrawHandle](./绘制句柄-DrawHandle.md)。 |
| `RendererType` | 渲染器类型：`Default`、`ImGui` 或 `DirectX`。 |
