# 内部 API

## 1. 模块概览

### 1.1 内部 API 分层

```
┌─────────────────────────────────────────┐
│           N-API 层 (对外)                │
│  interfaces/kits/js4.0/                 │
└─────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│          Inner SDK 层 (对内)             │
│  interfaces/inner_kits/native_cpp/       │
│  - IPC 客户端                           │
│  - 设备状态管理                         │
│  - 认证流程控制                         │
└─────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│          Service 层 (核心)               │
│  services/implementation/               │
│  - SA 核心实现                          │
│  - 设备认证                             │
│  - 设备发现                             │
│  - 软总线交互                           │
└─────────────────────────────────────────┘
```

## 2. Inner SDK 模块

### 2.1 目录结构

```
interfaces/inner_kits/native_cpp/
├── include/
│   ├── ipc/
│   │   ├── lite/              # 轻量系统 IPC 头文件
│   │   │   ├── ipc_skeleton.h
│   │   │   ├── ipc_req.h
│   │   │   └── ipc_rsp.h
│   │   └── standard/          # 标准系统 IPC 头文件
│   │       ├── ipc_skeleton.h
│   │       ├── ipc_req.h
│   │       ├── ipc_rsp.h
│   │       └── ...
│   └── notify/               # 回调通知头文件
│       ├── device_status_callback.h
│       ├── discovery_callback.h
│       └── auth_callback.h
└── src/
    ├── ipc/                   # IPC 核心实现
    │   ├── lite/
    │   └── standard/
    └── notify/               # 回调通知实现
        ├── device_status_callback.cpp
        ├── discovery_callback.cpp
        └── auth_callback.cpp
```

### 2.2 稳定性标注

| 层级 | 稳定性 | 说明 |
|-----|-------|------|
| `interfaces/kits/` | **稳定** | 对外 JS API |
| `interfaces/inner_kits/` | **不稳定** | 仅供框架内部使用 |
| `services/` | **不稳定** | SA 内部实现 |

> 稳定性判断依据：目录层级和命名约定

## 3. IPC 通信接口

### 3.1 IPC 命令码

```cpp
// 设备发现相关
CMD_START_DISCOVERY = 0x01
CMD_STOP_DISCOVERY = 0x02

// 设备认证相关
CMD_AUTH_DEVICE = 0x10
CMD_UNAUTH_DEVICE = 0x11
CMD_VERIFY_AUTH = 0x12

// 设备信息相关
CMD_GET_TRUSTED_DEVICE_LIST = 0x20
CMD_GET_LOCAL_DEVICE_INFO = 0x21

// 订阅相关
CMD_SUBSCRIBE_DEVICE_STATE = 0x30
CMD_UNSUBSCRIBE_DEVICE_STATE = 0x31
```

### 3.2 IPC 请求结构

```cpp
struct IpcReq {
    std::string bundleName;      // 调用方包名
    int32_t cmdCode;              // 命令码
    MessageParcel data;           // 请求数据
    // ...
};

struct IpcRsp {
    int32_t errCode;             // 错误码
    MessageParcel data;           // 响应数据
    // ...
};
```

### 3.3 IPC 客户端封装

**核心类**：`DeviceManager`

```cpp
class DeviceManager {
public:
    // 设备发现
    int32_t StartDiscovery(const std::string& subscribeInfo);
    int32_t StopDiscovery(int32_t subscribeId);

    // 设备认证
    int32_t AuthenticateDevice(const std::string& deviceInfo,
                               const std::string& authParam,
                               AuthCallback& callback);
    int32_t UnAuthenticateDevice(const std::string& deviceInfo);

    // 设备信息
    int32_t GetTrustedDeviceList(std::vector<DmDeviceInfo>& deviceList);
    int32_t GetLocalDeviceInfo(DmDeviceInfo& deviceInfo);

    // 设备发布
    int32_t PublishDeviceDiscovery(const std::string& publishInfo);
    int32_t UnPublishDeviceDiscovery(int32_t publishId);
};
```

## 4. Service 核心模块

### 4.1 服务入口

**文件**：`services/implementation/src/device_manager_service_impl.cpp`

