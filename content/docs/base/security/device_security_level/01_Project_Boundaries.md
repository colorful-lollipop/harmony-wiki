# 项目定位与边界

## 目的

本文档说明 DSLM 模块的定位、边界、核心能力、运行环境和关键概念。

## 适用范围

- ✅ DSLM 的职责边界
- ✅ 核心能力详细说明
- ✅ 运行环境与依赖
- ✅ DSLM 不负责的内容

## 项目定位

### 在分布式系统中的角色

```
┌─────────────────────────────────────────────────────────────────────┐
│                         OpenHarmony 分布式系统                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                   应用层（Apps）                               │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │               子系统层（Subsystems）                          │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  │   │
│  │  │Data     │  │Device   │  │ Other    │  │   │
│  │  │Transfer  │  │Auth     │  │Modules   │  │   │
│  │  └─────────┘  └─────────┘  └─────────┘  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │              系统服务层（System Services）                  │   │
│  │  ┌─────────────────────────────────────────────────────────┐   │
│  │  │  DSLM (SA 3511) - 设备安全等级管理   │   │
│  │  └─────────────────────────────────────────────────────────┘   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### 核心定位

**DSLM = 设备安全等级的"评估者 + 缓存者"**

| 角色 | 说明 | 证据 |
|------|------|------|
| **评估者** | 通过设备凭据（证书链+签名）评估设备安全等级 | `oem_property/ohos/common/dslm_ohos_verify.c` |
| **缓存者** | 缓存设备安全等级信息，减少重复查询 | `services/dslm/dslm_device_list.c` |
| **查询接口** | 提供同步/异步接口供其他模块查询 | `interfaces/inner_api/include/device_security_info.h` |
| **系统服务** | 作为 SA (System Ability 3511) 运行 | `services/sa/standard/dslm_service.cpp` |

## 核心能力

### 1. 设备安全等级评估

**证据**：`oem_property/ohos/common/dslm_ohos_verify.c:235-278`

**评估流程**：
1. 接收目标设备的安全等级凭据（Credential）
2. 验证证书链完整性（Root → Intermediate → Leaf）
3. 验证每个证书的 ECDSA 签名
4. 验证 Challenge-Nonce 匹配（防止重放）
5. 提取安全等级（SL1-SL5）

**支持的凭据类型**：
- `CRED_TYPE_STANDARD` - 标准 OHOS 设备
- `CRED_TYPE_SMALL` - Small 系统
- `CRED_TYPE_MINI` - Mini 系统

### 2. 安全等级查询接口

**证据**：`interfaces/inner_api/include/device_security_info.h:42-68`

**API 列表**：
- `RequestDeviceSecurityInfo()` - 同步查询（阻塞等待）
- `RequestDeviceSecurityInfoAsync()` - 异步查询（回调返回）
- `FreeDeviceSecurityInfo()` - 释放查询结果
- `GetDeviceSecurityLevelValue()` - 提取等级值

**参数说明**：
- `DeviceIdentify` - 设备标识符（UDID，最大 64 字节）
- `RequestOption` - 请求选项（challenge、timeout、extra）
- `DeviceSecurityInfo` - 安全等级信息（包含等级值）

### 3. 设备列表管理

**证据**：`services/dslm/dslm_device_list.c`

**功能**：
- 管理所有已知的设备信息
- 记录设备的在线/离线状态
- 记录设备的查询历史和响应时间
- 支持设备信息的增删查

### 4. 设备间通信

**证据**：`baselib/msglib/include/messenger.h`

**功能**：
- 封装 DSoftBus 和设备管理器接口
- 提供统一的跨设备消息发送/接收
- 支持设备在线状态查询
- 支持设备遍历处理

## 运行环境

### 系统形态支持

| 形态 | 实现 | 特点 |
|------|------|------|
| **Standard** | C++ | 完整 SA 框架、Binder IPC、多线程 |
| **Small** | C | Lite SAMGR、Binder/Socket IPC、资源受限 |
| **Mini** | C | 静态库、无 IPC（内部调用） |

### 运行时环境

- **进程**：`dslm_service`（SA 进程）
- **UID/GID**：3046/3046（证据：`profile/dslm_service.cfg`）
- **SELinux Context**：`u:r:dslm_service:s0`
- **APL**：`system_basic`

### 资源限制

- **ROM**：200KB
- **RAM**：2500KB

## 边界与限制

### DSLM 负责的内容

| 模块 | 职责 | 证据 |
|------|------|------|
| **凭据验证** | 验证设备证书链和签名 | `oem_property/ohos/common/dslm_ohos_verify.c` |
| **等级缓存** | 缓存设备安全等级 | `services/dslm/dslm_device_list.c` |
| **查询接口** | 提供 C API 查询 | `interfaces/inner_api/` |
| **SA 服务** | 作为系统服务运行 | `services/sa/standard/dslm_service.cpp` |
| **消息通信** | 跨设备消息封装 | `baselib/msglib/` |

### DSLM 不负责的内容

| 功能 | 负责模块 | 说明 |
|------|----------|------|
| **设备认证** | device_auth | 设备间的身份认证 |
| **数据分级保护** | dataclassification | 数据风险分级与传输控制 |
| **应用权限检查** | access_token | 应用运行时权限校验 |
| **跨设备数据加解密** | 其他模块 | 数据机密性保护 |
| **JS 接口** | - | DSLM 无 JS/N-API，只提供 C 原生接口 |
| **UI 展示** | 应用层 | 安全等级提示由应用层实现 |

## 依赖关系

### 强依赖（必须）

| 组件 | 用途 | 为什么必需 |
|------|------|----------|
| **device_auth** | 设备认证 | 设备身份认证是凭据验证的基础 |
| **huks (SA 3510)** | 密钥存储 | 存储根密钥，验证签名 |
| **safwk** | SA 框架 | DSLM 作为 SA 需要框架支持 |
| **samgr** | SA 管理 | 注册和发现 SA |
| **ipc** | IPC 通信 | 实现跨进程通信 |

### 可选依赖

| 组件 | 用途 | 条件 |
|------|------|------|
| **openssl** | 加密算法 | 证书链验证需要 |
| **cJSON** | JSON 解析 | 凭据配置解析 |
| **dsoftbus** | 设备通信 | 跨设备消息发送 |

## 关键概念

### DeviceIdentify（设备标识符）

**定义**：`interfaces/inner_api/include/device_security_defines.h:27-30`

```c
#define DEVICE_ID_MAX_LEN 64
typedef struct DeviceIdentify {
    uint32_t length;
    uint8_t identity[DEVICE_ID_MAX_LEN];
} DeviceIdentify;
```

**说明**：设备的唯一标识符（通常是 UDID），最大 64 字节。

### RequestOption（请求选项）

**定义**：`interfaces/inner_api/include/device_security_defines.h:32-36`

```c
typedef struct RequestOption {
    uint64_t challenge;  // 挑战码，防止重放攻击
    uint32_t timeout;    // 超时时间（毫秒）
    uint32_t extra;       // 额外参数
} RequestOption;
```

### DeviceSecurityInfo（安全等级信息）

**定义**：`interfaces/inner_api/include/device_security_info.h:27`

```c
typedef struct DeviceSecurityInfo DeviceSecurityInfo;
```

**说明**：不透明结构体，通过 `GetDeviceSecurityLevelValue()` 提取等级。

## 关键结论

1. **DSLM 是安全等级的"评估者 + 缓存者"**，负责凭据验证和查询接口
2. **边界清晰**：不负责设备认证、数据分级、应用权限、JS 接口
3. **支持三种系统形态**：Standard（C++）、Small（C）、Mini（C）
4. **SA ID 3511** 作为系统服务运行，UID/GID 3046/3046
5. **核心安全机制**：证书链验证 + ECDSA 签名验证

## 相关跳转

- [00_Overview.md](./00_Overview.md) - 项目概览
- [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录组织详解
- [03_Architecture.md](./03_Architecture.md) - 系统架构
- [05_Inner_APIs.md](./05_Inner_APIs.md) - 内部接口详解
