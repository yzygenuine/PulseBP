# AiHome 应用功能概览

## 应用结构

- 三个主 Tab：
  - `Tools` 工具入口（功能预览与快速开始），见 `aihome_app_ios/AiHomeiOS/Views/ToolsView.swift:25`
  - `Design` 设计流程页（根据当前选择的功能切换具体流程），见 `aihome_app_ios/AiHomeiOS/Views/ContentView.swift:261`
  - `My` 我的（历史记录、分享与筛选），见 `aihome_app_ios/AiHomeiOS/Views/MyView.swift:39`
- 顶层通过 `AppState` 统一管理当前 Tab、设计输入、流程步数、返回/关闭信号等，见 `aihome_app_ios/AiHomeiOS/App/AppState.swift:4`

## 工具页（Tools）

- 列出并引导进入五项核心功能：
  - 风格重设计（`STYLE`）
  - 多件家具替换（`FURNITURE_REPLACEMENT`）
  - 墙面重刷（`WALL_COLOR`）
  - 地面风格（`FLOOR_STYLE`）
  - 单件/多件家具替换（`FURNITURE`）
- 点击卡片会设置 `selectedType` 并切换到设计页，见 `aihome_app_ios/AiHomeiOS/Views/ToolsView.swift:29` 与 `aihome_app_ios/AiHomeiOS/Views/ToolsView.swift:31`
- 可从顶部进入订阅页 `SubscriptionPageView`，见 `aihome_app_ios/AiHomeiOS/Views/ToolsView.swift:69`

## 设计页（Design）

- 路由器将 `AppState.selectedType` 映射到具体流程：`DesignFlowRouterView`，见 `aihome_app_ios/AiHomeiOS/Views/Design/DesignFlowRouterView.swift:20`
  - `STYLE` → `StyleFlowView`
  - `FURNITURE` → `FurnitureFlowView`
  - `WALL_COLOR` → `WallColorFlowView`
  - `FLOOR_STYLE` → `FloorStyleFlowView`
  - `FURNITURE_REPLACEMENT` → `FurnitureReplacementView`
- 设计页顶部标题显示当前步数，`ContentView` 负责导航容器与标题工具栏，见 `aihome_app_ios/AiHomeiOS/Views/ContentView.swift:401`
- 若上次生成完成，回到设计页会自动恢复最新输入图，见 `aihome_app_ios/AiHomeiOS/Views/ContentView.swift:294`

## 各设计流程

- 风格重设计（`StyleFlowView`）
  - 步骤：选图 → 房间类型 → 风格选择 → 生成结果
  - 页面切换通过本地 `path` 控制，见 `aihome_app_ios/AiHomeiOS/Views/Design/STYLE/StyleFlowView.swift:59`
  - 发起生成：`UploadViewModel.runFlow(type: "STYLE", ...)`，见 `aihome_app_ios/AiHomeiOS/ViewModels/UploadViewModel.swift:55`

- 家具替换（`FurnitureFlowView`）
  - 步骤：选图 → 目标家具 → 风格选择 → 生成结果
  - 页面切换通过 `path` 条件渲染，见 `aihome_app_ios/AiHomeiOS/Views/Design/FURNITURE/FurnitureFlowView.swift:53`
  - 发起生成：`runFlow(type: "FURNITURE", furnitureType: ...)` 或批量，见 `aihome_app_ios/AiHomeiOS/ViewModels/UploadViewModel.swift:62` 与 `aihome_app_ios/AiHomeiOS/ViewModels/UploadViewModel.swift:181`

- 墙面重刷（`WallColorFlowView`）
  - 步骤：选图 → 模板配色/自定义 → 生成结果
  - 模板卡片仅展示色块，名称已隐藏，选中态有蓝色描边，见 `aihome_app_ios/AiHomeiOS/Views/Design/WALL_COLOR/WallColorFlowView.swift:287`
  - 页面切换与生成控制，见 `aihome_app_ios/AiHomeiOS/Views/Design/WALL_COLOR/WallColorFlowView.swift:103` 与底部按钮 `aihome_app_ios/AiHomeiOS/Views/Design/WALL_COLOR/WallColorFlowView.swift:302`
  - 发起生成：`runFlow(type: "WALL_COLOR", wallColorHex: ..., ceilingColorHex: ...)`，见 `aihome_app_ios/AiHomeiOS/ViewModels/UploadViewModel.swift:79`

- 地面风格（`FloorStyleFlowView`）
  - 步骤：选图 → 材质选择 → 生成结果
  - 页面切换通过 `path` 条件渲染，见 `aihome_app_ios/AiHomeiOS/Views/Design/FLOOR_STYLE/FloorStyleFlowView.swift:50`
  - 发起生成：`runFlow(type: "FLOOR_STYLE", floorMaterial: ...)`，见 `aihome_app_ios/AiHomeiOS/ViewModels/UploadViewModel.swift:114`

