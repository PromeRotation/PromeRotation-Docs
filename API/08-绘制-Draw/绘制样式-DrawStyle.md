# 绘制样式 DrawStyle

命名空间：`PromeRotation.Spatial.Drawing`

作用：定义绘制元素的填充颜色、描边颜色和描边粗细。

## 创建绘制样式

方法：

```csharp
new DrawStyle(fillColor, strokeColor, strokeThickness)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `fillColor` | `Vector4?` | 填充 RGBA 颜色。`null` 表示不填充。 |
| `strokeColor` | `Vector4?` | 描边 RGBA 颜色。`null` 表示不描边。 |
| `strokeThickness` | `float` | 描边粗细。可省略；默认为 `2`，必须大于 `0`。 |

作用：创建一个绘制样式。

返回值：`DrawStyle`。

说明：RGBA 分量必须是 `0` 到 `1` 的有限数值。

## Default

属性：

```csharp
DrawStyle.Default
```

参数：无。

作用：获取默认白色样式。

返回值：`DrawStyle`。

## Danger

属性：

```csharp
DrawStyle.Danger
```

参数：无。

作用：获取危险区域样式。

返回值：`DrawStyle`。

## Safe

属性：

```csharp
DrawStyle.Safe
```

参数：无。

作用：获取安全区域样式。

返回值：`DrawStyle`。

## 实例属性

| 属性 | 类型 | 含义 |
| --- | --- | --- |
| `FillColor` | `Vector4?` | 填充 RGBA 颜色。 |
| `StrokeColor` | `Vector4?` | 描边 RGBA 颜色。 |
| `StrokeThickness` | `float` | 描边粗细。 |

## 预设配置

| 属性 | 类型 | 默认值 | 含义 |
| --- | --- | --- | --- |
| `PresetFillAlpha` | `float` | `0.25` | 预设样式填充颜色的 Alpha 分量。 |
| `PresetStrokeAlpha` | `float` | `1` | 预设样式描边颜色的 Alpha 分量。 |
| `SafePresetColor` | `Vector3` | `(0, 1, 0)` | `Safe` 的 RGB 颜色。 |
| `DangerPresetColor` | `Vector3` | `(1, 0, 0)` | `Danger` 的 RGB 颜色。 |

作用：配置后续获取的预设样式。已创建的 `DrawStyle` 不受影响。
