---
title: AutoCAD MCP
aliases: [CAD 自动化, 控制 AutoCAD]
tags: [项目, AutoCAD, MCP, Python]
description: 让 AI 直接操控 AutoCAD 画图的 MCP 服务器，用 COM 自动化绕开文件格式问题
created: 2026-10-10
draft: false
---

# AutoCAD MCP

## 为什么要做这个

一开始的路子是「AI 生成 DXF 文件 → 用 AutoCAD 打开」。

走不通。

生成的 DXF 打开后一片黑，排查了一圈：缺 `$EXTMIN` / `$EXTMAX` 图形范围、缺实体句柄、`.scr` 脚本不能双击。

这些都能修，但思路本身就不对——**为什么要生成文件再让用户打开？**

换一条路：用 Windows 的 COM 接口，让 AI 直接往 AutoCAD 的模型空间里画。

## 原理

```mermaid
graph LR
    A[AI 客户端] -->|stdio JSON-RPC| B[MCP Server<br/>Python]
    B -->|pywin32 COM| C[AutoCAD 2025]
    C --> D[模型空间<br/>实体真的画进去了]

    style B fill:#4b6cb7,color:#fff
```

关键一行代码：

```python
acad = win32com.client.Dispatch("AutoCAD.Application")
doc = acad.ActiveDocument
doc.ModelSpace.AddLine((0, 0, 0), (100, 100, 0))
```

比生成文件强在：即时、可查询、可修改、不需要用户手动打开。

## 工具集

共 36 个，分八组：

| 组 | 工具 |
|---|---|
| 连接查询 | `autocad_connect` `autocad_status` `autocad_list_entities`（返回句柄） |
| **读回修正** | `get_entity`（按句柄读属性） `edit_entity`（erase/move/rotate/scale/layer） `check_annotations`（数值化查注记压盖） |
| 图层线型 | `create_layer` `set_layer` `create_layer_ex`（线型+线宽） `load_linetype` `set_var` |
| 绘图 | `draw_line` `draw_polyline` `draw_circle` `draw_arc` `draw_rectangle` `draw_text` `hatch_region` `hatch_pattern`（ANSI31 剖面线） |
| 尺寸标注 | `add_dim_rotated`（线性，数字随尺寸线转向） `add_dim_aligned` `add_dim_diameter` `add_dim_radial` |
| 国标环境 | `setup_gb_layers`（标准图层配色） `setup_gb_dim`（字高/箭头 1.5h/数字背景遮罩） `draw_sheet`（GB/T 14689 图幅+对中符号） `draw_title_block`（180×56 标题栏） `draw_roughness`（GB/T 131 粗糙度，可绕尖端旋转） `draw_gdt_frame`（GB/T 1182 框格+箭头引线） `draw_datum`（基准代号：实心三角+方框字母） |
| 表格视图 | `add_table`（真实 Table 对象） `zoom_extents` `send_command` `save_drawing` `clear_drawing` |
| 高层 | `draw_china_flag`、`gear_draft.py`（参数化齿轮零件图引擎） |

国标参数都来自知识库：[[机械制图国标速查]]、[[齿轮画法]]、[[表面粗糙度符号]]、[[几何公差标注]]。
画错的时候先查标准，再改代码——代码只是标准的翻译。

## 踩过的坑

> [!failure] mcp 2.x 装了不能用
> `pip install mcp` 默认装到 2.x，`FastMCP` 被改名成 `MCPServer`，导入路径也变了。
> 必须钉住 `mcp[cli]<2`。

> [!failure] 填充报「对象数组无效」
> `AddHatch` 的 `AppendOuterLoop` 要传 **VT_DISPATCH 数组**，传 Python list 会失败。
> 另外 `SOLID` 是预定义图案，类型参数要传 `1` 而不是 `0`。
> 正确写法：
> ```python
> loop = win32com.client.VARIANT(pythoncom.VT_ARRAY | pythoncom.VT_DISPATCH, [pl])
> h = doc.ModelSpace.AddHatch(1, "SOLID", True)
> h.AppendOuterLoop(loop)
> h.Evaluate()
> ```

> [!warning] 其他
> - 坐标要点转 VARIANT，部分版本直接传 tuple 会失败
> - `SendCommand` 的字符串末尾要加空格，否则命令不执行
> - 画完记得 `zoom_extents`，不然图形可能不在视野里

