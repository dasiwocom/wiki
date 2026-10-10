---
title: CAD 学习资源地图
aliases: [CAD教程, SolidWorks教程, AutoCAD教程, 学习路线]
tags: [资料, CAD, SolidWorks, AutoCAD, 机械设计]
description: 机械向 CAD 进阶去哪学——中文实战站、官方教材与样例、英文社区、认证路线，附一条从建模到出图的学习顺序
created: 2026-10-10
draft: false
---

# CAD 学习资源地图

> [!note] 这篇解决什么
> 基础操作到处都有，难找的是**机械向的进阶内容**——钣金焊件、GD&T、曲面连续、大装配性能、出图规范。
> 下面按「中文实战 → 官方系统 → 英文社区 → 认证」四层排，越往下越硬。

## 一、中文实战站（讲"车间里怎么用"，不是讲命令）

| 站 | 特点 | 代表作 |
|---|---|---|
| 海马机械设计 `sdjxpx.com` | 非标自动化视角，讲工艺约束（折弯系数、拔模、型材规格） | 钣金与焊件的结构思维、TOP-DOWN 全流程、大装配性能优化、粗糙度与 GD&T 注解 |
| 火星人教育 `2ds.cn` | 长文复盘向，把"翻车过程"写出来 | 焊件能力跃迁路线图、自顶向下 10 个技巧、曲面加厚 10 个致命技巧 |
| 济南机械设计 `jnjxsj.com` | 偏设计规范与建模策略 | 曲面建模与异形零件、自顶向下策略、工程图尺寸标注规范 |

> [!tip] 怎么判断一个中文教程值不值
> 看它有没有写**数字和工艺约束**（折弯扣除多少、K 因子怎么取、拔模几度）。
> 只演示"点这里、点那里"的，是操作录像，不是工程经验。

## 二、官方（系统、免费、最容易被忽略）

| 资源 | 入口 | 适合 |
|---|---|---|
| 内置教程 | 右上角问号 → 教程，分 Getting Started / Advanced / Design Evaluation | 离线免账号，第一次装机就该过一遍 |
| 自带样例模型 | `C:\Users\Public\Documents\SOLIDWORKS\SOLIDWORKS 2025\samples\tutorial` | **每个特征都带注释**，学建模顺序最好的材料 |
| 官方帮助 | https://help.solidworks.com/2025/chinese-simplified/ | 按版本查命令细节，中文 |
| MySolidWorks | guest 账号免费用，绑定订阅解锁全部 | 视频课 + 认证备考 |
| SDC Publications 教材 | 官方教材，项目式 | 《SOLIDWORKS Tutorial》含 CSWA 考题；《Mastering Sheet Metal and Weldments》讲钣金焊件到出图 |

SDC 那本钣金焊件（Lani Tran，2026 版，378 页）目录本身就是一张学习清单：
法兰 → 草图折弯 → 斜接法兰 → 折叠/展开 → 曲面法兰与跳步 → 凸舌开槽与多实体 → 放样折弯 → 角撑与通风口 → 转换到钣金 → 钣金装配 → 钣金工程图 → 焊件 → 切割清单 → CSWP-SM 备考。

## 三、英文社区（深度与工程判断）

| 站 | 强项 |
|---|---|
| GoEngineer | 认证备考体系最完整，CSWE-MD 有明确技能清单 |
| Hawk Ridge Systems | 单篇讲透一个机制（如 SpeedPak 的原理、更新时机、Graphics Circle） |
| Javelin | 课程大纲公开，等于免费拿到一份知识点清单（焊件课目录就 7 章 14 练习） |
| Solid Solutions | 曲面转实体的实操细节（蓝边线 = 未缝合，这条很实用） |
| 3DEngr | CSWA/CSWP/CSWE 备考资源聚合站 |
| Autodesk University 讲义 | AutoCAD 参数化与动态块的官方讲义 PDF，讲约束 + Block Properties Table |

## 四、认证路线（唯一行业认可的能力刻度）

| 层级 | 名称 | 内容 |
|---|---|---|
| 入门 | CSWA | 草图、基础特征、简单装配、工程图。180 分钟 14 题，240 分及格 165 |
| 专业 | CSWP | 配置、方程式、设计表、装配、干涉检查。核心是**改得动** |
| 专项 | CSWPA-SM / DT / SU / MM / WD | 钣金 / 工程图 / 曲面 / 模具 / 焊件 |
| 专家 | CSWE | 20 题 4 小时，需先有 CSWP + 4 个 CSWPA |

> [!important] 考试券是免费的
> 订阅用户每张 license 每年可领两次、每次三张 voucher（1–6 月、7–12 月各一次），共六张。
> voucher 开出后 180 天有效，且**可以给任何人用**——同学之间能互相转让。
> 学生版 SolidWorks 免费，所以这条路径对学生基本零成本。

