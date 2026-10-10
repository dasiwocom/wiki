---
title: AutoCAD 进阶技法
aliases: [AutoCAD, 动态块, 图纸集, XREF, 参数化约束]
tags: [领域, CAD, AutoCAD, 机械制图, 方法论]
description: 2D 出图的生产级做法——样板文件、动态块与参数化、外部参照、模型/图纸空间、图纸集与批量发布
created: 2026-10-10
draft: false
---

# AutoCAD 进阶技法

> [!note] 核心差别
> 业余把 AutoCAD 当电子绘图板，画的是孤立的线；
> 职业画的是**图层 + 块引用 + 参数化约束构成的动态系统**。
> 区别在改图时暴露：前者牵一发动全身，后者改一处全部同步。

## 一、样板文件（.dwt）：一切的起点

样板是"没有图形对象、只有环境设置"的空白文件。一份机械样板至少含：

- **图层**：粗实线 / 细实线 / 中心线 / 虚线 / 尺寸线 / 文字，颜色线型线宽按 [[机械制图国标速查]]
- **文字样式**：数字用 `gbeitc.shx`、汉字用 `gbenor.shx` 或 `gbcbig.shx`，宽高比 0.7（汉字约 2/3 字高）
- **标注样式**：一个"机械"父样式 + 若干子样式

| 子样式 | 用途 |
|---|---|
| 角度 | 角度尺寸数字保持水平 |
| 直径 / 半径 | 数字水平 |
| 非圆直径 | 任何尺寸前自动加 Ø |
| 标注一半 | 半剖图用，只显示一半尺寸线和界线 |

父样式关键参数：基线间距 7~10、超出尺寸线 2~2.5、起点偏移量 **0**、箭头大小 3~4、文字高 3.5、精度 0。

- **图框与标题栏**按 1:1 画
- **粗糙度符号做成带属性（RA 值为属性）的内部图块**

> [!important] 全局比例别乱
> 图形永远 **1:1 画在模型空间**。图纸比例靠三处同步：图框缩放、标注样式「使用全局比例」、线型管理器「全局比例因子」——三处填**同一个值**。
> 只改一处，虚线会变成实线，或者文字忽大忽小。

## 二、动态块：一个块顶十个

在块编辑器里加**参数**（线性、角度、可见性）+ **动作**（拉伸、旋转、翻转），
一个带线性参数的螺栓块能替代 10 个不同长度的螺栓块。

进阶组合：

- **块属性表（Block Properties Table）**：定义 `d1`、`d2` 等预设值，插入后夹点下拉即可切换标准规格（Ø5 / Ø8 / Ø10…）
- **可见性状态**：同一个块里存螺栓的不同头型、不同长度，按状态切换
- **约束放进块里**：块编辑器里加几何/尺寸约束（相等、平行、同心、相切），让块自己保持形状——六角螺母就这么做

> [!tip] 属性（Attribute）用来带数据
> 图号、材料、表面处理做成属性，可以设不可见、常量、预设、锁定位置。
> 之后用**数据提取**把全图的块属性导成 BOM 表或 Excel。

## 三、参数化约束（不是动态块，是另一套）

几何约束（水平/垂直/同心/相切/对称）+ 尺寸约束，直接加在模型空间的图形上。
尺寸之间可以写**表达式**（`长度 = 宽度 × 2`），改一个另一个跟着变。

适合方案比选阶段：定关系，不填死数——这是"设计意图"在 2D 里的等价物。

## 四、外部参照 XREF vs 插入块

| | INSERT | XATTACH (XREF) |
|---|---|---|
| 本质 | **复制**几何进当前文件 | **链接**源文件 |
| 源改了 | 当前文件不更新 | 所有引用它的图自动更新 |
| 体积 | 变大 | 几乎不变 |

装配图必须用 XREF：改了零件图，装配图自动跟上。用 INSERT 就等于断了更新链。

## 五、模型空间 vs 图纸空间

- **模型空间**：1:1 画真实尺寸
- **图纸空间（布局）**：摆图框、标题栏，用 `MVIEW` 开视口；进视口后 `ZOOM 1/10XP` 定比例；出来用 `VPLOCK` 锁死视口防误改
- **注释性**（annotative）文字和标注会自动按视口比例缩放，保证打印出来统一 3.5mm 字高

> [!warning] 打印范围选 Layout，别选 Display / Extents
> 打印比例固定 **1:1**（缩放由视口管），打印样式表用 `monochrome.ctb` 出黑白图。

## 六、图纸集 + 批量发布

`SHEETSET`（Ctrl+4）把多个 DWG 的布局收进一个 `.dst`：

- 图纸**自动编号**、集中管修订
- 在图纸集上右键「特性」统一设打印机、纸张、CTB，**所有图纸继承**
- `PUBLISH` 一次把整套图出成**一个 PDF**

> [!tip] 发布失败的排查
> 提示"无法打印"，多半是某张图里有损坏的块或无效外部参照。
> 单独打开那张图，`PU` 清理垃圾再重新发布。

## 七、多重引线（MLEADER）

先用 `MLEADERSTYLE` 定样式（引线、箭头、落地线、文字外观），别逐条手调。
内容用 **MText**：支持多行、堆叠分数、Ø / ± / ° 符号，还能插**字段**（图号、日期、对象属性），
配合 `UPDATEFIELD` 自动更新——改了图号不用手动改引线文字。

## 八、自检

- [ ] 所有图元属性是否 Bylayer？（0 层不画图，只用来定义块）
- [ ] 全局比例三处是否一致？
- [ ] 装配图用的是 XREF 还是 INSERT？
- [ ] 视口是否已锁定？
- [ ] 打印样式表是否是 monochrome？

## 参考来源

- Autodesk University《A Practical Guide to Parametric Drawing in AutoCAD》（约束 + 动态块 + Block Properties Table 官方讲义） https://static.au-uw2-stg.autodesk.com/handout_19238_GEN19238-L-Ellis-AU2016.pdf
- Skill-Lync《Mastering Dynamic Blocks in AutoCAD》（可见性状态、属性定义、块表与夹点） https://skill-lync.com/blogs/autocad-essentals-for-mechanical-engineers-mastering-dynamic-blocks-in-autoca
- ABC Trainings《AutoCAD Layers, Blocks and XREF》/《Sheet Sets, Title Blocks, and Plotting》（生产制图五项基本功、PUBLISH 与供应商提交清单） https://abctraining.in/blog/autocad-layers-blocks-and-xref-production-drawing-skill-1779142016
- NOVEDGE《Standardize Multileader Content》（MLEADERSTYLE / MText / 字段 / 打印比例复核） https://novedge.com/blogs/design-news/autocad-tip-standardize-multileader-content-for-clearer-autocad-documentation
- 浩辰 CAD《CAD 机械制图中图形样板文件的制作教程》（图层、文字样式、标注样式族完整参数） https://www.gstarcad.com/cmsDetail/9309/
- 国标数值见本库 [[机械制图国标速查]]

## 相关

- [[机械制图国标速查]] · [[表面粗糙度符号]] · [[齿轮画法]]
- [[AutoCAD MCP]] — 这些规范已固化进 MCP 工具
- [[工程图与GD&T]] — SolidWorks 侧的对应做法
