# SuperWin - Windows 平铺窗口管理器 项目规划书

> **版本 v2（修订版）** · 2026-09-26 · 本文档为项目**技术设计书 + 任务书依据**，执行进度见 `superwin/TASK.md`
> 原始 v1 存档于 `superwin/docs/plan-v1-original.md`

**项目定位**：仿 Hyprland 风格、极致轻量、多布局、可自定义快捷键的 Windows 原生窗口管理器
**语言**：Rust · **项目名**：SuperWin · **配置文件**：`superwin.toml`
**目标**：事件驱动、空闲低 CPU 占用、单静态编译 exe、无多余依赖，对标 Hyprland 体验，运行于 Windows 10/11

---

## 0. 修订记录（v1 → v2）

v1 的方向与模块划分正确，但存在 1 个事实错误和 4 个会在第一行代码就返工的设计缺失。本版逐项修正。

| # | v1 的问题 | 严重度 | v2 的修正 | 章节 |
|---|---|---|---|---|
| 1 | **事实错误**：把 `master`（主从布局）标注为 `dwindle`。Hyprland 默认布局 dwindle 全程缺失 | 高 | 拆开定义五种布局，修正阶段顺序：master 先做，dwindle 补上 | §7 |
| 2 | **完全没提 DPI 感知**。非 DPI-aware 进程跨屏坐标被虚拟化，窗口缩成 1/3，多显示器 gap 全错 | 致命 | 提升为 M0 门槛：入口第一行 Per-Monitor V2，全局统一物理像素坐标 | §3.1 |
| 3 | **没决策要不要去标题栏**。保留标题栏则 tiling 不成立；去标题栏又牵扯拖动冲突 | 致命 | 定案方案 C（DWM 无边框），并定义统一客户区/外框坐标契约 | §3.2 |
| 4 | **没提前台焦点锁**。`SetForegroundWindow` 跨进程通常被系统拒绝，`$mod+J/K` 会静默失效 | 致命 | 给出 `AttachThreadInput` 绕锁完整序列 | §11.1 |
| 5 | **分层污染**：`WinNode` 内嵌 `hwnd`，布局算法无法脱离真实窗口单测 | 高 | 强制三层分离 `Window`(Win32 实体) / `Tile`(布局抽象) / `SplitTree`(分割树) | §4 |
| 6 | **没提 z-order 恢复**。`SW_HIDE` 隐藏的工作区窗口切回来叠放顺序全丢 | 高 | 逆序 `SetWindowPos(HWND_TOP)` 重建，纳入 M5 验收 | §11.2 |
| 7 | **防抖定义错误**：只说"合并事件"，没覆盖 `SetWindowPos` 回显导致的重入死循环 | 高 | 三重防护：回调只投递 / `_applying` 抑制标志 / 期望矩形比对 | §5 |
| 8 | **漏了启动时枚举存量窗口**。`SetWinEventHook` 只能捕获之后的事件 | 中 | M1 强制项：`EnumWindows` 首扫 | §5.1 |
| 9 | **漏了最大化/全屏放行**。这类窗口参与布局会撑爆 gap | 中 | M6 强制项：`GetWindowPlacement` 判定后跳过布局 | §11.3 |
| 10 | **漏了单实例锁**。重复启动两个 WM 抢窗口 | 中 | M0 强制项：`CreateMutexW` | §3.3 |
| 11 | **依赖清单有误**：`windows = "0.52"` 已过时且缺 3 个必需 feature；`once_cell` 多余 | 中 | 换最新稳定版，补 `Dwm`/`LibraryLoader`/`KeyboardAndMouse`，删 `once_cell` | §12 |
| 12 | **`$mod` 默认 LWin 与系统冲突**：`Win+1~9` 是任务栏跳转，`Win+L` 不可屏蔽 | 中 | 默认改 `Alt`，附冲突对照表 | §9.2 |
| 13 | **时间线不现实**：MVP 估 1~3 天 | 中 | 改为里程碑制，每个里程碑带可验证的验收标准，不写天数 | §14 |

---

## 1. 核心设计原则（硬性，全程遵守）

1. **事件驱动，不轮询**。`GetMessageW` 阻塞等待，空闲时不占 CPU。窗口事件经 `SetWinEventHook` 捕获后**只投递不处理**。目标空闲 CPU < 0.5%。
2. **最小依赖**。只用 `windows` + `serde` + `toml`，静态链接，输出单文件 `superwin.exe`（release 预计 2~4 MB）。
3. **配置语法对齐 Hyprland**。TOML，`$mod` 修饰符，`windowrule` 窗口规则，bind 写法贴近 Hyprland。
4. **分层不可破**。布局算法只见 `Tile` 与 `Rect`，永不接触 `HWND`。见 §4。
5. **性能优先**。布局请求合并去抖；`SetWindowPos` 只在期望矩形 ≠ 实际矩形时调用。
6. **失败必须降级，不得崩**。任何 Win32 调用失败（权限、UWP、异常窗口）走 fallback，不 panic。

---

## 2. 先认清不可逾越的 Windows 约束

在写代码前必须接受这四条边界，否则会做无用功：

