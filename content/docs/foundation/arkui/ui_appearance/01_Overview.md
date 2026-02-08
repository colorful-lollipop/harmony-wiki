# 项目概述

> UI Appearance 子系统定位、边界与核心能力

## 项目定位

**UI Appearance** 是 OpenHarmony ArkUI 子系统中的一个组件，负责管理和配置系统的用户界面外观，特别是**深色模式服务**。

### 核心职责

1. **深色模式管理**: 提供系统级深色模式设置与获取
2. **字体缩放管理**: 支持系统字体大小和字重缩放
3. **多用户支持**: 为不同用户独立维护配置
4. **配置持久化**: 持久化保存用户界面外观参数
5. **状态联动**: 与 AMS、WMS、ResourceManager 联动更新

### 与其他模块的关系

```
┌─────────────────────────────────────────────────────────────┐
│                    UI Appearance                              │
├─────────────────────────────────────────────────────────────┤
│  核心能力: 深色模式 + 字体缩放                                │
├─────────────────────────────────────────────────────────────┤
│  依赖模块:                                                   │
│  ├── AMS (Ability Manager Service) - 配置更新                │
│  ├── WMS (Window Manager Service) - 窗口配置                 │
│  ├── Resource Manager - 资源限定词                           │
│  ├── Common Event Service - 公共事件订阅                      │
│  └── Account Manager - 多用户管理                            │
└─────────────────────────────────────────────────────────────┘
```

## 项目边界

### 包含范围

| 模块 | 路径 | 职责 |
|------|------|------|
| N-API 接口 | `interfaces/kits/napi/` | JS API 实现 |
| SA 服务 | `services/` | 深色模式核心逻辑 |
| 配置管理 | `services/utils/` | 设置数据、定时器、参数 |
| SA 配置 | `sa_profile/` | SA 7002 配置 |
| 持久化配置 | `etc/para/` | 系统参数文件 |

### 不包含范围

1. **产品化定制**: 不包括产品应用定制的外观的模式（如桌面简易模式）
2. **UI 渲染**: 不负责实际的 UI 渲染（由 AceEngine 负责）
3. **应用级配置**: 不处理应用内部的主题配置
4. **测试代码**: `test/` 目录不纳入文档范围

## 核心能力

### 1. 深色模式

**支持模式**:

| 模式 | 值 | 说明 |
|------|-----|------|
| ALWAYS_DARK | 0 | 始终深色 |
| ALWAYS_LIGHT | 1 | 始终浅色 |
| UNKNOWN | 2 | 未知状态 |

**自动切换模式**:

- **自定义时间段**: 用户指定开始和结束时间
- **日出日落模式**: 根据地理位置自动计算

### 2. 字体缩放

| 参数 | 范围 | 精度 | 默认值 |
|------|------|------|--------|
| fontScale | 0 ~ 5 | 0.01 | 1.0 |
| fontWeightScale | 0 ~ 5 | 0.01 | 1.0 |

### 3. 多用户支持

- 支持系统多用户场景
- 每个用户独立配置
- 用户切换时自动切换配置

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| **系统版本** | OpenHarmony 3.2+ |
| **系统能力** | SystemCapability.ArkUI.UiAppearance |
| **权限要求** | ohos.permission.UPDATE_CONFIGURATION |

### 运行时依赖

| 组件 | 用途 |
|------|------|
| SAFwk | 系统能力框架 |
| SAMgr | 服务管理 |
| AppMgr | 应用管理 |
| CommonEventService | 公共事件 |
| OsAccountManager | 账户管理 |

## 关键概念

### System Ability (SA)

UI Appearance 以 System Ability 形式运行：

| 属性 | 值 |
|------|-----|
| SA ID | 7002 |
| 进程名 | ui_service |
| 库路径 | libui_appearance_service.z.so |
| 启动方式 | 系统启动时创建 (run-on-create) |

### 权限模型

| 权限 | 名称 | 用途 |
|------|------|------|
| ohos.permission.UPDATE_CONFIGURATION | 更新系统配置 | 设置深色模式/字体缩放 |

### 配置存储

| 存储类型 | 键名前缀 | 说明 |
|----------|----------|------|
| 系统参数 | `persist.ace.darkmode.<userId>` | 深色模式 |
| 系统参数 | `persist.sys.font_scale_for_user.<userId>` | 字体缩放 |
| 系统参数 | `persist.sys.font_wght_scale_for_user.<userId>` | 字重缩放 |
| 设置数据 | `settings.uiappearance.*` | 用户设置 |

## 模块概览

```
arkui/ui_appearance/
├── interfaces/                    # 对外接口
│   ├── kits/napi/                  # N-API 实现
│   │   ├── include/js_ui_appearance.h
│   │   ├── src/js_ui_appearance.cpp
│   │   └── BUILD.gn
│   └── ets/ani/                    # ETS 接口
├── services/                       # 系统服务
│   ├── include/                    # 头文件
│   │   ├── ui_appearance_ability.h
│   │   ├── dark_mode_manager.h
│   │   └── ...
│   ├── src/                        # 实现
│   │   ├── ui_appearance_ability.cpp
│   │   ├── dark_mode_manager.cpp
│   │   └── ...
│   └── utils/                      # 工具类
├── sa_profile/                     # SA 配置
│   └── 7002.json
├── etc/                            # 配置文件
│   └── para/
└── BUILD.gn
```

## 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|----------|
| 3.2 | 2025 | 当前版本, 支持完整 N-API |

---

*相关内容: [架构设计](02_Architecture.md) | [N-API 接口](03_NAPI.md)*