- 多件家具替换（`FurnitureReplacementView`）
  - 步骤：选房间图 → 上传 1–N 家具图 → 生成结果
  - 支持一次上传多张家具参考图，见 `aihome_app_ios/AiHomeiOS/Views/Design/FURNITURE_REPLACEMENT/FurnitureReplacementView.swift:303`
  - 发起生成：`runFurnitureReplacement(...)`，见 `aihome_app_ios/AiHomeiOS/ViewModels/UploadViewModel.swift:276`

## 生成过程与进度（Designing）

- 统一的进度/状态页 `DesigningView`，显示进度条与状态文本，见 `aihome_app_ios/AiHomeiOS/Views/DesigningView.swift:13`
- 生成完成后展示首个结果图的全屏预览，并将 `lastDesignCompleted` 标记为已完成，见 `aihome_app_ios/AiHomeiOS/Views/DesigningView.swift:78`
- 服务端交互由 `UploadViewModel` 统一处理，含上传、任务创建、轮询状态与结果下载，见 `aihome_app_ios/AiHomeiOS/ViewModels/UploadViewModel.swift:18`

## 历史记录与分享（My）

- 历史列表与缩略图网格：`HistoryGridView`，见 `aihome_app_ios/AiHomeiOS/Views/HistoryGridView.swift:5`
- 支持筛选不同类型（全部/各功能），见 `aihome_app_ios/AiHomeiOS/Views/MyView.swift:21`
- 支持系统分享（图片或链接），见 `aihome_app_ios/AiHomeiOS/Views/HistoryGridView.swift:70`
- 点击历史可跳回设计页并恢复输入图，见 `aihome_app_ios/AiHomeiOS/Views/HistoryGridView.swift:148`

## 订阅与引导（Subscription & Onboarding）

- 订阅展示页：可轮播对比滑块查看四类功能示例，见 `aihome_app_ios/AiHomeiOS/Views/SubscriptionPageView.swift:17`
- 新手引导页：分屏展示每项功能的说明与使用提示，见 `aihome_app_ios/AiHomeiOS/Views/OnboardingView.swift:10`

## 设置（Settings）

- App 分享、意见反馈、查看引导、关于页面入口，见 `aihome_app_ios/AiHomeiOS/Views/SettingsView.swift:17`
- 图片缓存统计与清理，见 `aihome_app_ios/AiHomeiOS/Views/SettingsView.swift:42`

## 图片与缓存

- 选图：统一使用 `SharedImagePicker` 打开相册/相机（各流程的第一页均支持），如 `FurnitureReplacementView.swift:241`
- 缓存：`ImageCache` 负责下载结果与源图的持久化和读取；历史与设计网格均优先使用缓存，见 `aihome_app_ios/AiHomeiOS/Views/DesignGridView.swift:145` 与 `aihome_app_ios/AiHomeiOS/Views/HistoryGridView.swift:192`
- 历史：`HistoryStore` 记录每次生成的结果集合，供 `MyView` 使用过滤与展示，见 `aihome_app_ios/AiHomeiOS/Views/DesigningView.swift:167`

## 状态与交互约定

- 统一信号：
  - 返回：`designBackSignal`（流程内退一步），见 `aihome_app_ios/AiHomeiOS/App/AppState.swift:15`
  - 关闭：`designCloseSignal`（清空并回到首步），见 `aihome_app_ios/AiHomeiOS/App/AppState.swift:16`
  - 重置：`designResetSignal`（强制重建当前流程视图），见 `aihome_app_ios/AiHomeiOS/App/AppState.swift:17`
- 步数显示：`designStepCurrent` / `designStepTotal`，在各流程内按 `path` 更新，例 `aihome_app_ios/AiHomeiOS/Views/Design/FLOOR_STYLE/FloorStyleFlowView.swift:74`
- 生成完成：`lastDesignCompleted` 标记，回到设计页时用于恢复最近输入图，见 `aihome_app_ios/AiHomeIOS/Views/ContentView.swift:294`

## 视觉与可用性细节

- 选中态统一蓝色描边（如模板卡片），见 `aihome_app_ios/AiHomeiOS/Views/Design/WALL_COLOR/WallColorFlowView.swift:287`
- 网格缩略图使用 AsyncImage 或本地缓存下采样，保证性能与内存占用，见 `aihome_app_ios/AiHomeiOS/Views/HistoryGridView.swift:247`
- 订阅页与引导页提供对比滑块与自动展示，提升功能理解度，见 `aihome_app_ios/AiHomeiOS/Views/SubscriptionPageView.swift:210`

---

以上为当前版本的主要功能与模块结构，后续如需扩展新流程或对现有流程做统一导航/重置策略调整，可以在 `DesignFlowRouterView` 与各 Flow 的首屏/生成触发处进行增强与复用。
