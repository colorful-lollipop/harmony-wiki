# 项目概览

## 项目定位

OpenHarmony 通知子系统（Advanced Notification Service, ANS）是系统的核心消息通知管理服务，负责管理系统和应用的通知消息。

**源码位置**: `//base/notification/distributed_notification_service`

**子系统**: `notification`

## 核心能力

### 通知类型支持
| 类型 | 说明 | 代码位置 |
|------|------|----------|
| 普通文本 | 简短文本通知 | `notification_normal_content.h/cpp` |
| 长文本 | 支持长文本内容 | `notification_long_text_content.h/cpp` |
| 多行文本 | 多行列表形式 | `notification_multiline_content.h/cpp` |
| 图片通知 | 带图片附件 | `notification_picture_content.h/cpp` |
| 社交通知 | 会话消息形式 | `notification_conversational_content.h/cpp` |
| 媒体通知 | 媒体播放控制 | `notification_media_content.h/cpp` |
| 实时通知 | LiveView类型 | `notification_live_view_content.h/cpp` |

### 通知通道类型
- **社交通讯**: 消息类通知
- **服务提醒**: 提醒类通知
- **内容资讯**: 资讯类通知
- **其他**: 其他类型通知

### 核心功能
| 功能 | 说明 | 特性开关 |
|------|------|----------|
| 徽章管理 | 应用图标徽章显示 | `feature_badge_manager` |
| 免打扰管理 | 勿扰模式控制 | `feature_disturb_manager` |
| 实时通知 | LiveView 实时交互 | `feature_local_liveview` |
| 分布式同步 | 跨设备通知同步 | `feature_distributed_db` |
| 场景化协作 | 多设备场景协同 | `feature_all_scenario_collaboration` |
| 优先级通知 | 重要通知管理 | `feature_priority_notification` |
| 地理围栏 | 基于位置的通知 | `feature_support_geofence` |

**证据**: `notification.gni` 特性开关定义

## 运行环境

### 系统依赖
| 依赖组件 | 用途 | 必需 |
|----------|------|------|
| ability_runtime | 能力运行时 | 是 |
| access_token | 权限管理 | 是 |
| ipc | 进程间通信 | 是 |
| safwk | SA框架 | 是 |
| samgr | 服务管理 | 是 |

### 硬件要求
- **ROM**: ~3000KB
- **RAM**: ~16000KB

**证据**: `bundle.json` 资源占用

### 权限要求
| 权限 | 说明 |
|------|------|
| SystemCapability.Notification.Notification | 基础通知能力 |
| SystemCapability.Notification.ReminderAgent | 提醒代理能力 |
| SystemCapability.Notification.NotificationSettings | 通知设置能力 |

## 目录结构

```
distributed_notification_service/
├── interfaces/          # 接口层
│   ├── inner_api/       # Inner API (C++模块接口)
│   ├── kits/            # Kit API (JS/TS接口)
│   └── ndk/             # NDK API (C接口)
├── frameworks/          # 框架层
│   ├── ans/             # ANS客户端库
│   ├── core/            # 核心实现
│   ├── extension/       # 扩展能力
│   ├── js/napi/         # JS/N-API绑定
│   ├── ets/ani/         # ETS/ArkTS绑定
│   └── reminder/        # 提醒服务
├── services/            # 服务层
│   ├── ans/             # ANS核心服务
│   ├── distributed/     # 分布式模块
│   ├── dialog_ui/      # 对话框UI
│   └── reminder/        # 提醒服务实现
├── sa_profile/          # SA配置
├── tools/              # 工具
└── build/              # 构建配置
```

## 关键概念

### 通知生命周期
```
[发布] -> [存储] -> [分发] -> [展示] -> [取消/过期]
```

### 订阅者模式
- **普通订阅**: 接收所有通知
- **实时订阅**: LiveView 交互
- **扩展订阅**: 跨设备扩展

### 分布式特性
- 跨设备通知同步
- 设备间通知流转
- 分布式状态管理
