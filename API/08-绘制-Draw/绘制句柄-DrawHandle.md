# 绘制句柄 DrawHandle

命名空间：`PromeRotation.Spatial.Drawing`

作用：标识一个绘制元素，用于后续更新或移除对应元素。

## 获取句柄

方法：

```csharp
Plugin.Instance.DrawManager.Add(...)
Plugin.Instance.DrawManager.AddBound(...)
Plugin.Instance.DrawManager.AddTether(...)
```

参数：无。

作用：`Add`、`AddBound` 和 `AddTether` 都会返回 `DrawHandle`。不需要更新或移除元素时可以忽略返回值。

返回值：`DrawHandle`。

说明：`DrawHandle` 是值类型（`struct`），可以安全地存储和复制。

## Id

属性：

```csharp
drawHandle.Id
```

参数：无。

作用：获取绘制元素的 ID。

返回值：`string`。为创建元素时指定的 ID；未指定时是自动生成的 ID。

## IsValid

属性：

```csharp
drawHandle.IsValid
```

参数：无。

作用：判断句柄是否有效。

返回值：`bool`。ID 为空时为 `false`；有效句柄可用于更新或移除绘制元素。

## 在 DrawManager 中使用

方法：

```csharp
Plugin.Instance.DrawManager.Update(handle, ...)
Plugin.Instance.DrawManager.Remove(handle)
```

参数：

| 参数 | 类型 | 含义 |
| --- | --- | --- |
| `handle` | `DrawHandle` | 要更新或移除的绘制元素句柄。 |

作用：通过句柄更新或移除对应的绘制元素。

返回值：`Update` 和 `Remove` 都返回 `bool`。绘制元素不存在时为 `false`。