> [!failure] AutoCAD 2025 类型库和官方文档对不上（踩了一整轮）
> - `AddDimRotated` 文档说角度是**第一个**参数，实际 2025 签名是
>   `(ExtLine1, ExtLine2, DimLine, RotationAngle)` —— **角度在最后**
> - 直径标注不叫 `AddDimDiameter`，叫 `AddDimDiametric`，还多一个 `LeaderLength` 参数
> - `AddDimRadial` 只有 3 个参数，文字位置要事后设 `TextPosition` 属性
> - `IAcadLayer` 的颜色属性是**小写 `color`**（旧版 `Color`），装了 makepy 缓存后大小写敏感
> - 新建标注的 `TextHeight` / `ArrowheadSize` 默认 **0.18**（不是跟 DIMTXT 走），
>   不显式设置文字就小到看不见——表现为「标注线画了但没字」
> - pywin32 一旦生成 gen_py 缓存（`%TEMP%\gen_py`），Dispatch 全部变静态绑定，
>   属性名错一个字母就 `AttributeError`。排查签名直接看缓存里的 `IAcadModelSpace.py`
> - `DIMDEC` 系统变量对显示精度不生效 → 干脆所有标注都给 `TextOverride` 定死文字
> - **`MEASUREMENT=0`（英制模板）时 ANSI31 基础间距 0.125mm**，剖面线密成实心，
>   且调 PatternScale 也救不了（差 25.4 倍）——画毫米图前必须 `SetVariable("MEASUREMENT", 1)`
> - COM 偶发「被呼叫方拒绝接收呼叫」（AutoCAD 忙）→ 对绘图调用加重试即可

> [!failure] 标注 related 三连（第四轮排查出来的）
> - **静默降级最害人**：server.py 包装函数只认 `'x,y'` 字符串，脚本传元组
>   `(200, 248.6)` 时被悄悄忽略 → 形位公差的引线和箭头根本没画，还不报错。
>   修法：包装层坐标一律走 `_pt()` 容错转换（字符串/列表/元组都收），
>   解析不了就**抛错返回原因**，绝不悄悄放弃
> - **竖直尺寸的数字默认是水平的**：即使 `AddDimRotated` 传了 90°、DIMTIH=0，
>   `TextRotation` 读出来还是 0 → 一串横向数字全挤在中心线上互相压盖。
>   修法：按 GB/T 4458.4 方法 1 显式设 `dim.TextRotation`（官方可读可写）
> - **数字背景遮罩别用实体属性**：`TextFill=True + TextFillColor=0` 在 2025 里
>   会把数字填成一整块色块（0 被解释成随块色）。要走 `DIMTFILL=1`
>   （图样背景色遮罩，穿过的中心线自动在数字处断开）
> - **gen_py 静态绑定读不回属性**：`ModelSpace.Item()` 返回泛型 `IAcadEntity`，
>   取 `TextString` / `Coordinates` 直接 AttributeError。
>   修法：`win32com.client.dynamic.Dispatch(ent)` 换成动态分派再读

## 读回与修正回路（2026-10-10 加入）

参考 daobataotie/CAD-MCP（35 工具，「draw → inspect → correct」回路）的架构补齐三块：

1. **按句柄读回**：`autocad_list_entities` 返回句柄 → `get_entity` 按句柄读全部属性
   → `edit_entity` 局部 erase/move/rotate/scale，一处不对不用整图重画
2. **数值化自检**：`check_annotations` 用 TextPosition/TextHeight/显示字数估算
   「数字本身」的旋转矩形（不能拿整个标注的包围盒，尺寸界线会大量误报），
   SAT 求交报出**实际间隙**，代替肉眼看图
3. **截屏闭环**：`grab_cad.py` 置前窗口 + ImageGrab，最后一道防线

## 实测结果

AutoCAD 2025 上跑通：

- 连接、建图层、画线 / 圆 / 多段线 / 文字、缩放，全部正常
- `draw_china_flag(3000)` 一次出图：18 个实体，6 处填充全部成功

### 零件图复刻测试（2026-10-10）

拿两张真实的机械零件图截图做验证，看工具集够不够用。

| 图 | 脚本 | 实体 | 结果 |
|---|---|---|---|
| 轴架零件图 | `draw_part.py` | 31 | 成功（手敲坐标，粗糙版） |
| 直齿圆柱齿轮零件图 | `draw_gear.py` | 42 | 成功（手敲坐标，粗糙版） |
| 齿轮零件图 **参数化引擎重画** | `gear_draft.py` | 120 | 成功（公式推导 + 固定画图流程） |
| 同上，第四轮「标注重排」 | `gear_draft.py` | 144 | 成功（自检 0 压盖，数字竖排+背景遮罩） |

