# 内部 Inner API

## 1. 内部 SDK 模块概览

dmsfwk 项目提供三个主要的内部 SDK 模块，供 C++ 开发者使用。

## 2. common 模块

### 2.1 DistributedAbilityManagerClient

**文件位置**：`interfaces/innerkits/common/include/distributed_ability_manager_client.h`

**命名空间**：`OHOS::DistributedSchedule`

**单例模式**：使用 `DECLARE_SINGLE_INSTANCE` 宏

**公开方法**：

| 方法 | 用途 |
|------|------|
| `Register` | 注册接续 |
| `Unregister` | 注销接续 |
| `RegisterDeviceSelectionCallback` | 注册设备选择回调 |
| `UnregisterDeviceSelectionCallback` | 注销设备选择回调 |
| `UpdateConnectStatus` | 更新连接状态 |
| `StartDeviceManager` | 启动设备管理器 |

**私有方法**：

| 方法 | 用途 |
|------|------|
| `GetContinuationMgrService` | 获取接续管理服务 |

**依赖**：
- `device_selection_notifier_stub.h`
- `idistributed_ability_manager.h`
- `iremote_broker.h`

### 2.2 常量定义

**文件位置**：`interfaces/innerkits/common/include/dms_constant.h`

**用途**：定义分布式调度的常量值

### 2.3 类型定义

**文件位置**：`interfaces/innerkits/common/include/distributed_sched_types.h`

**用途**：定义分布式调度的公共类型

## 3. continuation_manager 模块

### 3.1 接续结果类型

**文件位置**：`interfaces/innerkits/continuation_manager/include/continuation_result.h`

**命名空间**：`OHOS::DistributedSchedule`

### 3.2 接续模式枚举

**文件位置**：`interfaces/innerkits/continuation_manager/include/continuation_mode.h`

```cpp
enum class ContinuationMode {
    COLLABORATION_SINGLE = 0,   // 单设备协作
    COLLABORATION_MUTIPLE = 1,  // 多设备协作
};
```

### 3.3 设备选择通知器

**文件位置**：
- `interfaces/innerkits/continuation_manager/include/device_selection_notifier_stub.h`
- `interfaces/innerkits/continuation_manager/include/idevice_selection_notifier.h`

**接口**：
- `IDeviceSelectionNotifier` - 设备选择通知器接口
- `DeviceSelectionNotifierStub` - 设备选择通知器存根

### 3.4 接续额外参数

**文件位置**：`interfaces/innerkits/continuation_manager/include/continuation_extra_params.h`

**用途**：定义接续的额外参数配置

## 4. distributed_event 模块

### 4.1 DMS 客户端

**文件位置**：`interfaces/innerkits/distributed_event/include/dms_client.h`

**命名空间**：`OHOS::DistributedSchedule`

### 4.2 DMS 事件处理器

**文件位置**：`interfaces/innerkits/distributed_event/include/dms_handler.h`

### 4.3 DMS 监听器存根

**文件位置**：`interfaces/innerkits/distributed_event/include/dms_listener_stub.h`

### 4.4 DMS SA 客户端

**文件位置**：`interfaces/innerkits/distributed_event/include/dms_sa_client.h`

### 4.5 辅助类

| 文件 | 用途 |
|------|------|
| `distributed_parcel_helper.h` | Parcel 辅助类 |

## 5. 服务层 Inner API

### 5.1 dtbabilitymgr 服务

**主类**：`DistributedAbilityManagerService`

**文件位置**：`services/dtbabilitymgr/include/distributed_ability_manager_service.h`

**功能**：
- 设备选择管理
- 接续管理
- 连接回调管理

### 5.2 dtbcollabmgr 服务

**主类**：
- `AbilityConnectionManager` - 能力连接管理器
- `ChannelManager` - 通道管理器
- `AVSenderEngine` - 音视频发送引擎
- `AVReceiverEngine` - 音视频接收引擎

## 6. 框架层 Inner API

### 6.1 DistributedExtension

**文件位置**：`frameworks/native/distributed_extension/include/distributed_extension.h`

**命名空间**：`OHOS::DistributedSchedule`

**基类**：提供分布式扩展能力

### 6.2 DistributedExtensionContext

**文件位置**：`frameworks/native/distributed_extension/include/distributed_extension_context.h`

**功能**：提供扩展上下文环境

### 6.3 DistributedExtensionService

**文件位置**：`frameworks/native/distributed_extension/include/distributed_extension_service.h`

**功能**：扩展服务生命周期管理

## 7. 依赖关系

```
DistributedAbilityManagerClient
    |
    v
IDistributedAbilityManager (IPC Interface)
    |
    v
DistributedAbilityManagerService
    |
    +---> DeviceSelectionNotifier
    +---> ContinuationManager
    +---> ConnectionCallbackManager

AbilityConnectionManager (dtbcollabmgr)
    |
    +---> ChannelManager
    +---> AVSenderEngine
    +---> AVReceiverEngine

DistributedExtension
    |
    +---> DistributedExtensionContext
    +---> DistributedExtensionService
```

## 8. 稳定性标注

### 8.1 稳定接口

| 接口 | 稳定性 | 证据 |
|------|--------|------|
| `DistributedAbilityManagerClient` | 稳定 | 单例模式，已在 innerkits 发布 |
| `DistributedSchedService` | 稳定 | 系统能力实现 |
| `IDistributedAbilityManager` | 稳定 | IPC 接口定义 |

### 8.2 演进中接口

| 接口 | 稳定性 | 证据 |
|------|--------|------|
| `AbilityConnectionManager` | 演进中 | N-API 正在完善 |
| `AVStream APIs` | 演进中 | 新增功能 |
| `ContinuationStateManager` | 演进中 | 新版 API |

### 8.3 内部使用接口

| 接口 | 用途 | 限制 |
|------|------|------|
| `DistributedSchedProxy` | IPC 代理 | 仅限 services 层 |
| `DistributedSchedStub` | IPC 存根 | 仅限 services 层 |

## 9. 可替换点

### 9.1 传输适配层

| 替换点 | 当前实现 | 可替换为 |
|--------|----------|----------|
| 软总线适配 | `SoftbusAdapter` | 其他传输协议 |
| 通道管理 | `ChannelManager` | 自定义通道实现 |

### 9.2 编解码适配

| 替换点 | 当前实现 | 可替换为 |
|--------|----------|----------|
| 视频编码 | `SurfaceEncoderFilter` | 其他编码器 |
| 视频解码 | `SurfaceDecoderFilter` | 其他解码器 |

### 9.3 设备发现

| 替换点 | 当前实现 | 可替换为 |
|--------|----------|----------|
| 设备选择 | `DeviceSelectionNotifier` | 自定义发现机制 |

## 10. 错误码映射

### 10.1 Inner API 错误码

| 错误码 | 说明 | 对应 N-API |
|--------|------|------------|
| 0 | 成功 | - |
| -1 | 失败 | 16600001 |
| -2 | 参数错误 | 401 |
| -3 | 权限不足 | 201 |

### 10.2 错误传播路径

```
Native Error Code  --->  N-API Error Code
     |                      |
     v                      v
  SERVICE_LAYER      -->  INTERFACES_LAYER
     |                      |
     v                      v
  IPC Layer          -->  N-API Error Mapping
```