**核心职责**：
- SA 生命周期管理
- IPC 请求分发
- 模块初始化

```cpp
class DeviceManagerServiceImpl : public SystemAbility {
public:
    // IPC 请求处理
    int32_t OnRemoteRequest(uint32_t code,
                            MessageParcel& data,
                            MessageParcel& reply);

    // 设备发现
    int32_t StartDiscovery(int32_t subscribeId, const DmSubscribeInfo& info);
    int32_t StopDiscovery(int32_t subscribeId);

    // 设备认证
    int32_t AuthenticateDevice(const DmDeviceInfo& deviceInfo,
                               const std::string& authParam,
                               int32_t& authId);
    int32_t UnAuthenticateDevice(const std::string& deviceId);
};
```

### 4.2 设备状态管理

**目录**：`services/implementation/src/devicestate/`

**核心文件**：`dm_device_state_manager.cpp`

**职责**：
- 设备上下线状态维护
- 设备信息变更通知
- 设备列表缓存管理

```cpp
class DmDeviceStateManager {
public:
    // 设备上线
    int32_t OnDeviceOnline(const DmDeviceInfo& deviceInfo);

    // 设备下线
    int32_t OnDeviceOffline(const std::string& deviceId);

    // 设备信息变更
    int32_t OnDeviceInfoChanged(const DmDeviceInfo& deviceInfo);

    // 获取设备状态
    DeviceState GetDeviceState(const std::string& deviceId);
};
```

### 4.3 设备认证管理

**目录**：`services/implementation/src/authentication/`

**核心文件**：`dm_auth_manager.cpp`

**职责**：
- 认证流程控制
- PIN 码处理
- 认证结果回调

```cpp
class DmAuthManager {
public:
    // 开始认证
    int32_t StartAuth(const std::string& deviceId,
                      int32_t authType,
                      const std::string& param);

    // 取消认证
    int32_t CancelAuth(const std::string& authId);

    // 处理 PIN 码
    int32_t InputPinCode(const std::string& authId,
                         const std::string& pinCode);

    // 认证结果回调
    void OnAuthResult(const std::string& authId,
                      int32_t result,
                      const std::string& data);
};
```

### 4.4 设备发现管理

**目录**：`services/implementation/src/discovery/`

**核心文件**：`dm_discovery_manager.cpp`

**职责**：
- 发现请求处理
- 发现结果分发
- 发现订阅管理

```cpp
class DmDiscoveryManager {
public:
    // 开始发现
    int32_t StartDiscovery(int32_t subscribeId,
                            const DmSubscribeInfo& info);

    // 停止发现
    int32_t StopDiscovery(int32_t subscribeId);

    // 发布设备
    int32_t PublishDiscovery(const DmPublishInfo& info);

    // 取消发布
    int32_t UnPublishDiscovery(int32_t publishId);

    // 发现结果回调
    void OnDeviceFound(const std::string& deviceId,
                       const DmDeviceInfo& deviceInfo);
};
```

## 5. 外部依赖模块

### 5.1 HiChain 交互

**目录**：`services/implementation/src/dependency/hichain/`

**核心文件**：
- `hichain_connector.cpp`：HiChain 连接器
- `hichain_auth_connector.cpp`：认证交互

**职责**：
- 设备群组管理
- 认证凭据交换
- 信任关系建立

```cpp
class HiChainConnector {
public:
    // 初始化
    int32_t Initialize();

    // 创建群组
    int32_t CreateGroup(const std::string& groupName);

    // 删除群组
    int32_t DeleteGroup(const std::string& groupId);

    // 获取设备凭据
    int32_t GetDeviceCredential(const std::string& deviceId,
                                 Credential& credential);

    // 认证设备
    int32_t AuthenticateDevice(const std::string& deviceId,
                              const std::string& groupId,
                              AuthResult& result);
};
```

### 5.2 DSoftBus 交互

**目录**：`services/implementation/src/dependency/softbus/`

**核心文件**：
- `softbus_connector.cpp`：软总线连接器
- `softbus_session.cpp`：会话管理