| 约束 | 结论 |
|---|---|
| 无法替换 DWM 合成器 | SuperWin **只做**窗口位置/大小、全局热键、事件监听。渲染、圆角、模糊、阴影全交给系统 DWM |
| 前台焦点锁 | 跨进程抢焦点必须用 `AttachThreadInput` 绕锁，见 §11.1 |
| z-order 全局唯一 | Windows 的叠放顺序是全局的，**不是 per-monitor**。per-monitor 独立布局必须自行管理并恢复 z-order，见 §11.2 |
| 无标题栏需自建拖动 | 去掉标题栏后系统拖动失效，必须自己实现 `WM_NCHITTEST`，而这会与 tiling 的鼠标交互冲突 → 采用 DWM 方案规避，见 §3.2 |

---

## 3. 关键技术决策（v1 缺失，本项目地基）

这四个决策**先定死再开工**，它们决定了坐标层 API 的形状，事后改动会全量返工。

### 3.1 决策一：DPI 感知 —— 进程级 Per-Monitor V2

**决策**：`main()` 第一行、创建任何窗口之前调用

```
SetProcessDpiAwarenessContext(DPI_AWARENESS_CONTEXT_PER_MONITOR_AWARE_V2)
```

**理由**：Win32 对 DPI-unaware 进程做坐标虚拟化。150% 缩放的 4K 屏上，`GetWindowRect` 返回被系统除过 1.5 的逻辑坐标，`SetWindowPos` 也按逻辑坐标解释 → 跨屏窗口缩成三分之一或异常放大，per-monitor gap 全部错位。这是多显示器支持的**前置条件**，不是可选项。

**配套约定**：

- 全项目**唯一坐标系 = 物理像素**。所有 `Rect` 都是物理像素，不做任何缩放换算。
- 监听 `WM_DPICHANGED` 跟随建议矩形（虽然 Per-Monitor V2 下跨屏由系统处理，但需覆盖 DPI 切换的边界情况）。
- 允许用户在配置中覆盖 `dpi_aware = false` 用于排查问题，但默认强制开启。

### 3.2 决策二：无边框方案 —— 选 C（DWM 无边框）

**候选对比**：

| 方案 | 做法 | 结论 |
|---|---|---|
| A 保留标题栏 | 布局坐标扣 31px | ✗ 平铺区不含标题栏，`gap_out` 到不了屏顶，拖标题栏退化为"移动模式"，不是 tiling |
| B `WM_NCCALCSIZE` 去标题栏 + 子类化窗口过程 | `SetWindowLongPtr(GWL_STYLE)` + subclass proc | ✗ 致命坑：去标题栏后系统拖动失效，必须自建 `WM_NCHITTEST` 拖动，与 tiling 鼠标交互直接冲突；自绘程序 / 游戏 / WinForms 兼容风险高；崩溃在别人进程里 |
| **C DWM 无边框** | `DwmExtendFrameIntoClientArea` + `DWMWA_BORDER_COLOR` 透明 | ✓ **采用**。兼容性最好（glazwm / FancyWM 同路线），保留系统全部鼠标交互与 resize 边框，不子类化任何窗口过程 |

**方案 C 的具体契约**：

- 边框色通过 `DwmSetWindowAttribute(DWMWA_BORDER_COLOR, 0)` 置为完全透明。
- **坐标统一用 `GetWindowRect`（外框矩形）做布局目标**。DWM 的隐形 resize border 在外框内缩，视觉上 `gap_out` 会比数值略小 1~2px，这是方案 C 的正常代价。**严禁混用 `DWMWA_EXTENDED_FRAME_BOUNDS`**，两套坐标混用会导致所有窗口偏移。
- 动画（可选功能）会短暂使用 DWM bounds 计算插值终点，起点终点都从 `GetWindowRect` 取。
- 保留标题栏拖动能力由系统提供，不自建拖动 → 无需处理 `WM_NCHITTEST` 冲突。

### 3.3 决策三：单实例锁

```
CreateMutexW(NULL, TRUE, L"Local\\SuperWin_SingleInstance")
```

`GetLastError() == ERROR_ALREADY_EXISTS` 则说明已有实例，激活其 IPC（`WM_COPYDATA` 或命名管道）后退出。**没有这把锁，重复启动会出现两个 WM 抢同一批窗口。**

### 3.4 决策四：崩溃隔离

SuperWin 操作的是**别人进程的窗口**。一次 `SetWindowPos` 失败不能带崩 WM。规则：

- 所有 Win32 调用返回 `Result`，失败走 fallback + 记日志，**不 `unwrap`/`expect`**。
- `panic = "abort"`，但通过 `catch_unwind` 包裹每个事件处理单元，保证单个窗口的问题不扩散。
- 内部维护的 `Window` 句柄在使用前一律 `IsWindow` 校验，失效即从注册表移除。

---

## 4. 分层架构（v1 最大的缺陷在此修正）

v1 的 `WinNode { hwnd, class_name, title, floating, workspace_id }` 把 Win32 实体和布局输入混在一起，导致布局模块**无法脱离真实窗口做单元测试**。v2 强制拆成三层，每层职责单一、依赖单向向下。

