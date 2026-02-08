# 项目概览

## 目的与适用范围

本文档介绍 OpenHarmony Window Manager 子系统的整体定位、核心能力和运行环境。

**适用范围**：
- 窗口管理子系统架构师
- 应用框架开发者
- 系统集成工程师
- 安全审计人员

## 项目定位

Window Manager（窗口管理器）是 OpenHarmony 的基础 UI 子系统，提供：
- **窗口管理**：窗口创建、销毁、生命周期管理
- **显示管理**：屏幕信息、分辨率、亮度、旋转
- **布局控制**：窗口树、Z序、拖拽、动画
- **多屏支持**：多显示器协同、折叠屏适配

### 子系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层 (Applications)                      │
├─────────────────────────────────────────────────────────────────┤
│  ArkUI (UI框架)  │  Ability (能力框架)  │  第三方应用              │
└────────┬────────────────────────────────────────────────────────┘
         │
┌────────▼────────────────────────────────────────────────────────┐
│                    Window Manager Client                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ Window       │  │ WindowManager│  │ DisplayManager│          │
│  │ (窗口对象)    │  │ (窗口管理)    │  │ (显示管理)    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
└────────┬────────────────────────────────────────────────────────┘
         │ IPC (Binder)
┌────────▼────────────────────────────────────────────────────────┐
│                    Window Manager Server                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Window Manager Service (WMS)                              │   │
│  │  - 窗口布局策略 (Layout Policy)                            │   │
│  │  - Z序控制 (Z-Order Control)                               │   │
│  │  - 窗口树管理 (Window Tree)                                │   │
│  │  - 动画管理 (Animation)                                    │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Display Manager Service (DMS)                             │   │
│  │  - 显示信息管理 (Display Info)                             │   │
│  │  - 截图控制 (Screenshot)                                   │   │
│  │  - 亮灭屏 (Power Control)                                  │   │
│  │  - 亮度控制 (Brightness)                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
         │
┌────────▼────────────────────────────────────────────────────────┐
│                       底层服务                                    │
│  Render Service │  Input Service │  Power Manager │ Sensor      │
└─────────────────────────────────────────────────────────────────┘
```

## 核心能力

### 1. 窗口管理 (wm/ + wmserver/)

**客户端能力** (`wm/`):
- 窗口对象抽象 (`Window` 类)
- 窗口配置选项 (`WindowOption`)
- 窗口事件监听 (生命周期、焦点、触摸)
- 画中画 (PIP) 支持
- 悬浮球 (Floating Ball) 支持

**服务端能力** (`wmserver/`):
- 窗口布局策略（层叠、平铺）
- Z序控制与窗口层级
- 窗口树结构管理
- 窗口拖拽与多窗口手势
- 窗口截图与快照

### 2. 显示管理 (dm/ + dmserver/)

**客户端能力** (`dm/`):
- 显示信息查询 (Display)
- 屏幕管理 (Screen)
- 屏幕分组 (ScreenGroup)

**服务端能力** (`dmserver/`):
- 抽象显示控制 (AbstractDisplay)
- 屏幕旋转管理
- 截图能力
- 亮灭屏控制
- 亮度调节

### 3. 场景管理 (window_scene/) - Scene Board 架构

新一代窗口管理架构，替代传统 WMS：
- **SceneSession**：场景会话，替代传统 Window
- **SessionManager**：会话管理器
- **ScreenSessionManager**：屏幕会话管理
- 支持分布式窗口协同

## 系统能力 (Syscap)

```json
{
  "SystemCapability.WindowManager.WindowManager.Core": true,
  "SystemCapability.Window.SessionManager": false
}
```

- `WindowManager.Core`：基础窗口管理能力
- `SessionManager`：场景管理能力（可选）

## 运行环境

### 支持设备类型
- 手机 (Phone)
- 平板 (Tablet)
- 智能屏 (Smart Screen)
- 车机 (Car)
- 折叠屏 (Foldable)
- 2in1 设备

### 编译条件
- **语言标准**：C++11 及以上
- **主要架构**：arm64, x86_64
- **系统类型**：标准系统 (standard)

## Feature Flags

| 特性标志 | 说明 | 默认值 |
|---------|------|--------|
| `window_manager_use_sceneboard` | 使用 Scene Board 架构 | true |
| `window_manager_fold_ability` | 折叠屏支持 | true |
| `window_manager_feature_multi_screen` | 多屏幕支持 | true |
| `window_manager_feature_multi_usr` | 多用户支持 | true |
| `window_manager_feature_support_dsoftbus` | 分布式软总线 | true |
| `window_manager_feature_screen_active_mode` | 屏幕主动模式 | true |
| `window_manager_feature_screen_color_gamut` | 色域支持 | true |
| `window_manager_feature_screen_hdr_format` | HDR 格式支持 | true |

## 依赖关系

### 核心依赖
- `ability_runtime`：Ability 生命周期管理
- `graphic_2d`：图形渲染服务
- `ipc` / `samgr`：IPC 通信和 SA 框架
- `input`：多模输入服务
- `access_token`：权限管理

### 可选依赖
- `power_manager`：电源管理
- `sensor`：传感器服务
- `dsoftbus`：分布式软总线

## 关键概念

### 窗口类型
| 类型 | 说明 | 使用场景 |
|------|------|----------|
| APP_WINDOW | 应用窗口 | 普通应用界面 |
| SYSTEM_WINDOW | 系统窗口 | 系统弹窗、通知 |
| FLOATING_WINDOW | 悬浮窗口 | 悬浮球、画中画 |
| SUB_WINDOW | 子窗口 | 对话框、菜单 |

### 窗口状态
```
BEGIN → CREATE → SHOW → HIDE → DESTROY → END
         ↑______↓      ↑______↓
```

### 显示模式
- **全屏 (Full Screen)**：占据整个显示区域
- **分屏 (Split Screen)**：多应用并排显示
- **悬浮 (Floating)**：可拖拽的小窗口
- **画中画 (PIP)**：视频小窗播放

## 相关文档

- [架构说明](02_Architecture.md)
- [目录结构](03_Directory_Structure.md)
- [N-API 参考](04_NAPI_Reference.md)
