# 架构设计

## 1. 整体架构

### 1.1 架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层 (JS/TS)                           │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  ohos.distributedDeviceManager / ohos.distributedHardware   │  │
│  │  .deviceManager                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     N-API 层 (JS Binding)                       │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  interfaces/kits/js4.0/src/native_devicemanager_js.cpp   │  │
│  │  - JS ↔ C++ 绑定                                          │  │
│  │  - 异步回调管理                                            │  │
│  │  - 事件注册/注销                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     SDK 层 (Inner API)                          │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  interfaces/inner_kits/native_cpp/                       │  │
│  │  - IPC 客户端封装                                          │  │
│  │  - 设备状态管理                                            │  │
│  │  - 认证流程控制                                            │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
┌───────────────────────────┐ ┌───────────────────────────┐
│     DSoftBus 子系统        │ │     DeviceAuth 子系统     │
│  - 设备发现                 │ │  - 群组管理                │
│  - 设备上下线通知           │ │  - 认证流程                │
│  - 认证通道建立             │ │  - 凭据管理                │
└───────────────────────────┘ └───────────────────────────┘
                    │                   │
                    └─────────┬─────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   DeviceManager Service (SA)                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  services/service/ + services/implementation/             │  │
│  │  - 设备状态管理                                              │  │
│  │  - 认证流程控制                                              │  │
│  │  - 软总线交互                                                │  │
│  │  - HiChain 交互                                             │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        系统底层                                  │
│  - IPC 通信 (IPCSkeleton)                                       │
│  - 系统能力管理 (SAMgr)                                         │
│  - 权限管理 (AccessToken)                                       │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 分层说明

| 层级 | 职责 | 代码位置 |
|-----|------|---------|
| **应用层** | 调用 JS API 进行设备管理 | 第三方应用 |
| **N-API 层** | JS ↔ C++ 绑定，参数转换，异步回调 | `interfaces/kits/js4.0/` |
| **SDK 层** | 设备管理逻辑，IPC 客户端 | `interfaces/inner_kits/native_cpp/` |
| **服务层** | SA 核心实现，设备认证，发现控制 | `services/` |

> 证据来源：`README_zh.md:7-9` 架构图

## 2. 组件职责

### 2.1 N-API 模块

**文件**：`interfaces/kits/js4.0/src/native_devicemanager_js.cpp`

**职责**：
- 模块注册：`napi_module_register(&g_dmModule)` (行 3014)
- JS 方法导出
- 异步回调管理（UV 队列）
- 事件监听注册/注销

**关键类**：
- `DeviceManagerNapi`：设备管理器 NAPI 封装
- `DmNapiInitCallback`：初始化回调
- `DmNapiDeviceStatusCallback`：设备状态回调
- `DmNapiDiscoveryCallback`：发现回调
- `DmNapiAuthenticateCallback`：认证回调

> 证据来源：`native_devicemanager_js.cpp:1-100`

### 2.2 Inner SDK 模块

**文件**：`interfaces/inner_kits/native_cpp/`

**职责**：
- IPC 通信客户端
- 设备信息查询
- 设备状态同步
- 认证请求发起

**目录结构**：
```
interfaces/inner_kits/native_cpp/
├── include/
│   ├── ipc/           # IPC 头文件
│   │   ├── lite/      # 轻量系统
│   │   └── standard/  # 标准系统
│   └── notify/        # 回调通知头文件
└── src/
    ├── ipc/           # IPC 核心代码
    └── notify/        # 回调通知实现
```

> 证据来源：`README_zh.md:43-53`

### 2.3 Service 实现模块

**文件**：`services/implementation/`

**核心功能模块**：

| 模块 | 路径 | 职责 |
|-----|------|------|
| **设备状态管理** | `src/devicestate/` | 设备上下线状态维护 |
| **设备发现** | `src/discovery/` | 发现流程控制 |
| **设备认证** | `src/authentication/` | PIN 码认证流程 |
| **凭据管理** | `src/credential/` | 认证凭据存储 |
| **HiChain 交互** | `src/dependency/hichain/` | 与 deviceauth 交互 |
| **软总线交互** | `src/dependency/softbus/` | 与 dsoftbus 交互 |