```
┌─────────────────────────────────────────────────────────────┐
│  L4  应用层    main.rs · 消息循环 · 事件分发 · 单实例 · 退出清理   │
├─────────────────────────────────────────────────────────────┤
│  L3  状态层    State{ monitors, workspaces, registry, focus } │
│                纯数据 + 纯变更，无 Win32 调用                    │
├─────────────────────────────────────────────────────────────┤
│  L2  布局层    Layout trait · SplitTree · master/dwindle/...   │
│                输入 Rect + Tile，输出 Vec<Placement>            │
│                ★ 零 Win32 依赖 · 全部可单测                      │
├─────────────────────────────────────────────────────────────┤
│  L1  适配层    win32/  ·  SetWindowPos · DPI · DWM · 事件钩子   │
│                唯一允许出现 HWND 的地方                          │
└─────────────────────────────────────────────────────────────┘
```

### 4.1 核心类型

```rust
// ── L2 布局层：纯数据，零 Win32 依赖 ─────────────────────────
#[derive(Copy, Clone, PartialEq, Eq, Hash, Debug)]
pub struct TileId(pub u64);            // 稳定句柄，窗口消失后不复用

#[derive(Copy, Clone, Debug, PartialEq)]
pub struct Rect { pub x: i32, pub y: i32, pub w: i32, pub h: i32 }
//            ↑ 物理像素，Per-Monitor V2 下无需缩放

#[derive(Debug, Clone)]
pub struct Placement { pub tile: TileId, pub rect: Rect }

/// 二叉分割树：除 stack 外所有布局的统一中间表示
#[derive(Debug, Clone)]
pub enum Node {
    Leaf(TileId),
    /// 并排组：n 个窗口等分同一区域（dwindle 的 master stack）
    Group(Vec<TileId>),
    Split {
        dir: Dir,                 // Vertical=上下切, Horizontal=左右切
        ratio: f32,               // first 占的比例 0.0~1.0
        first: Box<Node>,
        second: Box<Node>,
    },
}

#[derive(Debug, Clone, Default)]
pub struct SplitTree { pub root: Option<Node> }

/// 布局契约：由 tile 序列 + 私有树状态 → 目标矩形
pub trait Layout: fmt::Debug + Send + Sync {
    fn id(&self) -> LayoutId;

    /// 新窗口加入（focus = None 表示无焦点，如冷启动首扫）
    fn insert(&self, tree: &mut SplitTree, new: TileId, focus: Option<TileId>, ctx: &Ctx);

    /// 窗口移除：收缩树，兄弟节点按原比例瓜分
    fn remove(&self, tree: &mut SplitTree, gone: TileId);

    /// 焦点变化（master 布局据此决定是否交换主从）
    fn focus_changed(&self, tree: &mut SplitTree, _focus: TileId, _ctx: &Ctx) {}

    /// 树 → 矩形分配。除 stack 外共用同一份递归算法
    fn compute(&self, tree: &SplitTree, area: Rect, ctx: &Ctx) -> Vec<Placement>;
}

#[derive(Debug, Clone)]
pub struct Ctx {
    pub master_ratio: f32,
    pub split_ratio: f32,
    pub master_count: usize,
    pub dir: Dir,
    pub gap: Gap,
    pub equalize: bool,
}

#[derive(Debug, Clone, Copy)]
pub struct Gap { pub inner: i32, pub outer: i32 }

// ── L3 状态层：Win32 实体，只在 L1 被填充 ────────────────────
pub struct Window {
    pub hwnd: HWND,
    pub tile: TileId,
    pub class: String,
    pub title: String,
    pub pid: u32,
    pub monitor: MonitorId,
    pub workspace: WorkspaceId,
    pub floating: bool,          // windowrule 判定
    pub maximized: bool,         // 最大化/全屏 → 跳过布局
    pub silenced: bool,          // 规则要求不参与平铺
}

pub struct Workspace {
    pub id: WorkspaceId,
    pub monitor: MonitorId,
    pub layout: Box<dyn Layout>,
    pub tree: SplitTree,
    pub tiles: Vec<TileId>,      // 树的 in-order 遍历结果，缓存
    pub focus: Option<TileId>,
    pub gaps: Gap,               // 支持 per-workspace 覆盖
}
```

### 4.2 依赖方向铁律

- `layout/` 模块**禁止**出现 `HWND`、`windows` crate 的任何导入。用 CI 里的 `grep -R "HWND" src/layout/` 断言为空。
- `SplitTree::compute` 的矩形分配算法（gap 处理、Group 等分、Split 递归）是**唯一一份**，所有布局复用。
- L1 拿到 `Vec<Placement>` 后，负责把 `TileId` 映射回 `HWND` 并调用 `SetWindowPos`。

---

## 5. 事件驱动模型与重入防护

### 5.1 事件来源

| 来源 | API | 捕获时机 |
|---|---|---|
| 启动首扫 | `EnumWindows` + `IsWindowVisible` | 进程启动时，**必须**，否则漏掉已存在的窗口 |
| 窗口创建/销毁 | `SetWinEventHook(EVENT_OBJECT_CREATE / DESTROY)` | 之后 |
| 显示/隐藏 | `EVENT_OBJECT_SHOW / HIDE` | 之后 |
| 焦点变化 | `EVENT_SYSTEM_FOREGROUND` + `EVENT_OBJECT_FOCUS` | 之后 |
| 位置/尺寸 | `EVENT_OBJECT_LOCATIONCHANGE` | 之后（**高危，见 §5.3**） |
| Z 序变化 | `EVENT_OBJECT_REORDER` | 之后 |
| 热键 | `RegisterHotKey` 的 `WM_HOTKEY` | 之后 |
| 定时 | `WM_TIMER`（去抖节流） | 16ms |

