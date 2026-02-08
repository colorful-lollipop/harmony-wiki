# 内部 API 与模块接口

## 目的

本文档说明 DSLM 模块的内部 API，包括模块接口、依赖方向、稳定性说明。

## 适用范围

- ✅ 模块间接口说明
- ✅ 依赖方向（避免环依赖）
- ✅ 稳定/不稳定接口标注

## 核心模块接口

### 1. baselib/msglib - 消息库

**对外接口**：`baselib/msglib/include/messenger.h`

**接口类型**：C 函数指针和结构体

**核心接口**：
```c
typedef int32_t (*DeviceMessageReceiver)(const DeviceIdentify *devId, const uint8_t *msg, uint32_t msgLen);
typedef int32_t (*DeviceStatusReceiver)(const DeviceIdentify *devId, uint32_t status, int32_t level);
typedef int32_t (*MessageSendResultNotifier)(const DeviceIdentify *devId, uint64_t transNo, uint32_t result);

Messenger *CreateMessenger(const MessengerConfig *config);
void DestroyMessenger(Messenger *messenger);
bool IsMessengerReady(const Messenger *messenger);
void SendMsgTo(const Messenger *messenger, uint64_t transNo, const DeviceIdentify *devId, const uint8_t *msg, uint32_t msgLen);
bool GetDeviceOnlineStatus(const Messenger *messenger, const DeviceIdentify *devId, int32_t *level);
bool GetSelfDeviceIdentify(const Messenger *messenger, DeviceIdentify *devId, int32_t *level);
void ForEachDeviceProcess(const Messenger *messenger, const DeviceProcessor processor, void *para);
```

**调用者**：
- services/msg/ - DSLM 消息封装
- services/dslm/ - 核心逻辑

**稳定性**：✅ **稳定**（公共基础库接口）

---

### 2. services/sa - SA 服务

**对外接口**：`services/include/dslm_ipc_process.h`

**接口类型**：C 函数（Lite/Standard 共享）

**核心接口**：
```c
int32_t DslmProcessGetDeviceSecurityLevel(const DeviceIdentify *identify, const RequestOption *option,
    uint32_t cookie, const sptr<IRemoteObject> &callback);
int32_t DslmGetRequestFromParcel(MessageParcel &data, DeviceIdentify &identify, RequestOption &option,
    sptr<IRemoteObject> &object, uint32_t &cookie);
int32_t DslmSetResponseToParcel(MessageParcel &reply, uint32_t status);
```

**调用者**：
- interfaces/inner_api/ - SDK 的 DeviceSecurityLevelLoader

**被调用者**：
- services/dslm/ - 核心逻辑（DslmIpcProcess）

**稳定性**：✅ **稳定**（服务内部接口）

---

### 3. services/dslm - 核心逻辑

**对外接口**：`services/include/dslm_additions.h`

**接口类型**：C 函数指针

**核心接口**：
```c
typedef int32_t (*GetDeviceCred)(const DeviceIdentify *device, DslmCredBuff *credBuff);
typedef int32_t (*VerifyDslmCred)(const DeviceIdentify *device, uint64_t challenge,
    const DslmCredBuff *credBuff, DslmCredInfo *credInfo);
```

**调用者**：
- services/sa/ - SA 服务
- services/dslm/* - 核心逻辑内部调用

**被调用者**：
- oem_property/ - OEM 适配层

**稳定性**：⚠️ **内部接口**（模块边界接口）

---

### 4. oem_property - OEM 适配层

**对外接口**：`oem_property/include/dslm_credential.h`

**接口类型**：C 函数指针

**核心接口**：
```c
typedef int32_t (*GetDeviceCred)(const DeviceIdentify *device, DslmCredBuff *credBuff);
typedef int32_t (*VerifyDslmCred)(const DeviceIdentify *device, uint64_t challenge,
    const DslmCredBuff *credBuff, DslmCredInfo *credInfo);
```

**实现者**：
- oem_property/ohos/ - OHOS 实现
- oem_property/common/ - 默认实现

**稳定性**：✅ **可替换**（OEM 可定制）

---

### 5. baselib/utils - 工具库

**对外接口**：多个工具头文件（`baselib/utils/include/`）

**接口类型**：C 函数和宏定义

**稳定性**：✅ **稳定**（公共基础库）

---

## 依赖方向

### 依赖关系图

```
应用层
    ↓
interfaces/inner_api (SDK)
    ↓ 依赖
services/sa (SA)
    ↓ 依赖
services/dslm (Core)
    ↓ 依赖
oem_property (OEM Adapter)
    ↓ 依赖
baselib (基础库)
```

### 无环依赖验证

**依赖链**：
1. interfaces → services → dslm → oem_property → baselib（单向）
2. services/sa → services/dslm → services/msg → baselib/msglib → baselib/utils（单向）
3. 无循环依赖

**结论**：✅ **无环依赖**

## 接口稳定性标注

### 稳定接口（公共 API）

| 接口 | 稳定性 | 证据 |
|------|--------|------|
| Messenger 接口 | ✅ 稳定 | 公共基础库，多模块共享 |
| 工具函数接口 | ✅ 稳定 | 公共基础库，多模块共享 |
| IDeviceSecurityLevel (IPC) | ✅ 稳定 | OpenHarmony 标准接口 |
| 对外 C API | ✅ 稳定 | `device_security_info.h` 对外暴露 |

### 内部接口（模块边界）

| 接口 | 稳定性 | 证据 |
|------|--------|------|
| DslmIpcProcess | ⚠️ 内部 | SA 与 Core 的边界 |
| DslmCoreProcess | ⚠️ 内部 | Core 内部接口 |
| GetDeviceCred/VerifyDslmCred | ⚠️ 可替换 | OEM 可替换实现 |

### 可替换点

| 可替换点 | 说明 | 实现位置 |
|---------|------|----------|
| **设备凭据获取与验证** | OEM 可自定义安全等级评估逻辑 | `oem_property/ohos/` |
| **插件加载** | 支持动态加载插件扩展能力 | `services/sa/standard/dslm_service.cpp:182-189` |

## 关键结论

1. **依赖清晰**：无环依赖，单向调用链
2. **稳定接口**：baselib 层、Messenger 接口、IPC 接口、对外 C API
3. **内部接口**：DslmIpcProcess、DslmCoreProcess、OEM 接口
4. **可替换点**：OEM 凭据验证、插件加载机制

## 相关跳转

- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
- [03_Architecture.md](./03_Architecture.md) - 系统架构
- [06_GN_Targets.md](./06_GN_Targets.md) - GN Targets
