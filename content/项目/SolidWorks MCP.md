---
title: SolidWorks MCP
aliases: [SW MCP, 三维建模自动化, 控制 SolidWorks]
tags: [项目, SolidWorks, MCP, Python, CAD]
description: 让 AI 直接操控 SolidWorks 建三维模型的 MCP 服务器，56 个工具，实测通过 13 个建模用例
created: 2026-10-10
draft: false
---

# SolidWorks MCP

位置：`C:\Users\windows\Desktop\solidworks-mcp`
同类项目：[[AutoCAD MCP]]
建模规范依据：[[SolidWorks建模]]

## 为什么做这个

AutoCAD MCP 那套路子跑通了，SolidWorks 是同一个思路的第二次落地：
把只有 COM 接口、没有 API 的桌面软件，包成 MCP 工具给 AI 用。

二维画图和三维建模的差别在于——三维建模有**特征树**的概念。
AI 不只是"画线条"，而是要按建模规范组织特征顺序，否则改一个尺寸模型就崩。
所以这个项目不只是 API 封装，还把[[SolidWorks建模|建模规范]]固化进了工具设计。

## 原理

```mermaid
graph LR
    A[AI 客户端] -->|stdio JSON-RPC| B[MCP Server<br/>Python]
    B -->|队列投递| C[COM 专用 STA 线程]
    C -->|pywin32 COM| D[SolidWorks 2025]
    D --> E[特征树<br/>基体→功能→阵列→圆角]
```

关键设计：**所有 COM 调用必须发生在同一个 STA 线程**。
MCP 的同步工具会被丢进线程池，跨线程复用 COM 代理会崩。
桥接层起了一个专用 `ComThread`，调用用队列投递进去串行执行，还加了可重入保护（否则线程内再投递任务会自己等自己，实测死锁过）。

## 单位约定

| 项目 | 单位 |
|---|---|
| 长度/半径/深度 | 毫米 mm |
| 角度 | 度 |
| 质量 | kg |

SolidWorks API 内部一律用**米**，换算在桥接层完成。

## 能力清单（56 个工具）

- **文件**：新建零件/装配/工程图、打开、保存、另存、导出 STEP/STL/IGES、关闭
- **草图**：矩形、中心矩形、圆、直线、中心线、圆弧、正多边形、折线、清空
- **特征**：凸台拉伸、切除、贯穿切除、旋转、圆角、倒角、删除、改名
- **复杂建模**：射线拾取、面上开草图、偏移基准面、基准轴、圆周阵列、线性阵列、镜像、抽壳、全边倒圆角、改尺寸
- **属性视图**：材质、质量属性、包围盒、重建、视图切换、截图
- **一次成型**：长方体、圆柱、空心管、法兰、轴

## 建模规范是怎么固化进去的

从[[SolidWorks建模]]推出来的工具设计原则：

| 规范 | 对应的工具/行为 |
|---|---|
| 特征命名要人话 | `sw_rename_feature`，且 AI 靠名字引用特征做阵列/改尺寸，命名让流程可靠 |
| 重复特征用阵列 | `sw_circular_pattern` / `sw_linear_pattern` |
| 对称件画一半镜像 | `sw_mirror` |
| 圆角最后加 | 工具本身不强制顺序，但文档和提示词里写明 |
| 参考几何要稳定 | `sw_create_offset_plane` 提供全局坐标语义的任意高度基准面 |
| 参数化不硬编码 | `sw_set_dimension` 改尺寸并重建 |

> [!tip] 命名是刚需，不是洁癖
> 默认名 `凸台-拉伸1` / `切除-拉伸2` 在模型复杂后极易重号串台。
> 程序引用特征靠名字，串一次台整个流程就废了。

## 踩过的坑（精简版，完整版在项目 README.md）

### COM 层面