**职责**：
- 设备发现
- 设备上下线通知
- 认证通道建立

```cpp
class SoftBusConnector {
public:
    // 启动发现
    int32_t StartDiscovery(int32_t subscribeId,
                          const DmSubscribeInfo& info);

    // 停止发现
    int32_t StopDiscovery(int32_t subscribeId);

    // 发布设备
    int32_t PublishDevice(const DmPublishInfo& info);

    // 取消发布
    int32_t UnPublishDevice(int32_t publishId);

    // 获取本地设备信息
    int32_t GetLocalDeviceInfo(DmDeviceInfo& deviceInfo);

    // 获取可信设备列表
    int32_t GetTrustedDeviceList(std::vector<DmDeviceInfo>& list);
};
```

### 5.3 事件管理

**目录**：`services/implementation/src/dependency/commonevent/`

**核心文件**：`dm_common_event_manager.cpp`

**职责**：
- 系统事件订阅
- 事件发布

```cpp
class DmCommonEventManager {
public:
    // 订阅事件
    int32_t SubscribeEvent(const CommonEventSubscribeInfo& info,
                           const CommonEventCallback& callback);

    // 发布事件
    int32_t PublishEvent(const CommonEventData& data);

    // 取消订阅
    int32_t UnsubscribeEvent(const std::string& subscriberId);
};
```

## 6. 回调机制

### 6.1 回调类型

| 回调类型 | 说明 | 使用场景 |
|---------|------|---------|
| `DeviceStatusCallback` | 设备状态变更回调 | 设备上下线 |
| `DiscoveryCallback` | 发现回调 | 发现结果 |
| `AuthCallback` | 认证回调 | 认证结果 |
| `PublishCallback` | 发布回调 | 发布结果 |

### 6.2 回调注册流程

```
应用层
    │
    ▼
N-API: DeviceManagerNapi::On()
    │
    ▼
Inner SDK: DeviceManager::RegisterCallback()
    │
    ▼
Service: DeviceManagerServiceImpl::Subscribe()
    │
    ▼
DSoftBus/HiChain: 注册回调
    │
    ▼
回调触发 ──► Service ──► IPC ──► N-API ──► JS
```

## 7. 依赖关系图

### 7.1 模块依赖

```
                    ┌─────────────┐
                    │  N-API 层   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ Inner SDK   │
                    │ (ipc client)│
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
   ┌─────┴─────┐    ┌─────┴─────┐    ┌──────┴──────┐
   │   IPC    │    │  Service  │    │  DSoftBus  │
   │ Skeleton │◄───│   Core    │───►│  Connector  │
   └───────────┘    └─────┬─────┘    └─────────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
        ┌─────┴─────┐ ┌────┴────┐ ┌─────┴─────┐
        │ HiChain  │ │  Auth   │ │ Discovery │
        │Connector │ │ Manager │ │ Manager   │
        └───────────┘ └─────────┘ └───────────┘
```

### 7.2 依赖方向

**禁止循环依赖**：
- `interfaces/` → `services/` → `interfaces/`
- `services/` → `utils/` → `services/`

**允许依赖**：
- `interfaces/` → `services/`（单向）
- `services/` → `utils/`（单向）

## 8. 头文件包含规范

### 8.1 头文件层级

| 层级 | 头文件 | 示例 |
|-----|-------|------|
| **1** | 系统头文件 | `<stdio.h>`, `<string>` |
| **2** | OpenHarmony 框架头文件 | `ipc_skeleton.h`, `want.h` |
| **3** | DeviceManager 公共头文件 | `dm_*.h` |
| **4** | 本模块私有头文件 | `"dm_auth_manager.h"` |

### 8.2 包含示例

```cpp
// ✅ 正确示例
#include <string>
#include <mutex>

#include "ipc_skeleton.h"
#include "dm_device_info.h"
#include "dm_auth_manager.h"

#include "dm_auth_manager_impl.h"  // 本模块私有
```

```cpp
// ❌ 错误示例
#include "services/authentication/dm_auth_manager.h"  // 跨模块引用
#include "../../utils/log.h"  // 相对路径
```
