# 00_Overview - 项目概览

## 1. 项目定位

### 1.1 简介

`screenlock_mgr` 是 OpenHarmony 系统中负责屏幕锁定管理的核心子系统，提供以下能力：

- **应用层**: 为第三方应用提供屏幕解锁/锁定状态查询、密码设置等接口
- **系统层**: 提供屏幕状态事件回调，支持用户切换、屏保管理等系统级功能

### 1.2 适用范围

| 维度 | 说明 |
|------|------|
| 系统类型 | Standard (标准系统) |
| 适用设备 | 手机、平板、穿戴设备等 |
| API 版本 | 7+ |

### 1.3 核心概念

| 概念 | 说明 |
|------|------|
| ScreenLock | 屏幕锁定管理器，负责协调锁屏相关功能 |
| Strong Auth | 强认证机制，用于安全要求较高的场景 |
| Screen On/Off | 屏幕开关状态管理 |
| User Switch | 用户切换事件处理 |
| SA (System Ability) | 系统能力，锁屏服务作为 SA 运行 |

---

## 2. 目录结构

```
screenlock_mgr/
├── figures/                     # 架构图和设计文档
│   └── subsystem_architecture_zh.png
│
├── frameworks/                   # 框架层
│   ├── ets/ani/                 # ETS (ArkUI) 接口实现
│   │   ├── ets/
│   │   │   └── @ohos.screenLock.ets  # ETS API 定义
│   │   ├── include/            # 头文件
│   │   │   ├── ani_error_handler.h
│   │   │   ├── ani_event_listener.h
│   │   │   ├── ani_screenlock_ability.h
│   │   │   ├── ani_screenlock_callback.h
│   │   │   └── ani_screenlock_util.h
│   │   └── src/                 # C++ 实现
│   │       ├── ani_screenlock_ability.cpp
│   │       ├── ani_screenlock_callback.cpp
│   │       └── ...
│   │
│   ├── js/napi/                 # N-API 接口实现
│   │   ├── include/
│   │   │   ├── async_call.h
│   │   │   ├── event_listener.h
│   │   │   ├── napi_screenlock_ability.h
│   │   │   ├── screenlock_callback.h
│   │   │   └── ...
│   │   ├── src/
│   │   │   ├── napi_screenlock_ability.cpp   # N-API 注册和实现
│   │   │   ├── async_call.cpp
│   │   │   ├── screenlock_callback.cpp
│   │   │   └── ...
│   │   └── test/               # 单元测试
│   │
│   └── native/                 # Native 接口 (供系统应用)
│       ├── include/
│       │   ├── screenlock_callback_stub.h
│       │   ├── screenlock_manager_proxy.h
│       │   └── screenlock_system_ability_stub.h
│       └── src/
│           ├── screenlock_manager.cpp
│           ├── screenlock_manager_proxy.cpp
│           └── ...
│
├── interfaces/inner_api/       # 内部 API 接口
│   ├── include/
│   │   ├── sclock_log.h
│   │   ├── screenlock_callback_interface.h
│   │   ├── screenlock_common.h        # 公共定义 (错误码、常量)
│   │   ├── screenlock_inner_listener.h
│   │   ├── screenlock_manager.h
│   │   ├── screenlock_manager_interface.h  # 接口定义
│   │   └── screenlock_system_ability_interface.h
│   ├── BUILD.gn
│   └── screenlock_client.versionscript
│
├── services/                    # 锁屏服务核心实现
│   ├── include/
│   │   ├── command.h
│   │   ├── commeventsubscriber.h
│   │   ├── dump_helper.h
│   │   ├── innerlistenermanager.h
│   │   ├── preferences_util.h
│   │   ├── screenlock_callback_proxy.h
│   │   ├── screenlock_get_info_callback.h
│   │   ├── screenlock_inner_listener_proxy.h
│   │   ├── screenlock_manager_stub.h
│   │   ├── screenlock_server_ipc_interface_code.h
│   │   ├── screenlock_system_ability.h        # SA 实现
│   │   ├── screenlock_system_ability_proxy.h
│   │   └── strongauthmanager.h
│   ├── src/
│   │   ├── screenlock_system_ability.cpp     # SA 主实现
│   │   ├── screenlock_manager_stub.cpp
│   │   ├── screenlock_manager_stub.cpp
│   │   ├── strongauthmanager.cpp
│   │   ├── innerlistenermanager.cpp
│   │   └── ...
│   └── BUILD.gn
│
├── watch/                      # 可穿戴设备支持
│   ├── include/
│   │   ├── hisysevent_report.h
│   │   ├── setting_manager.h
│   │   ├── watch_applock_manager.h
│   │   └── wear_detection_observer.h
│   └── src/
│       ├── watch_applock_manager.cpp
│       └── ...
│
├── sa_profile/                 # SA 配置
│   ├── 3704.json              # SA ID 3704 配置
│   └── BUILD.gn
│
├── utils/                      # 工具类
│
├── BUILD.gn                    # 根构建入口
├── bundle.json                # 模块配置
├── screenlock.cfg             # 服务配置
└── screenlock.gni             # GN 变量定义
```

---

## 3. 模块职责

| 模块 | 职责 |
|------|------|
| `services/` | 锁屏服务核心逻辑、状态管理、事件处理 |
| `frameworks/js/napi/` | JS 接口适配层、N-API 实现 |
| `frameworks/ets/ani/` | ETS/ArkUI 接口适配层 |
| `frameworks/native/` | Native 接口 (供系统应用使用) |
| `interfaces/inner_api/` | 内部 API 定义和接口 |
| `watch/` | 可穿戴设备特定功能 (支付应用) |

---

## 4. 运行环境

### 4.1 依赖的子系统

| 子系统 | 用途 |
|--------|------|
| `ability_runtime` | 能力运行时 |
| `access_token` | 权限管理 |
| `common_event_service` | 公共事件 |
| `ipc` | 进程间通信 |
| `os_account` | 用户账户 |
| `safwk` | 系统能力框架 |
| `samgr` | 服务管理 |
| `user_auth_framework` | 用户认证 |
| `window_manager` | 窗口管理 |

### 4.2 系统服务

- **SA ID**: 3704
- **进程**: foundation
- **库**: `libscreenlock_server.z.so`

---

## 5. 关键配置

### 5.1 bundle.json 配置

```json
{
  "name": "@ohos/screenlock_mgr",
  "subsystem": "theme",
  "syscap": ["SystemCapability.MiscServices.ScreenLock"],
  "features": [
    "screenlock_mgr_so_crop",
    "screenlock_mgr_wearable_enable_payment_app"
  ]
}
```

### 5.2 特性开关

| 开关 | 说明 |
|------|------|
| `screenlock_mgr_so_crop` | 功能裁剪，减小二进制体积 |
| `screenlock_mgr_wearable_enable_payment_app` | 可穿戴设备支付应用支持 |

---

## 6. 相关文档

| 文档 | 链接 |
|------|------|
| N-API 参考 | [01_NAPI_Reference](01_NAPI_Reference.md) |
| 架构说明 | [02_Architecture](02_Architecture.md) |
| 构建系统 | [03_GN_Build](03_GN_Build.md) |
| 安全评审 | [04_Security_Review](04_Security_Review.md) |
