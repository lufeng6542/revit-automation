---
name: revit-automation
description: Revit MCP自动化 - 通过Revit MCP Server控制Revit 2018，支持模型查看、族放置、颜色标记、IronPython代码执行、视图截图导出、工作集同步等18种MCP操作。覆盖启动连接、模型浏览、可视化分析、构件放置、自定义代码、文档管理6大工作流。同义词：Revit、BIM、rvt、族、模型、revit操作、控制revit、revit自动化、打开revit、revit截图、放置族、BIM模型、color splash、revit代码
---

# Revit Automation -- Revit MCP 自动化

## 概述

通过 Revit MCP Server 控制 Autodesk Revit 2018。MCP 服务器（FastMCP）通过 pyRevit Routes HTTP API（端口 48884）与 Revit 进程通信。

**架构**: Claude → MCP 工具 (`mcp__Revit__*`) → HTTP → pyRevit Routes → Revit API

## 系统位置

- Revit MCP Server: `C:\Users\86183\revit-mcp-server\`
- pyRevit 扩展: `C:\Users\86183\AppData\Roaming\pyRevit\Extensions\mcp-server-for-revit-python.extension\`
- Revit 版本: 2018
- pyRevit 版本: 6.4.0

## 意图 → 工具映射

根据用户意图自动选择合适的 MCP 工具序列：

| 用户意图         | 工具序列                                                                        |
| ---------------- | ------------------------------------------------------------------------------- |
| 启动/连接 Revit  | `list_revit_installations` → `launch_revit`                                     |
| 查看模型概况     | `get_revit_model_info`                                                          |
| 截图导出视图     | `list_revit_views` → `get_revit_view`                                           |
| 浏览当前视图元素 | `get_current_view_info` → `get_current_view_elements`                           |
| 放置构件         | `list_families` → `list_levels` → `place_family`                                |
| 颜色标记分析     | `list_category_parameters` → `color_splash` → `get_revit_view` → `clear_colors` |
| 运行自定义代码   | `execute_revit_code`                                                            |
| 查看楼层         | `list_levels`                                                                   |
| 文档操作         | `open_document` / `save_document` / `close_document`                            |
| 工作集同步       | `sync_with_central`                                                             |

## 工作流 1：启动 Revit 并打开模型

```
1. list_revit_installations  → 检查可用版本
2. launch_revit              → 启动 Revit（可选传 file_path）
3. get_revit_status          → 确认 MCP 已就绪
4. open_document             → 如需工作共享选项（detach/audit）
```

## 工作流 2：模型浏览与截图

```
1. get_revit_model_info      → 项目名称、元素数、类别统计
2. list_revit_views          → 发现可用视图
3. get_revit_view            → 导出指定视图为 PNG
4. get_current_view_elements → 获取当前视图内元素列表
5. list_levels               → 了解楼层结构
```

## 工作流 3：可视化分析 (Color Splash)

按参数值为元素着色，快速识别模式：

```
1. list_category_parameters("Walls")    → 发现可用的着色参数
2. color_splash("Walls", "Type Name")   → 按参数值着色
3. get_revit_view("3D View")            → 截图保存结果
4. clear_colors("Walls")                → 清理恢复原色
```

## 工作流 4：族放置

```
1. list_families(contains="door")       → 搜索门族
2. list_family_categories               → 浏览可用类别
3. list_levels                          → 获取目标标高
4. place_family("Door", type="M_Single-Flush", x=0, y=0, level="Level 1")
```

## 工作流 5：自定义 IronPython 代码

直接在 Revit 进程中执行代码：

```
execute_revit_code(code="...", description="批量修改墙体参数")
```

### IronPython 关键注意事项

- **禁止 f-string** → 用 `"{} {}".format(a, b)` 替代
- **禁止类型注解** → 删除 `def func(x: int) -> str:` 中的类型
- **修改模型必须包裹 Transaction**：
  ```python
  t = DB.Transaction(doc, "修改说明")
  t.Start()
  # ... 修改模型 ...
  t.Commit()
  ```
- **可用对象**: `doc` (Document), `uidoc` (UIDocument), `DB` (Revit API), `revit` (pyRevit)
- **安全访问**: `getattr(element, 'Name', 'N/A')`

## 工作流 6：工作共享操作

```
1. open_document(file_path, detach=False)
2. sync_with_central(comment="修改说明", compact=False)
3. save_document() 或 save_document(file_path) 另存为
4. close_document(save=False)
```

## 18 个 MCP 工具速查

| 类别 | 工具                        | 用途                            |
| ---- | --------------------------- | ------------------------------- |
| 启动 | `launch_revit`              | 启动 Revit（可选指定文件/版本） |
| 启动 | `list_revit_installations`  | 发现已安装的 Revit 版本         |
| 状态 | `get_revit_status`          | 检查 MCP API 是否在线           |
| 状态 | `get_revit_model_info`      | 模型综合信息                    |
| 文档 | `open_document`             | 打开 RVT/RFA/RTE                |
| 文档 | `close_document`            | 关闭文档                        |
| 文档 | `save_document`             | 保存/另存为                     |
| 文档 | `sync_with_central`         | 与中心模型同步                  |
| 视图 | `list_revit_views`          | 列出所有可导出视图              |
| 视图 | `get_revit_view`            | 导出视图为图片                  |
| 视图 | `get_current_view_info`     | 当前视图详情                    |
| 视图 | `get_current_view_elements` | 当前视图内元素列表              |
| 族   | `list_families`             | 搜索族类型                      |
| 族   | `list_family_categories`    | 族类别列表                      |
| 放置 | `place_family`              | 在指定位置放置族实例            |
| 颜色 | `color_splash`              | 按参数值着色                    |
| 颜色 | `clear_colors`              | 清除颜色覆盖                    |
| 颜色 | `list_category_parameters`  | 类别可用参数                    |
| 执行 | `execute_revit_code`        | 执行 IronPython 代码            |
| 模型 | `list_levels`               | 获取所有标高                    |

## 故障处理

| 情况                | 处理                                           |
| ------------------- | ---------------------------------------------- |
| Revit 未运行        | 用 `launch_revit` 启动                         |
| MCP 无响应          | 检查 pyRevit Settings → Routes Server 是否启用 |
| IronPython 语法错误 | 去除 f-string、类型注解、async/await           |
| 工作共享对话框      | Revit 原生对话框无法被 MCP 绕过                |
| 端口冲突            | 多 Revit 实例使用顺序端口 (48884, 48885...)    |