## 五、AutoCAD 侧

- **参数化约束 + 动态块**：Autodesk University 官方讲义《A Practical Guide to Parametric Drawing in AutoCAD》，讲几何/尺寸约束怎么放进块编辑器、Block Properties Table 怎么做出可下拉选择的标准件族
- 动态块实战：Skill-Lync《Mastering Dynamic Blocks》讲可见性状态、属性定义、块表与夹点
- 生产制图五项基本功：图层标准、块与 WBLOCK、动态块、外部参照 XREF、模型/图纸空间 + PUBLISH 批量发布
- 图纸集管理器（SHEETSET / Ctrl+4）：多张图纸自动编号、集中改修订、一次 PUBLISH 出整套 PDF

## 六、建议的学习顺序（机械向）

| 阶段 | 学什么 | 检验标准 |
|---|---|---|
| 1 | 草图约束 + 特征顺序（见 [[SolidWorks建模]]） | 改一个主尺寸模型不报错 |
| 2 | 机械特有结构（见 [[机械零件建模]]） | 能参数化出一对正确啮合的齿轮 |
| 3 | 钣金与焊件（见 [[钣金与焊件]]） | 能出展开图 + 切割清单，可直接下料 |
| 4 | 工程图与 GD&T（见 [[工程图与GD&T]]） | 一张图交出去，车间不用回来问 |
| 5 | 曲面（见 [[SolidWorks曲面建模]]） | 补面后加厚不报错、斑马纹连续 |
| 6 | 大装配性能（见 [[大型装配体性能]]） | 1000+ 零件还能转得动 |
| 7 | 自动化 | 用 [[SolidWorks MCP]] / [[AutoCAD MCP]] 把重复动作脚本化 |

> [!warning] 顺序别跳
> 3 和 4 是分水岭：这两步之前画的是"形状"，之后画的才是"能造出来的零件"。

## 参考来源

- 海马机械设计《钣金与焊件——非标机架绘制的"结构思维"》 http://www.sdjxpx.com/7866.html
- 海马机械设计《TOP-DOWN 设计思路在 SW 非标自动化设备中的全流程实践》 http://www.sdjxpx.com/7761.html
- 海马机械设计《大型装配体性能优化：轻量化、封套与 SpeedPak 的实战应用》 http://www.sdjxpx.com?p=7418/
- 海马机械设计《表面粗糙度、形位公差与技术要求的高效注解》 http://www.sdjxpx.com/7661.html
- 火星人教育《SolidWorks 焊件结构框架进阶路线图》 https://www.2ds.cn/?p=32383
- 火星人教育《自顶向下设计：10 个直击要害的进阶技巧》 https://www.2ds.cn/
- 火星人教育《曲面加厚实体化进阶秘籍》 https://www.2ds.cn/
- SDC Publications《Mastering Sheet Metal and Weldments with SOLIDWORKS 2026》 https://www.sdcpublications.com/Textbooks/Mastering-Sheet-Metal-Weldments-SOLIDWORKS/ISBN/978-1-63057-795-7/
- GoEngineer《CSWE Mechanical Design》 https://www.goengineer.com/solidworks-certification/cswe-mechanical-design
- 3DEngr《SOLIDWORKS Certification Guide》 https://www.3dengr.com/solidworks-certification-guide
- Javelin《SOLIDWORKS Weldments》课程大纲 https://www.javelin-tech.com/3d/people/training/solidworks-weldments/
- Hawk Ridge Systems《Improving Assembly Performance with SpeedPak》 https://hawkridgesys.com/blog/solidworks-improving-assembly-performance-speedpak
- Solid Solutions《How to Convert Surfaces to Solid Bodies》 https://www.solidsolutions.co.uk/surface-modelling-how-to-convert-surfaces-to-solid-bodies-in-solidworks/
- Autodesk University《A Practical Guide to Parametric Drawing in AutoCAD》 https://static.au-uw2-stg.autodesk.com/handout_19238_GEN19238-L-Ellis-AU2016.pdf
- Skill-Lync《Mastering Dynamic Blocks in AutoCAD》 https://skill-lync.com/blogs/autocad-essentals-for-mechanical-engineers-mastering-dynamic-blocks-in-autoca
- SOLIDWORKS 官方帮助《提高大型装配体性能》 https://help.solidworks.com/2016/Chinese-simplified/SolidWorks/sldworks/c_Improving_Large_Assembly_Performance_SWassy.htm
- SOLIDWORKS 官方帮助《装配体布局草图》 https://help.solidworks.com/2026/chinese-simplified/solidworks/sldworks/c_Assembly_Layout_Sketch.htm

## 相关

- [[SolidWorks建模]] · [[机械零件建模]] · [[AutoCAD MCP]] · [[SolidWorks MCP]]
- [[机械制图国标速查]]
