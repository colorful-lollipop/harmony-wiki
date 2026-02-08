# 03_CodeMap - 目录结构与代码地图

## 文档说明

**目的**：提供完整的目录结构和代码导航图，帮助快速定位关键实现

**适用范围**：OpenHarmony ScreenLock Manager 代码库

**关键结论**：
- 项目采用清晰的分层架构（框架层/服务层）
- 代码组织良好，按功能模块划分
- 核心服务逻辑集中在 `services/` 目录

**相关链接**：
- [00_Overview](00_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 系统架构
- [01_NAPI_Reference](01_NAPI_Reference.md) - N-API 接口

---

## 1. 顶层目录结构（排除测试）

### 目录树

```
screenlock_mgr/
├── figures/                    # 架构图像文件
├── frameworks/                 # 框架层实现
│   ├── ets/ani/             # ArkTS/ANI 接口实现
│   ├── js/napi/             # N-API 接口实现
│   └── native/              # Native C++ 客户端库
├── interfaces/                # 对外接口定义
│   └── inner_api/           # 内部 API 接口
├── sa_profile/               # System Ability 配置
├── services/                # 锁屏服务核心实现
├── watch/                   # 可穿戴设备扩展
├── utils/                   # 工具类
├── BUILD.gn                 # 根构建入口
├── bundle.json             # 模块配置
├── screenlock.cfg          # 服务配置
└── screenlock.gni          # GN 变量定义
```

### 目录职责说明

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| **figures/** | 架构图和设计文档 | `subsystem_architecture_zh.png` |
| **frameworks/** | 框架层实现，提供应用接口 | 见下文详细说明 |
| **interfaces/** | 对外接口定义（Inner API） | `screenlock_manager.h` |
| **sa_profile/** | System Ability 配置文件 | `3704.json` |
| **services/** | 锁屏服务核心逻辑 | 见下文详细说明 |
| **watch/** | 可穿戴设备特定功能 | `watch_applock_manager.h` |
| **utils/** | 通用工具类 | 日志、常量定义 |
| **BUILD.gn** | 根构建配置 | 定义 `screenlock_mgr_packages` |
| **bundle.json** | 组件元数据 | 依赖、系统能力 |
| **screenlock.cfg** | Init 服务配置 | 创建数据目录 |
| **screenlock.gni** | GN 变量定义 | Feature 开关 |

---

## 2. 核心文件定位

### 2.1 入口文件

| 文件 | 用途 | 关键符号 |
|------|------|----------|
| **BUILD.gn** | 构建入口 | `screenlock_mgr_packages` |
| **sa_profile/3704.json** | SA 注册 | SA ID: 3704 |
| **services/src/screenlock_system_ability.cpp** | 服务主入口 | `ScreenLockSystemAbility::OnStart()` |
| **frameworks/js/napi/src/napi_screenlock_ability.cpp** | N-API 模块入口 | `ScreenlockInit()` |

**证据**：
- `sa_profile/3704.json:2` - SA ID 定义
- `services/include/screenlock_system_ability.h:129` - DECLARE_SYSTEM_ABILITY
- `frameworks/js/napi/src/napi_screenlock_ability.cpp:808` - napi_module_register

---

### 2.2 配置文件

| 文件 | 用途 | 关键配置 |
|------|------|----------|
| **bundle.json** | 组件元数据 | subsystem、syscap、deps |
| **screenlock.cfg** | Init 配置 | 创建 `/data/service/el1/public/screenlock/` |
| **screenlock.gni** | GN 变量 | `screenlock_mgr_so_crop`、`screenlock_mgr_wearable_enable_payment_app` |

**证据**：
- `bundle.json:2-16` - 组件定义
- `screenlock.cfg` - Init 脚本
- `screenlock.gni` - Feature 开关

---

### 2.3 核心实现文件

#### 服务端核心

| 文件 | 用途 | 关键类/函数 |
|------|------|-------------|
| **services/src/screenlock_system_ability.cpp** | 主服务实现 | `ScreenLockSystemAbility` |
| **services/src/screenlock_manager_stub.cpp** | IPC 请求处理 | `ScreenLockManagerStub::OnRemoteRequest()` |
| **services/src/strongauthmanager.cpp** | 强认证管理 | `StrongAuthManger` |
| **services/src/innerlistenermanager.cpp** | 监听器管理 | `InnerListenerManager` |

#### 客户端核心

| 文件 | 用途 | 关键类/函数 |
|------|------|-------------|
| **frameworks/native/src/screenlock_manager.cpp** | Native 客户端 API | `ScreenLockManager::GetInstance()` |
| **frameworks/native/src/screenlock_manager_proxy.cpp** | IPC 客户端代理 | `ScreenLockManagerProxy` |

#### N-API 核心

| 文件 | 用途 | 关键函数 |
|------|------|----------|
| **frameworks/js/napi/src/napi_screenlock_ability.cpp** | N-API 模块实现 | `Init()`, `Lock()`, `UnlockScreen()` |
| **frameworks/js/napi/src/screenlock_callback.cpp** | JS 回调处理 | `ScreenLockCallback` |

#### ANI 核心

| 文件 | 用途 | 关键函数 |
|------|------|----------|
| **frameworks/ets/ani/src/ani_screenlock_ability.cpp** | ANI 模块实现 | `AniScreenLockInit()` |
| **frameworks/ets/ani/src/ani_screenlock_callback.cpp** | ANI 回调处理 | `AniScreenLockCallback` |

---

## 3. 框架层详细结构

### 3.1 frameworks/js/napi/ - N-API 接口层

```
frameworks/js/napi/
├── include/                    # 头文件
│   ├── async_call.h          # 异步调用框架
│   ├── event_listener.h       # 事件监听器
│   ├── napi_screenlock_ability.h  # N-API 主要接口
│   ├── screenlock_callback.h  # JS 回调接口
│   ├── screenlock_js_util.h  # JS 工具函数
│   ├── screenlock_system_ability_callback.h  # SA 回调
│   └── uv_queue.h           # UV 队列处理
├── src/                      # 源文件
│   ├── async_call.cpp        # 异步调用实现
│   ├── napi_screenlock_ability.cpp  # **N-API 主实现文件**
│   ├── screenlock_callback.cpp      # 回调实现
│   ├── screenlock_js_util.cpp      # 工具函数实现
│   ├── screenlock_system_ability_callback.cpp  # SA 回调实现
│   └── uv_queue.cpp               # UV 队列实现
├── test/                     # 单元测试
└── BUILD.gn                  # 构建配置
```

**关键证据**：
- **N-API 注册点**：`src/napi_screenlock_ability.cpp:808-816`
- **JS 模块名**：`"screenlock"`
- **导出 API 数量**：15 个

**功能映射**：
| 功能 | 文件 | 关键函数 |
|------|------|----------|
| JS 接口注册 | `napi_screenlock_ability.cpp` | `Init()` |
| 异步调用 | `async_call.cpp` | `AsyncCall::Execute()` |
| 事件监听 | `event_listener.h` | `EventListener` |
| 回调处理 | `screenlock_callback.cpp` | `ScreenLockCallback::OnCallback()` |

---

### 3.2 frameworks/native/ - Native 客户端层

```
frameworks/native/
├── include/                    # 头文件
│   ├── screenlock_callback_stub.h        # 回调桩
│   ├── screenlock_inner_listener_stub.h  # 内部监听器桩
│   ├── screenlock_inner_listener_wapper.h # 监听器包装
│   ├── screenlock_manager_proxy.h       # IPC 代理
│   └── screenlock_system_ability_stub.h # SA 桩
└── src/                      # 源文件
    ├── screenlock_callback_stub.cpp        # 回调桩实现
    ├── screenlock_inner_listener_stub.cpp  # 内部监听器桩
    ├── screenlock_inner_listener_wapper.cpp # 监听器包装
    ├── screenlock_manager.cpp             # **Native 客户端主实现**
    ├── screenlock_manager_proxy.cpp       # IPC 代理实现
    └── screenlock_system_ability_stub.cpp # SA 桩实现
```

**关键证据**：
- **客户端入口**：`src/screenlock_manager.cpp` - `ScreenLockManager::GetInstance()`
- **IPC 代理**：`src/screenlock_manager_proxy.cpp`
- **输出库**：`libscreenlock_client.z.so`

**功能映射**：
| 功能 | 文件 | 关键类/函数 |
|------|------|-------------|
| Native API 暴露 | `screenlock_manager.cpp` | `ScreenLockManager` |
| IPC 通信 | `screenlock_manager_proxy.cpp` | `ScreenLockManagerProxy` |
| 回调处理 | `screenlock_callback_stub.cpp` | `ScreenLockCallbackStub` |

---

### 3.3 frameworks/ets/ani/ - ANI 接口层

```
frameworks/ets/ani/
├── include/                    # 头文件
│   ├── ani_error_handler.h           # 错误处理
│   ├── ani_event_listener.h          # ANI 事件监听器
│   ├── ani_screenlock_ability.h       # ANI 主要接口
│   ├── ani_screenlock_callback.h     # ANI 回调
│   ├── ani_screenlock_system_ability_callback.h  # ANI SA 回调
│   └── ani_screenlock_util.h       # ANI 工具函数
└── src/                      # 源文件
    ├── ani_screenlock_ability.cpp    # **ANI 主实现文件**
    ├── ani_screenlock_callback.cpp    # 回调实现
    └── ani_screenlock_util.cpp      # 工具函数实现
```

**关键证据**：
- **ANI 模块名**：`screenlock` (ArkTS)
- **输出文件**：`screenLock_static.abc` (ETS 字节码)

**功能映射**：
| 功能 | 文件 | 关键函数 |
|------|------|----------|
| ANI 接口注册 | `ani_screenlock_ability.cpp` | `AniScreenLockInit()` |
| ArkTS 回调 | `ani_screenlock_callback.cpp` | `AniScreenLockCallback` |

---

## 4. 服务层详细结构

### 4.1 services/ - 核心服务层

```
services/
├── include/                    # 头文件
│   ├── command.h                       # 命令处理
│   ├── commeventsubscriber.h            # 公共事件订阅
│   ├── dump_helper.h                   # Dump 调试
│   ├── innerlistenermanager.h           # **监听器管理**
│   ├── preferences_util.h               # 数据存储
│   ├── screenlock_callback_proxy.h      # 回调代理
│   ├── screenlock_get_info_callback.h   # 获取信息回调
│   ├── screenlock_inner_listener_proxy.h # 内部监听器代理
│   ├── screenlock_manager_stub.h       # **IPC 请求分发**
│   ├── screenlock_server_ipc_interface_code.h  # IPC 接口代码
│   ├── screenlock_system_ability.h      # **主服务类**
│   ├── screenlock_system_ability_proxy.h # 服务代理
│   └── strongauthmanager.h            # **强认证管理**
└── src/                      # 源文件
    ├── command.cpp                       # 命令实现
    ├── commeventsubscriber.cpp            # 公共事件实现
    ├── dump_helper.cpp                   # Dump 实现
    ├── innerlistenermanager.cpp           # 监听器管理实现
    ├── preferences_util.cpp               # 数据存储实现
    ├── screenlock_callback_proxy.cpp      # 回调代理实现
    ├── screenlock_get_info_callback.cpp   # 获取信息回调实现
    ├── screenlock_inner_listener_proxy.cpp # 内部监听器代理实现
    ├── screenlock_manager_stub.cpp       # **IPC 桩实现**
    ├── screenlock_system_ability.cpp      # **主服务实现**
    ├── screenlock_system_ability_proxy.cpp # 服务代理实现
    └── strongauthmanager.cpp            # 强认证管理实现
```

**关键证据**：
- **服务入口**：`src/screenlock_system_ability.cpp` - `ScreenLockSystemAbility::OnStart()`
- **IPC 桩**：`src/screenlock_manager_stub.cpp` - `ScreenLockManagerStub::OnRemoteRequest()`
- **SA ID**：`screenlock_server_ipc_interface_code.h` - `SAID = 3704`
- **输出库**：`libscreenlock_server.z.so`

**功能映射**：
| 功能 | 文件 | 关键类/函数 |
|------|------|-------------|
| 服务生命周期 | `screenlock_system_ability.cpp` | `ScreenLockSystemAbility::OnStart/OnStop()` |
| IPC 请求分发 | `screenlock_manager_stub.cpp` | `ScreenLockManagerStub::OnRemoteRequest()` |
| 强认证管理 | `strongauthmanager.cpp` | `StrongAuthManger` |
| 监听器管理 | `innerlistenermanager.cpp` | `InnerListenerManager` |
| 事件订阅 | `commeventsubscriber.cpp` | `CommEventSubscriber` |
| 数据存储 | `preferences_util.cpp` | `PreferencesUtil` |
| 调试支持 | `dump_helper.cpp` | `DumpHelper` |

---

## 5. 接口层详细结构

### 5.1 interfaces/inner_api/ - 内部 API 层

```
interfaces/inner_api/
├── include/                    # 头文件
│   ├── sclock_log.h                      # 日志宏
│   ├── screenlock_callback_interface.h       # 回调接口
│   ├── screenlock_common.h                # **公共定义**
│   ├── screenlock_inner_listener.h         # 内部监听器
│   ├── screenlock_inner_listener_interface.h # 内部监听器接口
│   ├── screenlock_manager.h               # **主要 API 类**
│   ├── screenlock_manager_interface.h     # IPC 接口定义
│   ├── screenlock_system_ability_interface.h # SA 回调接口
│   └── visibility.h                     # 可见性宏
├── BUILD.gn                           # 构建配置
└── screenlock_client.versionscript        # 版本脚本
```

**关键证据**：
- **公共定义**：`screenlock_common.h` - 错误码、枚举、常量
- **主 API 类**：`screenlock_manager.h` - `ScreenLockManager`
- **IPC 接口**：`screenlock_manager_interface.h` - `ScreenLockManagerInterface`
- **输出库**：`libscreenlock_client.z.so`

**重要定义**：
| 类型 | 定义位置 | 用途 |
|------|----------|------|
| 错误码 | `screenlock_common.h` | `ScreenLockError` 枚举 |
| 操作类型 | `screenlock_common.h` | `Action` 枚举 |
| 状态枚举 | `screenlock_system_ability.h` | `ScreenState`, `InteractiveState`, `AuthState` |
| 监听器类型 | `screenlock_common.h` | `ListenType` 枚举 |

---

## 6. 代码导航图

### 6.1 按功能查找代码

#### 锁屏/解锁功能

| 功能 | N-API 入口 | Native 客户端 | IPC 接口 | 服务端实现 |
|------|------------|--------------|----------|------------|
| **锁定屏幕** | `napi_screenlock_ability.cpp:Lock()` | `screenlock_manager.cpp:Lock()` | `LOCK` (2) | `screenlock_system_ability.cpp:Lock()` |
| **解锁屏幕** | `napi_screenlock_ability.cpp:UnlockScreen()` | `screenlock_manager.cpp:UnlockScreen()` | `UNLOCKSCREEN` (IPC 代码) | `screenlock_system_ability.cpp:UnlockScreen()` |
| **请求解锁** | `napi_screenlock_ability.cpp:Unlock()` | `screenlock_manager.cpp:Unlock()` | `UNLOCK` (3) | `screenlock_system_ability.cpp:Unlock()` |

#### 状态查询功能

| 功能 | N-API 入口 | Native 客户端 | IPC 接口 | 服务端实现 |
|------|------------|--------------|----------|------------|
| **查询锁屏状态** | `napi_screenlock_ability.cpp:IsScreenLocked()` | `screenlock_manager.cpp:IsScreenLocked()` | `IS_SCREEN_LOCKED` | `screenlock_system_ability.cpp:IsScreenLocked()` |
| **查询安全模式** | `napi_screenlock_ability.cpp:IsSecureMode()` | `screenlock_manager.cpp:IsSecureMode()` | `IS_SECURE_MODE` | `screenlock_system_ability.cpp:IsSecureMode()` |
| **查询设备锁定** | `napi_screenlock_ability.cpp:IsDeviceLocked()` | `screenlock_manager.cpp:IsDeviceLocked()` | `IS_DEVICE_LOCKED` | `screenlock_system_ability.cpp:IsDeviceLocked()` |

#### 认证管理功能

| 功能 | N-API 入口 | Native 客户端 | IPC 接口 | 服务端实现 |
|------|------------|--------------|----------|------------|
| **设置认证状态** | `napi_screenlock_ability.cpp:SetScreenLockAuthState()` | `screenlock_manager.cpp:SetScreenLockAuthState()` | `SET_SCREEN_LOCK_AUTH_STATE` (6) | `screenlock_system_ability.cpp:SetScreenLockAuthState()` |
| **获取认证状态** | `napi_screenlock_ability.cpp:GetScreenLockAuthState()` | `screenlock_manager.cpp:GetScreenLockAuthState()` | `GET_SCREEN_LOCK_AUTH_STATE` (7) | `screenlock_system_ability.cpp:GetScreenLockAuthState()` |
| **请求强认证** | `napi_screenlock_ability.cpp:RequestStrongAuth()` | `screenlock_manager.cpp:RequestStrongAuth()` | `REQUEST_STRONG_AUTH` (8) | `screenlock_system_ability.cpp:RequestStrongAuth()` |
| **获取强认证** | `napi_screenlock_ability.cpp:GetStrongAuth()` | `screenlock_manager.cpp:GetStrongAuth()` | `GET_STRONG_AUTH` (9) | `screenlock_system_ability.cpp:GetStrongAuth()` |

#### 事件监听功能

| 功能 | N-API 入口 | Native 客户端 | IPC 接口 | 服务端实现 |
|------|------------|--------------|----------|------------|
| **注册系统事件** | `napi_screenlock_ability.cpp:OnSystemEvent()` | `screenlock_manager.cpp:RegisterCallback()` | `ONSYSTEMEVENT` (1) | `innerlistenermanager.cpp:RegisterListener()` |
| **发送锁屏事件** | `napi_screenlock_ability.cpp:SendScreenLockEvent()` | `screenlock_manager.cpp:SendScreenLockEvent()` | `SEND_SCREENLOCK_EVENT` (3) | `screenlock_system_ability.cpp:SendScreenLockEvent()` |

#### 锁屏禁用功能

| 功能 | N-API 入口 | Native 客户端 | IPC 接口 | 服务端实现 |
|------|------------|--------------|----------|------------|
| **查询禁用状态** | `napi_screenlock_ability.cpp:IsScreenLockDisabled()` | `screenlock_manager.cpp:IsScreenLockDisabled()` | `IS_SCREEN_LOCK_DISABLED` (4) | `screenlock_system_ability.cpp:IsScreenLockDisabled()` |
| **设置禁用状态** | `napi_screenlock_ability.cpp:SetScreenLockDisabled()` | `screenlock_manager.cpp:SetScreenLockDisabled()` | `SET_SCREEN_LOCK_DISABLED` (5) | `screenlock_system_ability.cpp:SetScreenLockDisabled()` |

---

### 6.2 按层次查找代码

#### 应用层（JavaScript/ArkTS）

```
用户应用
    ↓
[napi_screenlock_ability.cpp] 或 [ani_screenlock_ability.cpp]
```

**关键文件**：
- `frameworks/js/napi/src/napi_screenlock_ability.cpp` - JavaScript API
- `frameworks/ets/ani/src/ani_screenlock_ability.cpp` - ArkTS API

---

#### 框架层（Native C++）

```
Native 应用
    ↓
[screenlock_manager.cpp] - 客户端 API
    ↓
[screenlock_manager_proxy.cpp] - IPC 代理
```

**关键文件**：
- `frameworks/native/src/screenlock_manager.cpp` - Native 客户端
- `frameworks/native/src/screenlock_manager_proxy.cpp` - IPC 代理

---

#### IPC 边界

```
[IPC Proxy] ←→ Binder/HIDL ←→ [IPC Stub]
```

**关键文件**：
- `frameworks/native/src/screenlock_manager_proxy.cpp` - 客户端
- `services/src/screenlock_manager_stub.cpp` - 服务端

---

#### 服务层（System Service）

```
[screenlock_manager_stub.cpp] - IPC 请求分发
    ↓
[screenlock_system_ability.cpp] - 主服务逻辑
    ↓
├── [strongauthmanager.cpp] - 强认证管理
├── [innerlistenermanager.cpp] - 监听器管理
├── [commeventsubscriber.cpp] - 事件订阅
└── [preferences_util.cpp] - 数据存储
```

**关键文件**：
- `services/src/screenlock_system_ability.cpp` - 主服务
- `services/src/strongauthmanager.cpp` - 强认证
- `services/src/innerlistenermanager.cpp` - 监听器
- `services/src/commeventsubscriber.cpp` - 事件订阅
- `services/src/preferences_util.cpp` - 数据存储

---

### 6.3 按模块查找代码

#### 权限检查模块

| 文件 | 关键函数 |
|------|----------|
| `services/src/screenlock_system_ability.cpp` | `CheckPermission()` |
| `interfaces/inner_api/include/screenlock_common.h` | `ScreenLockError` 枚举 |

#### 认证模块

| 文件 | 关键函数 |
|------|----------|
| `services/src/strongauthmanager.cpp` | `StrongAuthManger::RequestStrongAuth()` |
| `services/src/screenlock_system_ability.cpp` | `SetScreenLockAuthState()` |

#### 监听器模块

| 文件 | 关键函数 |
|------|----------|
| `services/src/innerlistenermanager.cpp` | `InnerListenerManager::RegisterListener()` |
| `services/src/innerlistenermanager.cpp` | `InnerListenerManager::NotifyEvent()` |
| `interfaces/inner_api/include/screenlock_inner_listener_interface.h` | `InnerListenerIf` 接口 |

#### 数据存储模块

| 文件 | 关键函数 |
|------|----------|
| `services/src/preferences_util.cpp` | `PreferencesUtil::WriteValue()` |
| `services/src/preferences_util.cpp` | `PreferencesUtil::ReadValue()` |

---

## 7. 可穿戴设备扩展

### watch/ 目录结构

```
watch/
├── include/                    # 头文件
│   ├── hisysevent_report.h             # HiSysEvent 报告
│   ├── setting_manager.h                # 设置管理
│   ├── watch_applock_manager.h          # **应用锁管理**
│   └── wear_detection_observer.h         # **穿戴检测**
└── src/                      # 源文件
    ├── hisysevent_report.cpp             # HiSysEvent 实现
    ├── setting_manager.cpp                # 设置实现
    ├── watch_applock_manager.cpp          # 应用锁实现
    └── wear_detection_observer.cpp         # 穿戴检测实现
```

**关键证据**：
- **应用锁管理器**：`watch_applock_manager.h/cpp`
- **穿戴检测**：`wear_detection_observer.h/cpp`
- **Feature 开关**：`screenlock_mgr_wearable_enable_payment_app`

**功能**：
- 监测设备是否被佩戴
- 在设备未佩戴时锁定支付应用
- 报告穿戴状态到系统

---

## 8. 编译产物映射

### 8.1 输出库文件

| 库文件 | BUILD 目标 | 源文件 | 安装路径 |
|--------|-----------|---------|----------|
| **libscreenlock_server.z.so** | `services:screenlock_server` | `services/src/*.cpp` | `system/lib/` |
| **libscreenlock_client.z.so** | `inner_api:screenlock_client` | `frameworks/native/src/*.cpp` | `system/lib/` |
| **libscreenlockability.z.so** | `napi:screenlock` | `frameworks/js/napi/src/*.cpp` | `system/lib/module/app/` |
| **screenLock_static.abc** | `ani:screenLock_static` | `frameworks/ets/ani/src/*.cpp` | `system/framework/` |

**证据来源**：
- `services/BUILD.gn:26-50`
- `interfaces/inner_api/BUILD.gn:28-90`
- `frameworks/js/napi/BUILD.gn:24-80`
- `frameworks/ets/ani/BUILD.gn`

### 8.2 配置文件

| 文件 | BUILD 目标 | 安装路径 |
|------|-----------|----------|
| **screenlock.cfg** | `:screenlock_cfg` | `/etc/init/` |
| **3704.json** | `sa_profile:screenlock_sa_profiles` | `/system/profile/` |

---

## 9. 快速查找表

### 按文件类型查找

| 文件类型 | 目录 | 示例 |
|----------|------|------|
| **头文件 (.h)** | 所有 include/ 目录 | `services/include/screenlock_system_ability.h` |
| **实现文件 (.cpp)** | 所有 src/ 目录 | `services/src/screenlock_system_ability.cpp` |
| **构建文件 (BUILD.gn)** | 所有模块目录 | `services/BUILD.gn` |
| **配置文件 (.json, .cfg, .gni)** | 根目录 | `bundle.json` |

### 按关键词查找

| 关键词 | 相关文件 |
|--------|----------|
| **SA / System Ability** | `services/src/screenlock_system_ability.cpp` |
| **IPC** | `services/src/screenlock_manager_stub.cpp`, `frameworks/native/src/screenlock_manager_proxy.cpp` |
| **N-API** | `frameworks/js/napi/src/napi_screenlock_ability.cpp` |
| **权限** | `services/src/screenlock_system_ability.cpp:CheckPermission()` |
| **监听器** | `services/src/innerlistenermanager.cpp` |
| **认证** | `services/src/strongauthmanager.cpp` |
| **事件** | `services/src/commeventsubscriber.cpp` |

---

## 10. 相关文档

- [00_Overview](00_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 系统架构
- [01_NAPI_Reference](01_NAPI_Reference.md) - N-API 接口
- [03_GN_Build](03_GN_Build.md) - 构建系统

---

**更新日期**：2026-02-07
**文档状态**：完成
