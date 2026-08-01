# 几何图形接口 IGeometryShape

命名空间：`PromeRotation.Spatial.Geometry`

作用：作为 `DrawManager` 的几何图形参数，决定绘制元素的形状、位置和尺寸。

## 在 DrawManager 中使用

方法：

```csharp
Plugin.Instance.DrawManager.Add(geometry, ...)
Plugin.Instance.DrawManager.AddBound(geometry, binding, ...)
Plugin.Instance.DrawManager.Update(handle, geometry, ...)
Plugin.Instance.DrawManager.Update(id, geometry, ...)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `geometry` | `IGeometryShape` | 要添加或更新的几何图形。 |

作用：设置绘制元素的几何形状。

返回值：`Add` 和 `AddBound` 返回 `DrawHandle`；`Update` 返回 `bool`。

说明：`geometry` 需要使用下列具体类型创建。

## CircleGeometry

方法：

```csharp
new CircleGeometry(position, radius)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `position` | `Vector3` | 圆心的世界坐标。 |
| `radius` | `float` | 半径，必须大于 `0`。 |

作用：创建圆形几何图形。

返回值：`CircleGeometry`。

## DonutGeometry

方法：

```csharp
new DonutGeometry(position, innerRadius, outerRadius)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `position` | `Vector3` | 月环圆心的世界坐标。 |
| `innerRadius` | `float` | 内半径，必须大于或等于 `0`。 |
| `outerRadius` | `float` | 外半径，必须大于内半径。 |

作用：创建月环几何图形。

返回值：`DonutGeometry`。

## FanGeometry

方法：

```csharp
new FanGeometry(position, rotation, radius, angleRadians)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `position` | `Vector3` | 扇形圆心的世界坐标。 |
| `rotation` | `float` | 扇形中心朝向，使用 FFXIV Rotation 弧度。 |
| `radius` | `float` | 扇形半径，必须大于 `0`。 |
| `angleRadians` | `float` | 扇形总张角，单位为弧度，范围为 `(0, 2π]`。 |

作用：创建扇形几何图形。

返回值：`FanGeometry`。

## RectangleGeometry

方法：

```csharp
new RectangleGeometry(position, rotation, width, length)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `position` | `Vector3` | 矩形中心的世界坐标。 |
| `rotation` | `float` | 矩形朝向，使用 FFXIV Rotation 弧度。 |
| `width` | `float` | 矩形总宽度，必须大于 `0`。 |
| `length` | `float` | 矩形总长度，必须大于 `0`。 |

作用：创建一个以 `position` 为中心的矩形几何图形。

返回值：`RectangleGeometry`。

## RayRectangleGeometry

方法：

```csharp
new RayRectangleGeometry(position, rotation, width, length)
new RayRectangleGeometry(position, rotation, minLocalX, maxLocalX, length)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `position` | `Vector3` | 矩形后边中心的世界坐标。 |
| `rotation` | `float` | 矩形向前延伸的朝向，使用 FFXIV Rotation 弧度。 |
| `width` | `float` | 左右对称时的总宽度，必须大于 `0`。 |
| `minLocalX` | `float` | 左侧局部坐标边界。 |
| `maxLocalX` | `float` | 右侧局部坐标边界，必须大于 `minLocalX`。 |
| `length` | `float` | 向前延伸的长度，必须大于 `0`。 |

作用：创建一个从 `position` 向前延伸的矩形几何图形。

返回值：`RayRectangleGeometry`。

## LocalPolygonGeometry

方法：

```csharp
new LocalPolygonGeometry(position, rotation, localPoints)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `position` | `Vector3` | 多边形局部原点的世界坐标。 |
| `rotation` | `float` | 多边形朝向，使用 FFXIV Rotation 弧度。 |
| `localPoints` | `IEnumerable<Vector2>` | 相对局部原点的顶点。X 正方向为右侧，Y 负方向为前方。 |

作用：使用局部坐标顶点创建多边形几何图形。

返回值：`LocalPolygonGeometry`。

说明：至少需要三个有效顶点；边不能自相交。

## WorldPolygonGeometry

方法：

```csharp
new WorldPolygonGeometry(worldPoints)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `worldPoints` | `IEnumerable<Vector3>` | 多边形顶点的世界坐标。 |

作用：使用世界坐标顶点创建多边形几何图形。

返回值：`WorldPolygonGeometry`。

说明：至少需要三个有效顶点；边不能自相交。该类型不支持 `AddBound`。

## 动态绑定行为

`AddBound` 对不同几何图形的处理如下：

| 几何图形 | `Follow` | `LookAt` | `To` |
| --- | --- | --- | --- |
| `CircleGeometry` | 跟随位置 | 跟随位置 | 半径变为起点到目标的距离 |
| `DonutGeometry` | 跟随位置 | 跟随位置 | 保持内外半径 |
| `FanGeometry` | 跟随位置和朝向 | 朝向目标 | 半径变为起点到目标的距离 |
| `RectangleGeometry` | 跟随位置和朝向 | 朝向目标 | 中心移到两点中间，长度变为两点距离 |
| `RayRectangleGeometry` | 跟随位置和朝向 | 朝向目标 | 长度变为起点到目标的距离 |
| `LocalPolygonGeometry` | 跟随位置和朝向 | 朝向目标 | 保持局部顶点并朝向目标 |
| `WorldPolygonGeometry` | 不支持 | 不支持 | 不支持 |