**关键源文件**：
- `device_manager_service_impl.cpp`：服务入口
- `dm_device_state_manager.cpp`：设备状态管理
- `dm_auth_manager.cpp`：认证管理器
- `softbus_connector.cpp`：软总线连接器
- `hichain_connector.cpp`：HiChain 连接器

> 证据来源：`README_zh.md:82-98`

## 3. 数据流

### 3.1 设备发现流程

```
JS App
    │
    ▼
startDiscovering(params)  ──► N-API 层
    │
    ▼
IPC 调用 ──► DeviceManager Service
    │
    ▼
DSoftBus ──► 发布发现请求
    │
    ▼
发现结果回调 ──► Service
    │
    ▼
IPC 返回 ──► N-API 层
    │
    ▼
discoverSuccess 事件 ──► JS App
```

### 3.2 设备认证流程

```
JS App (发起端)
    │
    ▼
bindTarget(deviceId, bindParam)  ──► N-API 层
    │
    ▼
IPC 调用 ──► DeviceManager Service
    │
    ▼
HiChain ──► 发起认证请求
    │
    ├──► [认证被控端] ──► PIN 码确认
    │
    ▼
认证结果 ──► Service
    │
    ▼
IPC 返回 ──► N-API 层
    │
    ▼
认证成功回调 ──► JS App
```

## 4. 线程模型

### 4.1 线程划分

| 线程 | 职责 | 备注 |
|-----|------|------|
| **JS 主线程** | JS 事件循环，API 调用 | |
| **UV 工作线程池** | 异步操作执行 | N-API 回调处理 |
| **IPC 通信线程** | 跨进程通信 | 系统 IPC 框架管理 |
| **Service 主线程** | SA 消息处理 | |
| **软总线线程** | 设备发现/认证 | DSoftBus 内部线程 |

### 4.2 异步回调机制

N-API 层使用 libuv 实现异步回调：

```cpp
// 证据来源：native_devicemanager_js.cpp:45
uv_queue_work_with_qos_internal(loop, work,
    [] (uv_work_t *work) { /* 工作执行 */ },
    [] (uv_work_t *work, int status) { /* 回调处理 */ },
    uv_qos_user_initiated, "Dm_DeviceIconInfo_OnResult");
```

## 5. IPC 通信机制

### 5.1 标准系统 IPC

**头文件**：`services/implementation/include/ipc/standard/`

**通信模式**：
- `IPC_PATTERN_RPC`：远程过程调用
- `IPC_PATTERN_LISTEN`：监听模式

**关键文件**：
- `ipc_skeleton.h`：IPC 骨架
- `ipc_req.h`/`ipc_rsp.h`：请求/响应

### 5.2 轻量系统 IPC

**头文件**：`services/implementation/include/ipc/lite/`

**适配**：Hi3516DV300 等轻量设备

## 6. 生命周期

### 6.1 服务生命周期

```
系统启动 ──► BOOT_COMPLETED 事件 ──► DeviceManager SA 启动
                                        │
                                        ▼
                                   初始化完成
                                        │
                    ┌───────────────────┼───────────────────┐
                    ▼                   ▼                   ▼
               设备发现请求         设备认证请求         设备状态查询
                    │                   │                   │
                    ▼                   ▼                   ▼
               返回结果             返回结果             返回结果
```

### 6.2 实例生命周期

```
createDeviceManager() ──► DeviceManager 实例创建
                              │
                              ▼
                        设备管理操作
                              │
                              ▼
releaseDeviceManager() ──► 实例销毁
```

## 7. 扩展机制

### 7.1 UI 回调

**用途**：PIN 码输入界面交互

**事件**：`uiStateChange`

**触发条件**：
- 认证被控端需要用户确认
- PIN 码显示/输入完成

```javascript
dmClass.on('uiStateChange', (data) => {
    const param = JSON.parse(data.param);
    // 处理 UI 状态变更
});
```

### 7.2 PIN 码显示 HAP

**组件**：`display/` 目录

**功能**：
- 授权提示页面
- PIN 码显示页面
- PIN 码输入页面

**运行方式**：作为系统应用 `DeviceManager_UI.hap` 预置

> 证据来源：`README_zh.md:389-410`
