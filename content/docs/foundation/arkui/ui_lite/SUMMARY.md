# SUMMARY - ui_lite Wiki 导航

## 快速开始

- [Wiki 首页](README.md) - 文档说明与阅读指南
- [项目概览](01_Overview.md) - 定位、边界、核心能力

## 架构与设计

- [架构说明](02_Architecture.md) - 组件图、数据流、线程模型、时序图
- [目录结构](03_Directory_Structure.md) - 模块职责与文件组织

## API 文档

- [对外 C++ API](04_Public_API.md) - 外部接口完整列表
  - 组件类 API
  - 事件系统 API
  - 动画 API
  - 字体 API
  - 主题 API
  - 窗口 API
- [内部 API](05_Internal_API.md) - 模块间接口
  - 核心管理器
  - 绘制引擎
  - 输入设备

## 构建与产物

- [GN 构建目标](06_GN_Targets.md) - Targets、依赖、配置
- [编译产物](07_Build_Artifacts.md) - 输出文件与加载关系

## 安全与问题

- [安全风险评审](08_Security.md) - 攻击面、风险点、修复建议
- [常见问题](09_Troubleshooting.md) - 构建/运行/调试问题

## 附录

- [配置宏清单](appendix/Config_Flags.md) - Feature Flags 详解
- [调用链分析](appendix/Callgraphs.md) - 关键调用路径

---

## 符号索引

### 核心类

- `UIView` - 视图基类 ([interfaces/kits/components/ui_view.h](interfaces/kits/components/ui_view.h))
- `RootView` - 根视图 ([interfaces/kits/components/root_view.h](interfaces/kits/components/root_view.h))
- `Window` - 窗口管理 ([interfaces/kits/window/window.h](interfaces/kits/window/window.h))
- `TaskManager` - 任务管理 ([interfaces/innerkits/common/task_manager.h](interfaces/innerkits/common/task_manager.h))
- `GraphicStartUp` - 图形启动 ([interfaces/innerkits/common/graphic_startup.h](interfaces/innerkits/common/graphic_startup.h))

### 组件类

- `UILabel` - 文本标签
- `UIButton` - 按钮
- `UIImageView` - 图像视图
- `UIList` - 列表
- `UIScrollView` - 滚动视图
- `UISlider` - 滑块
- `UICheckBox` - 复选框
- `UIRadioButton` - 单选按钮
- `UIProgressBar` - 进度条
- `UIDialog` - 对话框
- `UICanvas` - 画布

### 事件类

- `ClickEvent` - 点击事件
- `PressEvent` - 按下事件
- `ReleaseEvent` - 释放事件
- `DragEvent` - 拖拽事件
- `KeyEvent` - 按键事件
