# Revit MCP 工具完整参考

## 启动工具

### launch_revit

启动 Revit，可选打开文件。

```
参数: file_path (str, optional), version (str, optional, e.g. "2018"), language (str, optional), timeout (int, default 120)
```

### list_revit_installations

发现所有已安装的 Revit 版本及路径。

## 状态工具

### get_revit_status

检查 MCP API 是否在线。返回状态和可用性。

### get_revit_model_info

获取当前模型综合信息：项目名称、文件路径、元素总数、类别列表、房间数、标高数等。

## 文档工具

### open_document

```
参数: file_path (str), detach (bool, default false), audit (bool, default false)
```

支持工作共享文件的 detach/audit 选项。

### close_document

```
参数: save (bool, default false)
```

### save_document

```
参数: file_path (str, optional) -- 无参数为 QSAVE，有参数为 Save As
```

### sync_with_central

```
参数: comment (str), compact (bool, default false), relinquish_all (bool, default true)
```

## 视图工具

### list_revit_views

列出所有可导出的视图。返回视图名称、类型、ID 列表。

### get_revit_view

导出指定视图为 PNG 图片。

```
参数: view_name (str)
```

### get_current_view_info

返回当前活动视图详情：名称、类型、比例、详细程度、裁剪框、视图族类型、规程、模板状态。

### get_current_view_elements

返回当前视图中可见元素列表。每个元素含 ID、名称、类别、类别 ID。

```
参数: limit (int, default 5000), include_levels (bool), include_location (bool)
```

## 族工具

### list_families

搜索族类型列表。

```
参数: contains (str, optional) -- 按名称过滤, limit (int, default 50)
```

### list_family_categories

获取所有族类别的列表。

### list_category_parameters

获取类别中元素可用的参数列表（用于 color_splash 参考）。

```
参数: category_name (str)
```

### place_family

在模型指定位置放置族实例。

```
参数: family_name (str, required), type_name (str, optional), x, y, z (float, default 0), rotation (float, default 0), level_name (str, optional), properties (dict, optional)
```

## 颜色工具

### color_splash

按参数值对类别中的元素着色。

```
参数: category_name (str), parameter_name (str), use_gradient (bool), custom_colors (list[str], optional)
```

### clear_colors

清除类别中元素的颜色覆盖。

```
参数: category_name (str)
```

## 代码执行工具

### execute_revit_code

在 Revit 上下文中执行 IronPython 代码。

```
参数: code (str), description (str, optional)
```

可用对象：`doc`, `uidoc`, `DB`, `revit`, `print`

## 模型工具

### list_levels

获取当前 Revit 模型中所有标高列表。