钩子标志：`WINEVENT_OUTOFCONTEXT | WINEVENT_SKIPOWNPROCESS`。out-of-context 保证回调在**本进程主线程的消息队列**上触发 —— 这是"单线程、无锁"的基础。

### 5.2 事件流水线

```
[Win32 事件源]
      │  SetWinEventHook 回调
      ▼
┌─────────────────────────────────┐
│ ① 回调内：只做投递，一行布局代码都不能有  │
│   if _applying { return }        │  ← 抑制回显
│   queue.push(ev);                │
│   PostMessage(hwnd, WM_APP_TICK) │  ← 唤醒主循环
└─────────────────────────────────┘
      │
      ▼  GetMessageW 循环（阻塞，零 CPU）
┌─────────────────────────────────┐
│ ② 主线程：drain 队列 + 去抖合并      │
│   同一批次内 N 个事件 → 1 次 relayout│
└─────────────────────────────────┘
      ▼
┌─────────────────────────────────┐
│ ③ 纯状态变更（不碰 Win32）           │
│   窗口增删 → registry              │
│   windowrule 匹配 → floating       │
│   树结构变更 → tree.insert/remove  │
└─────────────────────────────────┘
      ▼
┌─────────────────────────────────┐
│ ④ relayout：唯一的 SetWindowPos 出口 │
│   _applying = true                │
│   for p in layout.compute():      │
│       if get_rect(hwnd) != p.rect │
│           SetWindowPos(NOZORDER   │
│              | NOACTIVATE)        │
│   _applying = false               │
└─────────────────────────────────┘
```

### 5.3 重入死循环 —— 三重防护（v1 完全没覆盖）

`SetWindowPos` 会触发 `EVENT_OBJECT_LOCATIONCHANGE`。若在回调里直接重排 → 无限递归 → 窗口疯狂闪烁、CPU 打满、最终系统卡死。这是自建 Windows WM 最容易踩的坑。

| 防护 | 作用 |
|---|---|
| ① 回调只投递 | 钩子回调里**绝不**调用布局或 `SetWindowPos`，回调只往队列塞消息 |
| ② `_applying` 抑制标志 | relayout 期间置位，期间到达的 `LOCATIONCHANGE` 直接丢弃 |
| ③ 矩形比对 | 只在 `GetWindowRect(hwnd) != expected` 时才调 `SetWindowPos`，程序自己造成的回显天然被过滤 |

**残留问题**：用户手动拖动窗口也会产生 `LOCATIONCHANGE`。v1 的"防抖"没有区分"程序移动的回显"与"用户移动"。v2 的处理：收到 `LOCATIONCHANGE` 时比对当前位置是否落在任一 `expected` 矩形上——落在期望值上 = 回显，忽略；偏离 = 用户手动操作 → **临时把该窗口标记为 `_user_moved`，在鼠标释放前不参与重排**（M6 实现）。

---

## 6. 模块结构

```
superwin/
├── Cargo.toml
├── superwin.toml              # 默认配置模板（同时是文档）
├── TASK.md                    # 任务书（里程碑 + 勾选 + 验收）
├── docs/
│   ├── plan-v1-original.md    # v1 存档
│   ├── design.md              # 本文档
│   └── win32-notes.md         # 踩坑记录（实现期持续追加）
└── src/
    ├── main.rs                # 入口：单实例 → DPI → 首扫 → 钩子 → 消息循环
    ├── app.rs                 # L4：事件循环、退出清理、命令分发
    ├── config.rs              # TOML 解析 + 校验 + 热重载
    ├── error.rs               # 错误类型（thiserror 不引入，手写 impl From）
    ├── logging.rs             # 可关闭的分级日志，默认 WARN
    ├── hotkey.rs              # 键名解析 + RegisterHotKey 绑定表
    ├── rules.rs               # windowrule 匹配引擎
    ├── ipc.rs                 # 命名管道 IPC（M8）
    ├── core/
    │   ├── mod.rs
    │   ├── window.rs          # L3 Window 实体 + 过滤规则
    │   ├── tile.rs            # TileId 分配器
    │   ├── monitor.rs         # 显示器枚举 + 工作区
    │   ├── workspace.rs       # 工作区
    │   └── state.rs           # 全局状态 + relayout 调度
    ├── layout/
    │   ├── mod.rs             # Layout trait + Ctx + 共享 compute 算法 ★
    │   ├── tree.rs            # SplitTree 操作：insert/remove/find/遍历
    │   ├── master.rs          # 主从布局
    │   ├── dwindle.rs         # Hyprland 默认布局
    │   ├── bsp.rs             # 二叉分割
    │   ├── panel.rs           # 等分面板
    │   └── stack.rs           # 堆叠（全部同矩形）
    └── win32/
        ├── mod.rs
        ├── api.rs             # 窗口信息 / 客户区 / DPI / 无边框
        ├── hook.rs            # SetWinEventHook + 事件投递
        ├── input.rs           # AttachThreadInput 焦点操控 / z-order
        ├── keys.rs            # 键名 ↔ VK 码
        └── keymap.rs          # 修饰键 + 热键注册
```

