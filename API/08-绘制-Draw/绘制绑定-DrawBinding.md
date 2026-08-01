# 绘制绑定 DrawBinding

命名空间：`PromeRotation.Spatial.Drawing`

作用：定义绘制元素如何跟随实体、朝向目标或延伸到目标。

## 在 DrawManager 中使用

方法：

```csharp
Plugin.Instance.DrawManager.AddBound(geometry, binding, ...)
Plugin.Instance.DrawManager.Update(handle, binding: binding)
Plugin.Instance.DrawManager.Update(id, binding: binding)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `binding` | `DrawBinding` | 要应用到绘制元素的动态绑定。 |

作用：为绘制元素设置动态位置、朝向或尺寸。

返回值：`AddBound` 返回 `DrawHandle`；`Update` 返回 `bool`。

说明：绑定会在每帧绘制时解析。计算绑定所需的实体不存在时，绘制元素暂不显示，但不会被移除。

## Follow

方法：

```csharp
DrawBinding.Follow(entity)
DrawBinding.Follow(entity, rotationOffset)
DrawBinding.Follow(entityId)
DrawBinding.Follow(entityId, rotationOffset)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `entity` | `IGameObject` | 要跟随的实体。 |
| `entityId` | `uint` | 要跟随的实体 ID。 |
| `rotationOffset` | `float` | 在实体朝向上增加的弧度偏移。可省略；默认为 `0`。 |

作用：使绘制元素跟随实体的位置和朝向。

返回值：`DrawBinding`。

说明：圆形和月环没有可见朝向，只会跟随位置。

## From

方法：

```csharp
DrawBinding.From(entity)
DrawBinding.From(entityId)
DrawBinding.From(position)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `entity` | `IGameObject` | 作为绑定起点的实体。 |
| `entityId` | `uint` | 作为绑定起点的实体 ID。 |
| `position` | `Vector3` | 作为绑定起点的固定世界坐标。 |

作用：指定 `LookAt` 或 `To` 的起点。

返回值：`DrawBindingBuilder`。需要继续调用 `LookAt` 或 `To` 才能得到 `DrawBinding`。

说明：`DrawBindingBuilder` 必须由 `DrawBinding.From()` 创建。

## LookAt

方法：

```csharp
DrawBinding.From(...).LookAt(target)
DrawBinding.From(...).LookAt(targetEntityId)
DrawBinding.From(...).LookAt(targetPosition)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `target` | `IGameObject` | 绘制元素要朝向的实体。 |
| `targetEntityId` | `uint` | 绘制元素要朝向的实体 ID。 |
| `targetPosition` | `Vector3` | 绘制元素要朝向的固定世界坐标。 |

作用：使绘制元素位于 `From` 起点并朝向目标。

返回值：`DrawBinding`。

说明：不会改变形状大小，只会计算起点到目标点的朝向；圆形和月环没有可见朝向，只会更新到 `From` 起点。

## To

方法：

```csharp
DrawBinding.From(...).To(target)
DrawBinding.From(...).To(targetEntityId)
DrawBinding.From(...).To(targetPosition)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `target` | `IGameObject` | 绘制元素要延伸到的实体。 |
| `targetEntityId` | `uint` | 绘制元素要延伸到的实体 ID。 |
| `targetPosition` | `Vector3` | 绘制元素要延伸到的固定世界坐标。 |

作用：使绘制元素从 `From` 起点朝目标延伸。

返回值：`DrawBinding`。

说明：会根据起点到目标点的距离自动计算几何大小；起点与目标重合时，绘制元素暂不显示。各几何图形的尺寸变化见 [几何图形接口 IGeometryShape](./几何图形接口-IGeometryShape.md#动态绑定行为)。

## 限制

- `WorldPolygonGeometry` 不支持动态绑定。
- `Update` 的 `binding: null` 表示不修改现有绑定，不能用于移除绑定。