1. **大量 `Get*` 在 COM 里是属性不是方法**
   `GetTitle()` / `GetType()` / `FirstFeature()` / `CreateMassProperty()` / `CreateSelectData()` 全是属性。
   pywin32 的 `CDispatch` 自带 `__call__`，`callable()` 判断永远为真，没法用它区分。
   写了 `_g()` 取值器：带 `_oleobj_` 的一律当属性值返回。

2. **`SelectByID2` 的 Callout 参数**必须是 `VARIANT(VT_DISPATCH, None)`，传 `None`/`0`/`""` 一律类型不匹配。

3. **必须强制 `dynamic.Dispatch`**
   本机 `gen_py` 若生成过类型库，Dispatch 会走早期绑定，属性/方法语义和没生成过的机器不一致。

4. **COM 线程要防重入死锁**。

5. **几处返回值不可靠**：`SaveAs3`（按落盘判断，且它自己会改扩展名大小写）、`SetMaterialPropertyName2`（读回 `MaterialIdName` 验证）、`GetPartBox`（返回全 0，改用 `GetBodies2`+`GetBodyBox`）。

6. **MassProperty 必须 `UseSystemUnits=True`** 才是 SI。

7. **方法是属性还是方法说不准**：`InsertAxis2` / `InsertFeatureShell` 在 **IModelDoc2** 上，不在 FeatureManager。

### 建模层面

8. **单位是米不是毫米**，忘了 ×0.001 模型放大 1000 倍。

9. **切除默认方向和凸台相反**，容易切到空气里 → 失败自动反向重试，救活了所有 cut 用例。

10. **旋转轮廓不能跨过旋转轴**，必须画半剖面。第一版 `sw_make_shaft` 就画错了。

11. **旋转轴选不中**：草图段没有 `Name`，`SelectByID2` 按名字选不中。
    改为遍历找 `ConstructionGeometry=True` 的中心线 + `Select4(SelectData{Mark=16})`。

12. **mark 是 API 的隐藏规则**，传错静默失败：

    | 操作 | mark |
    |---|---|
    | 圆角/倒角的边 | 1 |
    | 抽壳开口面 | 1 |
    | 线性阵列方向1 / 方向2 | 1 / 2 |
    | 阵列被复制的特征 | 4 |
    | 镜像：镜像平面 / 被镜像特征 | 2 / 1 |
    | 旋转轴 | 16 |

13. **圆角在 2025 必须走 `CreateDefinition(1)+Initialize(0)+CreateFeature`**
    `FeatureFillet3`（14 参数）官方标 obsolete，实测走不通。
    三个枚举值（`swFmFillet=1`、`swConstRadiusFillet=0`、`swFeatureFilletCircular=0`）是运行时逐个试探 `CreateDefinition(0..19)` 找到的——JS 渲染的帮助页抓不到枚举值，**运行时探测反而最快**。

14. **射线拾取 `SelectByRay` 是 AI"用坐标点模型"的钥匙**
    AI 看不见屏幕，没法点鼠标。从一个坐标沿方向"开一枪"，命中什么选什么。
    - 选 **face** 时半径被忽略（无限细射线），随便打都能中
    - 选 **edge/vertex** 时半径才生效，默认给 2mm 容差，且**射线要瞄准棱边**（两个面的交线位置），瞄在面的中间高度打不中

## 实测结果

13 个用例全部通过，体积全部与理论值精确一致：

基础 7 个（长方体、圆柱、空心管、六棱柱、旋转轴、阶梯轴、法兰+导出）
复杂 6 个（顶面开孔、圆周阵列 6 孔、抽壳、偏移基准面开贯穿孔、改尺寸、全边倒圆角）
规范 3 个（特征重命名+用新名改尺寸、线性阵列 4 孔、镜像对称件）

脚本：`examples/smoke_all.py`、`examples/smoke_complex.py`、`examples/smoke_pattern.py`

## 下一步

还没做的：线性阵列的双向、扫描/放样、方程式与全局变量、装配配合、工程图标注。
套路已经趟熟，需要时补起来快。