`src/layout/` 是本项目**唯一必须 100% 单测覆盖**的模块，其余模块靠集成测试。

---

## 7. 布局引擎详细设计

### 7.1 共享的矩形分配算法

所有布局（除 stack）共用同一份递归分配，gap 在此统一处理：

```
apply(node, area, gap) :
  match node:
    Leaf(t)            → emit(t, area)
    Group(ids)         → ids 在 area 内按 count 等分（方向 = split_dir 的正交方向）
    Split{dir, ratio, a, b}:
        if dir == Vertical:      // 上下切
            avail_h = area.h - gap.inner
            first_h  = round(avail_h * ratio)
            apply(a, Rect(x, y, w, first_h))
            apply(b, Rect(x, y + first_h + gap.inner, w, area.h - first_h - gap.inner))
        if dir == Horizontal:    // 左右切
            同理按宽度切
```

`gap.outer` 在最外层对 monitor 工作区一次性内缩。`SplitTree` 的 `remove` 收缩时，兄弟节点**按原比例瓜分**被移除的空间（不改变相邻比例），避免窗口尺寸突变。

### 7.2 五种布局的精确定义

| 布局 | 树形态 | insert 行为 | 对标 |
|---|---|---|---|
| **master** | `Split{dir, master_ratio, master_chain, stack_chain}`。`master_chain` = `master_count` 个主窗口竖直堆叠；`stack_chain` 从窗口竖直堆叠 | 新窗口进 `stack_chain` 尾部。`focus_changed` 时若焦点在从区且 `focus_follows_master=true`，把该窗口与主窗口**交换**（树节点位置互换，比例不变） | Hyprland `master` |
| **dwindle** | 纯二叉树 + `Group` 节点。焦点所在处存在一个"并排组" | 若焦点 leaf 已在 `Group` 内且未满 `master_count` → 并入（组内等分）；否则把焦点子树切开，`Split{dir: split_dir, ratio: 1/(n+1), first: Group([focus...]), second: 新空位}` | Hyprland **默认**布局 |
| **bsp** | 平衡二叉树 | 焦点 leaf 按 `split_dir` 对半切（ratio 0.5），新窗口占 `second` | 经典 BSP，非 Hyprland 布局 |
| **panel** | N 路等分链 | 追加为新叶子，重算等分 | FancyWM 面板均分 |
| **stack** | 无树 | 覆盖 `compute`：所有 tile 返回**同一** `area`，靠 z-order 区分层级，只有 active 置顶 | Hyprland `stack` |

**修正说明**：v1 把 master 误标为 dwindle，且阶段计划里**真正的 dwindle 全程缺失**。修正后的阶段顺序见 §14。

### 7.3 单测要求（`src/layout/`）

纯函数、无 I/O，全部可单测。最低覆盖：

- 0/1/2/3/N 个 tile 时各布局的矩形不重叠、不留缝（gap 正确）
- 最小窗口尺寸保护：剩余空间不足时钳位而非产生负数宽高
- `remove` 后兄弟节点比例守恒
- `dwindle` 的 `Group` 填满后正确触发 split
- 极端输入：面积小于 `gap.inner * n`、`master_count = 0`、`master_ratio` 越界

---

## 8. 配置规范 `superwin.toml`

```toml
# ── 全局 ─────────────────────────────────────
mod = "Alt"                 # 主修饰符：Alt | LWin | RWin | LAlt | RAlt | LCtrl | RCtrl
gap_in = 8                  # 窗口间距（物理像素）
gap_out = 12                # 屏幕边缘留白
layout = "dwindle"          # 默认布局
master_ratio = 0.55         # 主区占比
master_count = 1
split_ratio = 0.5           # dwindle 新分割块占比
dpi_aware = true            # 建议保持 true，排查问题时可关
borderless = true           # DWM 无边框（§3.2 方案 C）
poll_interval = 0           # 保留字段，恒为 0（本项目不轮询）

# ── 窗口规则：匹配后自动浮动/静音/指定工作区 ─────────────
[[windowrule]]
match_class = "TaskManagerWindow"
float = true

[[windowrule]]
match_title = ".*(保存|另存为|Save As).*"
float = true

[[windowrule]]
match_exe = "chrome.exe"
workspace = 2
float = false

# ── 按键绑定 ─────────────────────────────────
# 不写 [[bind]] 时使用内置默认绑定（见 §9.2）
[[bind]]
key = "$mod, Q, exec, close"
[[bind]]
key = "$mod, Enter, exec, wt.exe"
[[bind]]
key = "$mod, E, layout, next"
```

**配置设计约束**：

- 热键语法用 Hyprland 的逗号分隔风格 `$mod, Shift, J`（v1 用 `+` 号，但 `+` 在解析时和"按键名"混淆，逗号更明确）。
- 所有字段都有默认值，**空配置文件必须能启动**。
- 配置解析失败**保留旧配置继续运行**，仅打印错误 —— 热重载绝不能把 WM 搞死。
- 配置路径优先级：`--config <path>` > `%APPDATA%\superwin\superwin.toml` > 可执行文件同级。

