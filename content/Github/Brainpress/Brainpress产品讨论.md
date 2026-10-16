# Brainpress 产品讨论与第一版范围

更新日期：2026-10-14

## 记录原则

本目录持续记录产品想法与讨论结论。`brainpress介绍.md` 保留用户原始构想。本文以最新确认范围为准；助手建议不视为已确认需求。

## 最新确认：第一版极简范围

1. 暂不做用户功能，不做注册、登录、个人中心。
2. 只有一套文章内容体系和一套文章分类体系，不单独建立论坛帖子系统。
3. 支持单层文章分类，每篇文章只选择一个分类，以及按分类浏览文章。
4. 支持前台投稿：输入标题、选择分类、填写 Markdown 正文并提交。
5. 首页文章列表参考 Zibll 论坛首页的呈现方式，具体视觉细节后续再研究。
6. 点击列表中的文章进入详情页阅读。
7. 数据库直接保存 Markdown 原文，前台负责渲染阅读。
8. 首版仅支持 CommonMark 基础语法，暂不扩展到完整 Obsidian 语法。

最小流程：前台投稿 → 保存 Markdown 原文与分类关系 → 首页或分类列表显示 → 点击进入文章阅读。

## Markdown 首版范围

按 CommonMark 处理标题、段落、强调、引用、有序和无序列表、行内代码、代码块、链接、图片、分隔线等基础语法。

表格、任务列表、删除线、数学公式、Mermaid、Callout、双链、笔记嵌入暂不实现。代码块基础显示与语法高亮是两项能力，高亮暂不列为必需功能。图片语法显示不代表同时实现图片上传。

CommonMark 中包含原始 HTML 语法；助手建议网页端禁用或严格过滤原始 HTML，并过滤危险链接，作为明确的安全限制，不改变数据库中的原文。

