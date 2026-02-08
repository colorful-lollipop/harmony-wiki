# 系统架构

## 架构概述

```
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (Application)                     │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ JS/N-API    │  │ ETS/ANI     │  │ NDK                 │  │
│  │ (框架)      │  │ (ArkTS)     │  │ (C)                 │  │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │
├─────────┼────────────────┼─────────────────────┼─────────────┤
│         │                │                     │             │
│         ▼                ▼                     ▼             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Inner API (C++ 模块接口)                │    │
│  │  notification_helper.h (83KB核心API)                 │    │
│  └───────────────────────┬─────────────────────────────┘    │
│                          │                                 │
│                          ▼                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │               Framework Layer                       │    │
│  │  frameworks/ans/ (ANS客户端库)                       │    │
│  │  - IAnsManager.idl (446个IPC方法)                   │    │
│  │  - IPC Proxy/Stub                                  │    │
│  └───────────────────────┬─────────────────────────────┘    │
│                          │                                 │
│                          ▼                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │               Service Layer                          │    │
│  │  services/ans/ (ANS核心服务)                         │    │
│  │  - SA ID: 3203                                      │    │
│  │  - libans.z.so                                      │    │
│  └───────────────────────┬─────────────────────────────┘    │
│                          │                                 │
│                          ▼                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              基础设施层                               │    │
│  │  - Preferences (偏好设置)                            │    │
│  │  - RDB (关系数据库)                                 │    │
│  │  - Distributed DB (分布式数据库)                    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## 组件职责

### 框架层组件

| 组件 | 路径 | 职责 |
|------|------|------|
| ANS Client | `frameworks/ans/` | IPC通信客户端、类型定义 |
| Core | `frameworks/core/` | 核心数据处理、转换工具 |
| Extension | `frameworks/extension/` | 订阅者扩展能力 |
| JS/N-API | `frameworks/js/napi/` | JS接口绑定 |
| ETS/ANI | `frameworks/ets/ani/` | ArkTS接口绑定 |

**证据**: `frameworks/ans/BUILD.gn` 定义了完整的客户端库构建

### 服务层组件

| 组件 | 路径 | 职责 |
|------|------|------|
| ANS Service | `services/ans/` | 核心通知服务 |
| Distributed | `services/distributed/` | 分布式同步 |
| Dialog UI | `services/dialog_ui/` | 通知使能对话框 |
| Reminder | `services/reminder/` | 提醒服务 |

**证据**: `services/ans/BUILD.gn` 定义了核心服务构建

### 服务管理组件

| 组件 | 路径 | 职责 |
|------|------|------|
| Subscriber Manager | `notification_subscriber_manager.cpp` | 订阅者管理 |
| Preferences | `notification_preferences.cpp` | 用户配置 |
| Slot Service | `advanced_notification_slot_service.cpp` | 通道管理 |
| Publish Service | `advanced_notification_publish_service.cpp` | 发布服务 |

## IPC 接口

### Service Ability 配置
```json
{
  "name": 3203,
  "libpath": "libans.z.so",
  "run-on-create": true,
  "depend": [3299],
  "extension": ["backup", "restore"],
  "distributed": false
}
```

**证据**: `sa_profile/3203.json`

### IAnsManager 接口
- **方法数量**: 446+ 个IPC方法
- **接口定义**: `frameworks/ans/IAnsManager.idl`
- **主要方法分类**:
  - 发布/取消通知
  - 订阅/退订管理
  - 通道(Slot)管理
  - 设置管理
  - 分布式功能
  - 提醒服务

**证据**: `IAnsManager.idl:40-445`

## 数据流

### 通知发布流程
```
应用 -> NotificationRequest -> Publish() -> AdvancedNotificationPublishService
                                                      |
                                                      v
                              +-----------------------+-----------------------+
                              |                       |                       |
                              v                       v                       v
                        [Preferences]           [Subscriber Manager]      [Distributed]
```

### 通知订阅流程
```
应用 -> IAnsSubscriber -> Subscribe() -> NotificationSubscriberManager
                                              |
                                              v
                                        [事件分发]
```

## 线程模型

### 主线程
- SA主线程处理IPC请求
- 回调在主线程执行

### 工作线程
- 数据库操作：RDB工作线程
- 分布式同步：Dsoftbus线程
- 图片处理：Image线程

## 依赖关系

```
┌──────────────┐         ┌──────────────┐
│   App        │         │  SystemUI    │
└──────┬───────┘         └──────┬───────┘
       │                        │
       v                        v
┌─────────────────────────────────────┐
│         frameworks/ans               │
│         (IPC Proxy)                 │
└─────────────────┬───────────────────┘
                  v
┌─────────────────────────────────────┐
│         services/ans                 │
│         (SA ID: 3203)               │
└─────────────────┬───────────────────┘
                  v
┌────────────┬────┴────┬────────────┐
│            │         │            │
v            v         v            v
[Preferences]  [RDB]  [Distributed]  [Subscriber]
```

## 关键文件索引

| 功能 | 关键文件 |
|------|----------|
| 主服务入口 | `services/ans/src/advanced_notification_service.cpp` |
| 订阅管理 | `services/ans/src/notification_subscriber_manager.cpp` |
| 发布服务 | `services/ans/src/advanced_notification_publish_service.cpp` |
| 通道服务 | `services/ans/src/advanced_notification_slot_service.cpp` |
| 配置管理 | `services/ans/src/notification_preferences.cpp` |
| 分布式管理 | `services/ans/src/distributed_manager/advanced_notification_distributed_manager_service.cpp` |