---

## 9. 全局热键

### 9.1 机制选型

| 阶段 | 机制 | 理由 |
|---|---|---|
| M4~M7 | `RegisterHotKey` | 系统级注册，最省电最稳定。`MOD_ALT`/`MOD_CONTROL` 组合完全可用 |
| M8+（可选） | 追加 `WH_KEYBOARD_LL` | 只有需要 Hyprland 式**序列绑定**（`$mod` 松开后接 `J` 才触发）或按下/释放分离时才必需 |

v1 未意识到这一点。MVP 阶段用 `RegisterHotKey` 完全够用，**序列绑定推迟到 M8**，避免过早引入低级钩子的复杂度与反作弊软件误报风险。

`RegisterHotKey` 的硬限制（必须在文档/配置注释里告知用户）：`Win+L`、`Ctrl+Alt+Del`、`Alt+Tab` 等系统保留序列**无法注册**，注册会返回 `ERROR_HOTKEY_ALREADY_REGISTERED`。

### 9.2 默认绑定与冲突规避

**v1 的 `$mod = LWin` 是个体验陷阱**：`Win+1~9` 在 Windows 上默认是"启动/切换任务栏固定程序"，抢占会打断肌肉记忆；`Win+L`、`Win+Tab` 更是系统保留、根本抢不到。

**决策：默认 `mod = "Alt"`**。Alt 组合是 Windows 占用最少的修饰键。

| 绑定 | 功能 | 冲突检查 |
|---|---|---|
| `$mod + J` / `$mod + K` | 焦点 下 / 上 | Alt+J/K 空闲 ✓ |
| `$mod + Shift + J` / `+ K` | 移动窗口 下 / 上 | ✓ |
| `$mod + Q` | 关闭当前窗口 | Alt+Q 空闲（避开 Alt+F4）✓ |
| `$mod + R` | 反转分割方向 | Alt+R 空闲（避开 Win+R）✓ |
| `$mod + F` | 全屏切换 | Alt+F 空闲 ✓ |
| `$mod + E` | 切换到下一个布局 | Alt+E 空闲 ✓ |
| `$mod + ,` / `$mod + .` | 主区比例 − / + | ✓ |
| `$mod + 1`~`0` | 切换工作区 | 避开 Alt+Tab ✓ |
| `$mod + Shift + 1`~`0` | 移动窗口到工作区 | ✓ |
| `$mod + Enter` | 打开终端（`wt.exe`） | Alt+Enter 空闲 ✓ |
| `$mod + Space` | 平铺 / 浮动切换 | ✓ |
| `$mod + Shift + R` | 强制重载配置 | ✓ |
| `$mod + Esc` | 退出（需二次确认） | 避开 Alt+Esc 的菜单行为 ✓ |

用户想用 Win 键可设 `mod = "LWin"`，但需自行避开 §9.1 的保留序列。

### 9.3 钩子回调卫生

`RegisterHotKey` 回调不需要，但 `SetWinEventHook` 回调**必须**极简：只做 `queue.push()` + `PostMessage()`，任何 `String` 分配、锁等待、日志 IO 都是隐患。

---

## 10. 工作区与多显示器

### 10.1 工作区

- 每个工作区持有独立的 `tree` / `layout` / `gaps` / `focus`，切换工作区不丢布局状态。
- 切换：当前工作区窗口 `ShowWindow(SW_HIDE)`，目标工作区窗口 `ShowWindow(SW_RESTORE)` 并**逆序重建 z-order**（§11.2）。
- `$mod + Shift + 1` 把当前窗口迁移到目标工作区（窗口跟随，跨工作区）。
- 工作区数量 `workspace_count = 10`，按需分配。

### 10.2 多显示器

- `EnumDisplayMonitors` + `GetMonitorInfoW` 枚举，区域取 `rcWork`（**排除任务栏**）而非 `rcMonitor`。
- 每个显示器是一个独立的布局上下文：独立工作区集合、独立 tree、独立 gaps。
- 窗口跟随显示器迁移：在 relayout 时按当前窗口中心点落在哪个 monitor 重新归属。
- 拔插显示器：`WM_DISPLAYCHANGE` + `WM_DEVICECHANGE` 触发重建，已有窗口迁移到主显示器。

---

## 11. Windows 专有难点（v1 全部遗漏）

### 11.1 前台焦点锁绕过

`SetForegroundWindow` 跨进程调用通常被系统静默拒绝 → v1 的 `$mod+J/K` 会"按了没反应"。必须：

```
let fg = GetForegroundWindow();
let me = GetCurrentThreadId();
let tgt = GetWindowThreadProcessId(hwnd, null);

AttachThreadInput(me, tgt, TRUE);
if !fg.is_null() { AttachThreadInput(me, GetWindowThreadProcessId(fg, null), TRUE); }

if IsIconic(hwnd) { ShowWindow(hwnd, SW_RESTORE); }
BringWindowToTop(hwnd);
SetForegroundWindow(hwnd);
SetActiveWindow(hwnd);

if !fg.is_null() { AttachThreadInput(me, GetWindowThreadProcessId(fg, null), FALSE); }
AttachThreadInput(me, tgt, FALSE);   // ← 必须解绑，否则死锁
```