参考：[CommonMark 官方规范](https://spec.commonmark.org/)。

## 最小页面整理

- 首页：文章列表、分类入口、投稿入口。
- 分类页：展示指定分类下的文章，可与首页复用列表布局。
- 投稿页：标题、分类、Markdown 正文、提交按钮。
- 文章页：标题、分类、发布时间、渲染后的正文。

以上字段与页面组织为本次范围的整理，具体样式和交互仍可调整。

## 数据设计建议（尚未最终确认）

先围绕文章与分类设计数据，不提前加入用户、消息或论坛专用数据。

- 文章：ID、标题、Markdown 正文、创建时间、更新时间。
- 分类：ID、名称。
- 分类关系：已确认每篇文章只属于一个分类，文章记录关联一个分类 ID；分类不设父子层级。

渲染后的 HTML 不作为正文的权威来源；如后续需要缓存，可以重新从 Markdown 生成。

## 尚未确定的小问题

- 分类初期通过什么方式添加、修改？
- 没有用户系统时，投稿入口是仅供自己使用，还是对所有访客开放？不将“没有用户功能”自动解释为允许任何人公开投稿。
- 投稿后是否立即发布，以及首版是否需要文章编辑、删除。

这些问题可按开发顺序逐个讨论，不需要现在全部决定。

## 保留的长期方向（不属于首版）

- 以 WordPress 基础内容管理思路为参考；是否需要接口、数据库、插件或主题兼容尚未提出明确要求。
- 极简样式或参考 Zibll 美化；文章和论坛呈现共用内容与分类体系。
- Markdown 逐步对齐 Obsidian，后续再制定扩展语法兼容清单。
- 用户体系、社区互动、消息、搜索、AI 对话。
- REST API、App、智能体接入。
- 自定义分类体系、双链、公式、Mermaid、思维导图等。
- 导出 Markdown 原文件；打包附件和元数据为助手建议。
- 原始部署环境：宝塔、PHP 8.2、Nginx 1.3、MySQL 5.7，实际版本与适用性在开发部署时核实。

Obsidian 参考资料：
- https://obsidian.md/help/obsidian-flavored-markdown
- https://obsidian.md/help/syntax
- https://obsidian.md/help/file-formats

## 决策记录

### 2026-10-14：收紧第一版范围

此前助手建议过用户注册、更多 Markdown 扩展、预览等功能。用户最新决定先不做用户功能，第一版只做分类、前台投稿、参考 Zibll 论坛首页的文章列表、文章阅读，以及 CommonMark 基础渲染。此前较大的范围不再作为第一版要求。尚未开始产品代码开发。

### 2026-10-14：确认分类方案

用户确认：第一版采用单层分类，每篇文章只选择一个分类。


### 2026-10-14：明确 post 与前台参考的关系

用户澄清：Brainpress 的文章统一使用 `post`，分类负责组织内容。用户所说的“论坛”主要指前台分类浏览、内容列表和阅读体验。Zibll 只作为前台设计参考，无需围绕其内部论坛实现设计 Brainpress。

后续研究重点：Zibll 论坛首页布局、分类入口、文章卡片的信息层次、阅读页及移动端呈现。此前源码笔记中关于 Zibll 内容类型的说明仅为实现事实，不作为 Brainpress 架构要求。

### 2026-10-14：个人测试与图片方案

用户确认：
- 初期仅个人测试使用，不做用户系统，不做投稿身份验证与审核，提交直接发布。
- 图片放在独立目录中。
- 可以直接借鉴或复用 WordPress 的实现，结合 Zibll 前台设计；去掉原有编辑器，改为原生 Markdown 编辑与正文存储。

范围解释：用户所说“无检验”在本次讨论中指投稿身份验证和审核；必要的字段校验、参数化数据库写入、渲染安全处理仍为基础实现。当前个人测试方案不自动延伸为公开部署方案。

助手判断与待确认事项：
- 图片目录只能解决文件存放，投稿页面若需要选择本地图片，还需要上传入口并返回可插入 Markdown 的图片地址。第一版可采用一个统一上传目录，不先做媒体库。
- “去掉编辑器”理解为替换原有富文本或区块编辑器，仍保留 Markdown 文本输入。
- 开发路线尚需明确：直接基于 WordPress 改造，或独立开发并参考 WordPress 的内容管理方式。两者不能视为相同方案。
- WordPress 核心复用范围与 Zibll 代码复用范围需分别判断；当前只明确可以研究本地源码，尚未作具体代码复用或分发许可结论。

### 2026-10-14：开发路线最终确认

Brainpress 完全自研，独立运行，不基于 WordPress 改造。WordPress 用于借鉴功能设计、内容管理思路及成熟实现方式；Zibll 用于参考前台布局和展示体验。此前“改造 WordPress 或独立开发”的待确认问题已结束，不再重复询问。

当前已确定首版：个人测试，无用户系统、投稿身份验证与审核；统一 post、单层分类、每篇一个分类；前台 Markdown 投稿并直接发布；CommonMark 基础渲染；参考 Zibll 论坛首页的文章列表及文章详情；图片存独立目录。分类与文章维护方式、图片上传交互等具体细节尚未最终确认。

### 2026-10-14：首版实现完成

源码目录：`C:\Users\windows\Desktop\brainpress`。

已实现：统一 posts 数据、单层单分类、新建分类、Markdown 投稿与编辑、预览、直接发布、首页卡片列表、分类筛选、分页、文章阅读，以及独立目录图片上传。正文原文保存在 posts.markdown。CommonMark 渲染使用 league/commonmark，仅开启基础语法。

界面采用浅灰背景、白色文章卡片、紫蓝色强调色，桌面左侧分类、右侧列表；移动端分类改为横向排列。这是借鉴论坛列表思路的独立实现，后续可继续调整视觉细节。

技术：PHP 8.2+、PDO，开发测试使用 SQLite。提供 MySQL 配置和建表分支，尚未在真实 MySQL 环境验证。未引入 WordPress 或 Zibll 运行依赖。

运行：本机已准备忽略提交的便携 PHP；双击项目 start.cmd 后打开 http://127.0.0.1:8081 。默认数据库位于 storage/brainpress.sqlite，图片位于 public/uploads。安装部署说明见项目 README.md。没有推送 GitHub。

验证：PHP 和 JavaScript 语法检查；Markdown 原文往返、基础语法、安全链接、HTML 转义、非 GFM 模式、外键测试通过；浏览器完成分类创建、投稿、预览、发布和详情阅读；HTTP 图片上传、下载一致性及伪图片拒绝测试通过。保留一篇“你好，Brainpress”示例文章供体验。分类重命名/删除和文章删除未实现。

### 2026-10-14: Frontend and Markdown scope updated

Latest user requests supersede the earlier basic-only rendering scope: use the supplied Zibll screenshots/source as the UI reference, use English for the project, add mobile bottom-tab placeholders, Mermaid, LaTeX, common Markdown extensions, and light/dark modes. Implementation and validation details are recorded in `Frontend-revision-2026-10-14.md` in this directory.

### 2026-10-14: Header and search refinement

User requested a Git baseline before changes; baseline commit is `a5e43b0`. Removed the Zibll-like logo and mobile menu button; moved the mobile theme toggle left. Publish stays stationary. Mobile search now uses a top drawer based on Zibll search behavior, with history, category shortcuts, automatic focus, backdrop/Escape dismissal, scroll locking, and focus restoration. Desktop search has an expandable suggestions panel. UI remains English. Details are also recorded in project CHANGELOG.md.

### 2026-10-14：后台、菜单与侧栏升级

用户确认执行待办第 1、2、3 项，并增加后台文章、媒体库、分类、系统、标签、菜单及可自定义顶部导航。现已实现 /admin 六模块，目录独立吸顶、侧栏仅留 Profile 与 Contents；顶部单层菜单支持增删排序隐藏并在手机／桌面共用。完整状态与验收记录以 [[待办]] 为准。个人测试无登录、导出最后做、正式数据库升级暂缓的决定不变。


## 2026-10-14：布局区域与菜单面板

当前版本先提交为 2935ffa；七个页面布局区域全部开放，保留现有四种小工具，不新增 Markdown 文本小工具。顶部菜单二级项作为宽面板列标题，列下配置文字和可选链接，链接留空显示普通文字，不加左侧描述区。手机底部 Tab 保持独立。部署包搁置；近期任务已重新整理到待办，旧清单归档保留。