用户对第一版的批评一针见血：**「不能靠看，应该靠画图技巧准确确定，画图方法固定」**。
手敲坐标 = 目测描图，键槽深度、轮毂位置全是猜的。第二版改成参数化引擎：

- 齿轮几何全部由 `m`、`z` 公式推导：`d=mz`、`da=d+2ha*m=176`、`df=d-2(ha+c*)m=167`
- 键槽按 GB/T 1095 由孔径查表（孔 52 → 键 16×10，毂槽 `d+t2=56.3`）
- 固定画图流程：环境(图层/线型/CENTER 点划线) → A3 图框+标题栏 → 中心基准线 →
  左视图全剖（轮廓+C1/C2 倒角+ANSI31 剖面线，剖到齿根线为止）→
  右视图**局部视图**（波浪断裂线截断外圈圆，弧段相交点数值求解）→
  10 个真实 Dim 标注（含公差堆叠 `Ø52+0.03/0`）→ 参数表（Table 对象）→ 技术要求/粗糙度/形位公差
- 用 PIL 截屏 AutoCAD 窗口（`win32gui` 置前 + `ImageGrab`）做视觉闭环验证

> [!tip] 第三轮修正（用户对照原图挑出三处）
> 1. 剖面线太密 → ANSI31 `PatternScale` 0.75→1.25（先查实体属性确认赋值生效）
> 2. 左剖视外轮廓画成断开 → 原图是**贯通闭合矩形**（空刀区为内部白区），端面线改整条
> 3. 右视图画成完整圆 → 原图是**局部视图**：波浪线截断 da/df/分度/轮缘/分布圆，
>    每个圆与波浪线的交点用数值法求解后画左侧长弧，内孔/轮毂保留整圆
>
> 教训：**画完必须截屏放大自查一遍再交付**，肉眼过一遍能拦住大半这类问题。

> [!tip] 第四轮修正（用户指出尺寸/粗糙度标注重叠）
> 1. 根因：竖直尺寸数字是水平的，五个直径数字全挤在水平中心线上。
>    修复 = 数字按 GB 转到与尺寸线同向（`TextRotation` 显式设置）+ 间距 8→10mm
> 2. 数字被点划线穿过 → `DIMTFILL=1` 背景遮罩
> 3. 技术要求从左上角挪到左下标题栏上方（左上角留给尺寸列和粗糙度符号）
> 4. 发现并修复「引线静默丢失」：包装层只认字符串，元组参数被悄悄丢弃
> 5. 验收不靠看：`check_annotations` 数值自检，39 个数字两两间隙 > 2mm 才算过
>
> 教训：**包装层的容错转换要「解析不了就报错」，静默降级比报错可怕得多。**

> [!abstract] 顺带发现：原图参数表自相矛盾
> 那张课程设计图纸（爱给网下载）参数表写 `z=27`，但 `Ø172 = 2×86` 只有 z=86 才与
> `Ø176 齿顶`、`a=113=(86+27)×2/2` 自洽。图上 `R58.35` 的虚线圆在标准画法里应是分度圆 R86。
> 结论：网上下载的图纸不能照抄，按公式重算。

## 项目位置

```
C:\Users\windows\Desktop\autocad-mcp\     ← 2026-10-10 移到桌面（与 solidworks-mcp 并排）
├── server.py              MCP 服务器（36 个工具定义）
├── autocad_bridge.py      COM 桥接层（实际操作 AutoCAD）
├── gear_draft.py          参数化齿轮零件图引擎（改参数字典即可画任意齿轮）
├── draw_part.py           轴架零件图（旧版手敲坐标，留档）
├── draw_gear.py           齿轮零件图（旧版手敲坐标，留档）
├── grab_cad.py            截屏 AutoCAD 窗口（视觉验证闭环）
├── test_connection.py     冒烟测试
├── requirements.txt       mcp[cli]<2 + pywin32 + pillow
├── README.md
└── 客户端配置示例.md
```

配置在 `~/.workbuddy/mcp.json`，其他客户端见项目里的配置示例。

## 可以继续做的

- 参数化图形库：齿轮、螺栓、法兰、标准件
- 标注尺寸（`AddDimRotated` 之类）
- 读取现有图纸并做统计分析
- 批量出图 / 批量转 PDF

## 相关

- [[Windows 回收站]] — 同样是 Windows 上的操作技巧
- [[第二大脑站点]] — 另一个在做的项目