### 11.2 z-order 恢复（Windows z-order 是全局的）

`ShowWindow(SW_HIDE)` 隐藏的窗口，切回来时叠放顺序**完全丢失**。恢复方式：

```
// 逆序遍历窗口列表，逐个置顶 → 最后遍历的排在最上
for hwnd in workspace.tiles.iter().rev() {
    SetWindowPos(hwnd, HWND_TOP, 0, 0, 0, 0,
                 SWP_NOMOVE | SWP_NOSIZE | SWP_NOACTIVATE);
}
```

### 11.3 最大化 / 全屏放行

```
let mut wp = zeroed();
GetWindowPlacement(hwnd, &mut wp);
let blocked = wp.showCmd == SW_SHOWMAXIMIZED || wp.showCmd == SW_SHOWFULLSCREEN
           || (GetWindowLongW(hwnd, GWL_STYLE) as u32 & WS_MAXIMIZE) != 0;
```

`blocked` 的窗口**跳过布局**，交还系统全屏/maximized 语义。窗口从最大化恢复时重新入布局。漏这条会让 `gap_out` 被最大化窗口撑爆。

### 11.4 权限降级

管理员权限窗口（如以管理员身份运行的任务管理器）无法被普通权限进程移动。检测 `OpenProcess` 失败 → 标记 `silenced = true`，不进布局，**不反复重试**（否则刷爆日志）。

### 11.5 UWP / 现代窗口

`ApplicationFrameWindow`、`ShellExperienceHost` 等 HWND 行为特殊。内置类名黑名单 + 窗口尺寸异常检测（宽或高为 0）→ 自动浮动。

---

## 12. 依赖清单（修正版）

```toml
[package]
name = "superwin"
version = "0.1.0"
edition = "2024"          # 修正：v1 的 2021 已过时
rust-version = "1.85"

[profile.release]
opt-level = 3             # 布局计算不吃 CPU，别用 z/s
lto = "fat"
codegen-units = 1
strip = true
# panic = "abort" 见下方说明

[dependencies]
windows = { version = "0.6", features = [
    "Win32_Foundation",
    "Win32_Graphics_Dwm",             # 新增：DWM 无边框（§3.2）
    "Win32_System_LibraryLoader",     # 新增：GetModuleHandleW
    "Win32_UI_Input_KeyboardAndMouse",# 新增：热键与按键转换，必需
    "Win32_UI_WindowsAndMessaging",
] }
serde = { version = "1", features = ["derive"] }
toml   = "0.8"
```

修正说明：

| 问题 | 修正 |
|---|---|
| `windows = "0.52"` 已过时（0.58+ 把 `HWND` 等改成 `*mut c_void`） | 用当前最新稳定版，`cargo add windows` 生成，不手写版本号 |
| 缺 `Win32_Graphics_Dwm` | 补上，方案 C 必需 |
| 缺 `Win32_System_LibraryLoader` | 补上，取模块实例句柄必需 |
| 缺 `Win32_UI_Input_KeyboardAndMouse` | 补上，热键与 `MapVirtualKey` 必需 |
| `once_cell` 多余 | **删除**，功能已在 `std::sync::OnceLock` |
| 无 `[profile.release]` | 补上，单 exe 目标依赖它 |
| `panic = "abort"` | ⚠️ 与 §3.4 的 `catch_unwind` 冲突。二选一：**默认 `panic = "unwind"` + `catch_unwind` 隔离**（崩溃安全优先）；确认稳定后再切 `abort` 瘦身 |

M8 追加：`Win32_UI_Shell_NotifyIcon`（托盘）、`Win32_System_Threading`（`CreateProcessW` 拉起程序）。

**分发目标**：静态链接 + `strip` 后单 exe 约 2~4 MB，无外部运行库依赖。**目标现实可达。**

---

## 13. 风险与坑位清单（可执行检查项）

| # | 坑 | 触发条件 | 对策 | 里程碑 |
|---|---|---|---|---|
| 1 | DPI 坐标虚拟化 | 未设 DPI 感知 | §3.1 入口强制 | M0 |
| 2 | 标题栏占位 | 保留标题栏 | §3.2 DWM 方案 C | M3 |
| 3 | 前台焦点锁 | 跨进程 SetForegroundWindow | §11.1 AttachThreadInput | M3 |
| 4 | 重入死循环 | 回调内调 SetWindowPos | §5.3 三重防护 | M2 |
| 5 | z-order 丢失 | 切工作区 | §11.2 逆序重建 | M5 |
| 6 | 最大化撑爆 gap | 未过滤最大化窗口 | §11.3 跳过布局 | M6 |
| 7 | 权限不足 | 管理员窗口 | §11.4 标记 silenced | M6 |
| 8 | UWP 窗口异常 | 特殊类名 / 0 尺寸 | §11.5 黑名单 + 尺寸兜底 | M1 |
| 9 | 最小尺寸被压成 0 | 窗口过多 | 布局层 min 尺寸钳位 + 溢出降级为滚动 | M2 |
| 10 | 双实例抢窗口 | 重复启动 | §3.3 单实例锁 | M0 |
| 11 | 热重载炸掉 WM | 配置语法错误 | 解析失败保留旧配置 | M4 |
| 12 | 布局模块被 Win32 污染 | 图省事直接传 HWND | CI 断言 `src/layout/` 无 HWND | 全程 |

