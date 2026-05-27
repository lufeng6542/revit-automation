# IronPython 编程指南

## IronPython vs CPython 关键差异

### 禁止使用的语法

- **f-string**: `f"Hello {name}"` → 用 `"Hello {}".format(name)`
- **类型注解**: `def func(x: int) -> str:` → 删除类型 → `def func(x):`
- **async/await**: IronPython 不支持异步
- **walrus operator**: `:=` 不可用
- **match/case**: 不可用

### 可用对象

执行环境提供：

- `doc` — 活动 Revit Document
- `uidoc` — 活动 UIDocument（UI 操作，如切换视图）
- `DB` — Autodesk.Revit.DB 命名空间
- `revit` — pyRevit 模块

### 安全属性访问

```python
name = getattr(element, 'Name', 'N/A')
param = element.LookupParameter('Mark')
value = param.AsString() if param else ''
```

### Transaction 管理

```python
t = DB.Transaction(doc, "操作说明")
t.Start()
# ... 修改模型 ...
t.Commit()
```

## 常用 Revit API 模式

### 收集元素

```python
# 所有墙体
walls = DB.FilteredElementCollector(doc).OfCategory(DB.BuiltInCategory.OST_Walls).WhereElementIsNotElementType().ToElements()

# 所有门族类型
doors = DB.FilteredElementCollector(doc).OfCategory(DB.BuiltInCategory.OST_Doors).WhereElementIsElementType().ToElements()

# 按类别收集
elements = DB.FilteredElementCollector(doc).OfClass(DB.FamilyInstance).ToElements()
```

### 获取/修改参数

```python
for wall in walls:
    param = wall.LookupParameter('Comments')
    if param and not param.IsReadOnly:
        param.Set('New Value')
```

### 切换活动视图

```python
all_views = DB.FilteredElementCollector(doc).OfClass(DB.View).ToElements()
target = next((v for v in all_views if v.Name == 'Level 1'), None)
if target:
    uidoc.ActiveView = target
```

### 获取标高

```python
levels = DB.FilteredElementCollector(doc).OfClass(DB.Level).ToElements()
for level in levels:
    print('{}: {}'.format(level.Name, level.Elevation))
```