---

## 14. 里程碑与验收标准

> 不写天数（v1 的时间线不现实）。每个里程碑以**可验证的验收标准**结束，不通过不进入下一阶段。
> 详细勾选清单见 `superwin/TASK.md`。

### M0 · 地基
单实例锁、DPI 感知、日志（可关闭）、Win32 错误类型、优雅退出（还原所有被修改的窗口）。
**验收**：重复启动第二个实例立即退出；日志确认 DPI 上下文为 Per-Monitor V2；Ctrl+C 或 `$mod+Esc` 退出后无残留窗口状态。

### M1 · 窗口发现与追踪
`EnumWindows` 首扫、`SetWinEventHook` 事件接入、窗口元数据采集、基础过滤（不可见/最小化/工具窗口/0 尺寸/UWP 黑名单）、事件队列与主循环。
**验收**：在 SuperWin 启动前打开的窗口能被纳管；开/关任意应用窗口，注册表准确增删；空闲 5 分钟 CPU < 0.5%。

### M2 · 布局引擎 ★
`SplitTree` + `Layout` trait + 共享 compute + `master` + `dwindle` + `Gap`。**本阶段纯逻辑，不接 Win32。**
**验收**：`src/layout/` 单测全绿（§7.3 全部用例）；`grep -R "HWND" src/layout/` 结果为空。

### M3 · 窗口操控
rect 比对应用、焦点切换（AttachThreadInput）、窗口移动/交换、关闭、DWM 无边框、最大化放行。
**验收**：开 4 个窗口自动排成 dwindle 且不闪烁；焦点热键可靠生效（连按 20 次无失败）；窗口拖出屏幕边缘不产生错位。

### M4 · 配置与热键
TOML 解析与校验、`RegisterHotKey` 绑定表、默认绑定、热重载（失败保留旧配置）。
**验收**：改 `gap_in` 重载后立即生效；故意写坏 TOML 不崩溃且沿用旧配置；Alt 组合全部可用。

### M5 · 工作区
多工作区、切换、隐藏/显示、**z-order 恢复**、窗口迁移到工作区。
**验收**：切走再切回，窗口叠放顺序与切走前一致；工作区布局状态独立保持。

### M6 · 稳定性
去抖合并、`_applying` 抑制、用户手动移动检测（`_user_moved`）、多显示器与 per-monitor 布局、`WM_DISPLAYCHANGE`、权限降级、崩溃隔离。
**验收**：连续开关 30 个窗口不卡死不闪烁；拔插显示器不崩溃且窗口合理重排；最大化窗口不破坏 gap。

### M7 · 布局补全
`bsp` / `panel` / `stack`、布局热切换（保留 tree）、浮动切换、per-workspace gaps。
**验收**：5 种布局均可切换且窗口不重叠不溢出；切换布局不丢窗口顺序。

### M8 · 体验与集成
托盘图标、命名管道 IPC（供状态栏读状态）、session 持久化（exe 路径 + 工作区，重启恢复）、结构化日志、动画（可选，默认关）。
**验收**：重启后窗口按 session 恢复；外部程序能通过 IPC 读到工作区与焦点窗口标题；托盘可重载/退出。

### M9 · 打磨（可选）
序列绑定（`WH_KEYBOARD_LL`）、窗口规则增强（置顶/静默/透明度）、安装脚本与 Release 打包、发布签名。

---

## 15. 明确的非目标

写清楚不做什么，避免范围蔓延：

- ❌ 不做虚拟桌面（Windows 已有 Win+Ctrl+数字，SuperWin 用隐藏/显示模拟）
- ❌ 不做渲染/圆角/模糊/主题（DWM 的职责）
- ❌ 不做鼠标自由拖拽窗口（v1 提到但明确推迟：与 tiling 交互冲突，列入 M9 之后）
- ❌ 不做 Wayland/X11 兼容层
- ❌ 不做窗口组、快照等实验性功能（v1 的"后续拓展"暂不排期）

---

## 16. 测试策略

| 层级 | 范围 | 手段 |
|---|---|---|
| 单元测试 | `src/layout/` **100%** | 纯函数单测，CI 必跑 |
| 单元测试 | `config` 解析、键名解析、rule 匹配 | 纯函数单测 |
| 集成测试 | Win32 调用层 | 真窗口操作，人工验收清单 |
| 静态检查 | 分层不变量 | `grep -R "HWND" src/layout/` 必须为空 |
| 性能 | 空闲占用 | 任务管理器观察 5 分钟 |
| 稳定性 | 长时运行 | 连续开关 30 窗口、拔插显示器、切 10 工作区 |

---

**执行入口**：`superwin/TASK.md`（里程碑勾选 + 每项验收命令）
**踩坑记录**：`superwin/docs/win32-notes.md`（实现期持续追加真实 API 行为）
