# 目录结构与代码地图

> 最后更新：2026-02-07
> 版本：v3.0.0

## 3.1 顶层目录结构

### 完整目录树

```
base/security/device_security_level/
├── baselib/                    # 基础库
│   ├── msglib/                # 消息通信库
│   │   ├── include/           # 头文件
│   │   ├── src/              # 源代码
│   │   │   ├── common/       # 公共实现
│   │   │   ├── standard/     # Standard 平台实现
│   │   │   ├── lite/         # Lite 平台实现
│   │   │   └── utils/        # 工具函数
│   │   └── BUILD.gn
│   ├── utils/                 # 工具库
│   │   ├── include/          # 头文件
│   │   ├── src/              # 源代码
│   │   └── BUILD.gn
│   └── BUILD.gn
├── common/                    # 公共头文件
│   ├── include/              # 公共头文件
│   └── BUILD.gn
├── interfaces/                # 对外接口
│   └── inner_api/            # Inner API
│       ├── include/          # 公共头文件
│       ├── src/              # 源代码
│       │   ├── standard/     # Standard 平台实现
│       │   ├── lite/         # Lite 平台实现
│       │   │   ├── include/ # Lite 头文件
│       │   │   ├── small/   # Small 平台
│       │   │   └── mini/    # Mini 平台
│       │   └── BUILD.gn
│       └── BUILD.gn
├── oem_property/              # OEM 适配层
│   ├── include/              # 头文件
│   ├── common/               # 公共实现
│   ├── ohos/                 # OpenHarmony 适配
│   │   ├── common/          # 公共适配
│   │   ├── standard/         # Standard 平台适配
│   │   │   ├── impl/        # 实现
│   │   │   └── *.cfg        # 配置文件
│   │   ├── lite/             # Lite 平台适配
│   │   │   ├── impl/        # 实现
│   │   │   └── *.cfg        # 配置文件
│   │   └── BUILD.gn
│   └── BUILD.gn
├── profile/                   # 组件配置
│   ├── dslm_service.cfg      # SA 服务配置
│   ├── dslm_service.rc       # 资源文件
│   ├── dslm_service.xml      # XML 配置
│   └── BUILD.gn
├── services/                  # 服务框架代码
│   ├── common/               # 公共服务
│   ├── dfx/                  # DFX 诊断
│   ├── dslm/                 # DSLM 核心逻辑
│   │   ├── *.c              # 核心源文件
│   │   ├── *.h              # 核心头文件
│   │   └── BUILD.gn
│   ├── msg/                  # 消息处理
│   │   ├── *.c              # 消息源文件
│   │   └── BUILD.gn
│   ├── sa/                   # SA 服务
│   │   ├── common/          # 公共 SA 代码
│   │   ├── standard/         # Standard SA
│   │   │   ├── *.cpp        # Standard SA 源文件
│   │   │   └── *.h          # Standard SA 头文件
│   │   ├── lite/            # Lite SA
│   │   │   ├── small/       # Small SA
│   │   │   └── mini/        # Mini SA
│   │   └── BUILD.gn
│   └── BUILD.gn
├── test/                     # 测试代码（本文档不涉及）
├── figures/                   # 文档图片
├── bundle.json               # 组件配置
├── hisysevent.yaml           # HiEvent 配置
├── README.md                 # 英文文档
├── README_ZH.md             # 中文文档
└── LICENSE                   # Apache 2.0 许可证
```

### 目录职责总览

| 目录 | 职责 | 关键证据 |
|------|------|----------|
| **baselib/** | 基础能力库（消息、定时器、状态机等） | `baselib/BUILD.gn` 定义 `messenger_static` 和 `utils_static` |
| **common/** | 公共头文件（SA ID、IPC 接口码、错误码） | `common/include/idevice_security_level.h:30` |
| **interfaces/** | 对外 SDK（C API、IPC Proxy） | `interfaces/inner_api/include/device_security_info.h` |
| **oem_property/** | OEM 适配层（凭证生成/验证） | `oem_property/include/dslm_credential.h` |
| **profile/** | 组件配置（SA 配置、权限） | `profile/dslm_service.cfg` |
| **services/** | 服务实现（SA、核心逻辑、消息） | `services/sa/standard/dslm_service.cpp` |

---

## 3.2 核心文件定位

### SDK 层关键文件

| 文件路径 | 职责 | 代码证据 |
|----------|------|----------|
| `interfaces/inner_api/include/device_security_info.h` | 公共 API 声明 | `interfaces/inner_api/include/device_security_info.h:42-68` |
| `interfaces/inner_api/include/device_security_defines.h` | 数据结构和错误码 | `interfaces/inner_api/include/device_security_defines.h:25-92` |
| `interfaces/inner_api/src/standard/device_security_level_proxy.h` | IPC Proxy | `interfaces/inner_api/src/standard/device_security_level_proxy.h` |
| `interfaces/inner_api/src/standard/device_security_level_callback_helper.h` | 回调管理 | `device_security_level_callback_helper.h` |

### SA 服务层关键文件

| 文件路径 | 职责 | 代码证据 |
|----------|------|----------|
| `services/sa/standard/dslm_service.cpp` | SA 主服务 | `services/sa/standard/dslm_service.cpp:38-144` |
| `services/sa/standard/dslm_service.h` | SA 服务头文件 | `services/sa/standard/dslm_service.h` |
| `services/sa/standard/dslm_ipc_process.cpp` | IPC 请求处理 | `services/sa/standard/dslm_ipc_process.cpp:62-117` |
| `services/sa/standard/dslm_callback_proxy.cpp` | 回调代理 | `services/sa/standard/dslm_callback_proxy.cpp:33-80` |
| `services/sa/common/dslm_rpc_process.c` | 服务初始化 | `services/sa/common/dslm_rpc_process.c:80-116` |

### 核心逻辑层关键文件

| 文件路径 | 职责 | 代码证据 |
|----------|------|----------|
| `services/dslm/dslm_core_process.c` | 核心处理入口 | `services/dslm/dslm_core_process.c:52-139` |
| `services/dslm/dslm_core_defines.h` | 核心数据结构 | `services/dslm/dslm_core_defines.h:38-61` |
| `services/dslm/dslm_fsm_process.c` | 状态机实现 | `services/dslm/dslm_fsm_process.c` |
| `services/dslm/dslm_device_list.c` | 设备列表管理 | `services/dslm/dslm_device_list.c` |
| `services/dslm/dslm_msg_utils.c` | 消息序列化 | `services/dslm/dslm_msg_utils.c` |

### OEM 适配层关键文件

| 文件路径 | 职责 | 代码证据 |
|----------|------|----------|
| `oem_property/include/dslm_credential.h` | 凭证操作接口 | `oem_property/include/dslm_credential.h:18-32` |
| `oem_property/include/dslm_cred.h` | 凭证数据结构 | `oem_property/include/dslm_cred.h` |
| `oem_property/common/dslm_credential.c` | 凭证函数注册 | `oem_property/common/dslm_credential.c` |
| `oem_property/common/dslm_credential_utils.c` | 凭证解析验证 | `oem_property/common/dslm_credential_utils.c:522-577` |
| `oem_property/ohos/common/dslm_ohos_verify.c` | OHOS 凭证验证 | `oem_property/ohos/common/dslm_ohos_verify.c` |
| `oem_property/ohos/common/dslm_ohos_request.c` | OHOS 凭证请求 | `oem_property/ohos/common/dslm_ohos_request.c` |
| `oem_property/ohos/common/hks_adapter.c` | HUKS 适配 | `oem_property/ohos/common/hks_adapter.c` |

### 公共头文件

| 文件路径 | 职责 | 代码证据 |
|----------|------|----------|
| `common/include/idevice_security_level.h` | SA ID 和接口定义 | `common/include/idevice_security_level.h:30` |
| `common/include/dslm_service_ipc_interface_code.h` | IPC 命令码 | `common/include/dslm_service_ipc_interface_code.h:26` |

---

## 3.3 功能映射表

### SDK API 实现映射

| 功能 | C API 函数 | IPC Proxy | SA IPC 处理 | 核心处理 |
|------|------------|-----------|-------------|----------|
| 同步查询 | `RequestDeviceSecurityInfo()` | `device_security_level_proxy.cpp` | `dslm_ipc_process.cpp` | `dslm_core_process.c` |
| 异步查询 | `RequestDeviceSecurityInfoAsync()` | `device_security_level_proxy.cpp` | `dslm_ipc_process.cpp` | `dslm_core_process.c` | 
| 释放资源 | `FreeDeviceSecurityInfo()` | - | - | - |
| 提取等级 | `GetDeviceSecurityLevelValue()` | - | - | - |

### 凭证功能映射

| 功能 | OHOS 标准版 | OHOS 轻量版 |
|------|-------------|-------------|
| 初始化 | `dslm_ohos_init.c (standard)` | `dslm_ohos_init.c (lite)` |
| 凭证请求 | `dslm_ohos_request.c` | `dslm_ohos_request.c` |
| 凭证验证 | `dslm_ohos_verify.c` | `dslm_ohos_verify.c` |
| 签名验证 | `dslm_credential_utils.c` | `dslm_credential_utils.c` |
| HUKS 集成 | `hks_adapter.c` | `hks_adapter.c` |

### 平台适配映射

| 组件 | Standard | Small | Mini |
|------|----------|-------|------|
| SA 服务 | `dslm_service.cpp` | `dslm_service.c` | `dslm_service_feature.c` |
| IPC 处理 | `dslm_ipc_process.cpp` | `dslm_ipc_process.c` | - |
| SDK | `dslm_sdk` | `dslm_sdk_small` | `dslm_sdk_mini` |
| 凭证 | 标准凭证 | Small 凭证 | Mini 凭证 |

---

## 3.4 代码导航图

### 快速定位指南

#### 查询设备安全等级

```
问题：如何查询设备的系统安全等级？

步骤：
1. SDK 入口：interfaces/inner_api/include/device_security_info.h
2. API 函数：RequestDeviceSecurityInfo() 或 RequestDeviceSecurityInfoAsync()
3. 数据结构：DeviceIdentify, RequestOption
4. 错误码：device_security_defines.h 错误码枚举

核心文件：
- api 接口定义 → device_security_info.h
- 参数数据结构 → device_security_defines.h
- 异步回调 → device_security_level_callback_helper.h
```

#### 凭证生成和验证

```
问题：如何验证远端设备的凭证？

步骤：
1. 凭证验证入口：oem_property/include/dslm_credential.h
2. 验证函数：VerifyDslmCredential()
3. OHOS 实现：dslm_ohos_verify.c
4. 底层验证：dslm_credential_utils.c (ECDSA 签名)

核心文件：
- 凭证接口定义 → dslm_credential.h
- OHOS 凭证验证 → dslm_ohos_verify.c
- JWS 签名验证 → dslm_credential_utils.c:522-577
- 证书链验证 → external_interface_adapter.c:107-161
```

#### SA 服务生命周期

```
问题：DslmService SA 如何启动和停止？

步骤：
1. SA 注册：services/sa/standard/dslm_service.cpp:38
2. 启动入口：OnStart() → dslm_service.cpp:83
3. 初始化流程：InitService() → dslm_rpc_process.c:80
4. 停止流程：OnStop() → dslm_service.cpp:98

核心文件：
- SA 注册宏 → dslm_service.cpp:38 REGISTER_SYSTEM_ABILITY_BY_ID
- 启动逻辑 → dslm_service.cpp:83 OnStart()
- 初始化逻辑 → dslm_rpc_process.c:80 InitService()
```

#### 状态机处理

```
问题：设备安全等级查询的状态机如何工作？

步骤：
1. 状态定义：dslm_fsm_process.h:28-33 (STATE_INIT, STATE_WAITING_CRED_RSP, etc.)
2. 事件定义：dslm_fsm_process.h:36-46 (EVENT_SDK_GET, EVENT_CRED_RSP, etc.)
3. 状态调度：dslm_fsm_process.c ScheduleDslmStateMachine()
4. 状态处理：dslm_fsm_process.c Process* 函数

核心文件：
- 状态定义 → dslm_fsm_process.h:28-33
- 事件定义 → dslm_fsm_process.h:36-46
- 主入口 → dslm_core_process.c:139 OnRequestDeviceSecLevelInfo()
```

---

## 3.5 关键数据结构导航

### DslmDeviceInfo

```c
// services/dslm/dslm_core_defines.h:38-61
typedef struct DslmDeviceInfo {
    ListNode linkNode;              // 设备链表节点
    StateMachine machine;            // 状态机
    DeviceIdentify identity;        // 设备标识
    uint32_t version;               // 协议版本
    uint32_t onlineStatus;          // 在线状态
    uint64_t nonce;                 // 挑战值
    uint64_t nonceTimeStamp;        // 挑战时间戳
    uint64_t lastOnlineTime;        // 最后上线时间
    uint64_t lastOfflineTime;       // 最后下线时间
    uint64_t lastRequestTime;       // 最后请求时间
    uint64_t lastResponseTime;      // 最后响应时间
    uint64_t lastVerifyTime;        // 最后验证时间
    uint64_t transNum;              // 事务号
    TimerHandle timeHandle;         // 定时器句柄
    uint32_t queryTimes;            // 查询次数
    uint32_t result;                // 结果
    DslmCredInfo credInfo;          // 凭证信息
    uint32_t notifyListSize;       // 回调列表大小
    ListHead notifyList;           // 回调链表
    uint32_t historyListSize;      // 历史列表大小
    ListHead historyList;          // 历史链表
    uint32_t osType;               // 操作系统类型
} DslmDeviceInfo;
```

### DslmCredInfo

```c
// oem_property/include/dslm_cred.h
typedef struct DslmCredInfo {
    uint32_t magicNum;              // 魔数
    uint32_t credType;              // 凭证类型
    uint32_t securityLevel;         // 安全等级
    uint8_t udid[DEVICE_ID_MAX_LEN]; // UDID
    char manufacture[64];           // 厂商
    char brand[64];                 // 品牌
    char model[64];                  // 型号
    // ... 更多字段
} DslmCredInfo;
```

### DeviceIdentify

```c
// interfaces/inner_api/include/device_security_defines.h:27-30
#define DEVICE_ID_MAX_LEN 64

typedef struct DeviceIdentify {
    uint32_t length;
    uint8_t identity[DEVICE_ID_MAX_LEN];
} DeviceIdentify;
```

---

## 3.6 构建配置导航

### 关键 GN Targets

| Target | 类型 | 输出 | 路径 |
|--------|------|------|------|
| **dslm_sdk** | ohos_shared_library | libdslm_sdk.z.so | `interfaces/inner_api/BUILD.gn` |
| **dslm_service** | ohos_shared_library | libdslm_service.z.so | `services/sa/BUILD.gn` |
| **service_dslm_obj** | ohos_source_set | - | `services/dslm/BUILD.gn` |
| **messenger_static** | ohos_static_library | libmessenger.a | `baselib/msglib/BUILD.gn` |
| **utils_static** | ohos_static_library | libutils.a | `baselib/utils/BUILD.gn` |

### Feature Flags

| Feature | 定义位置 | 用途 |
|---------|----------|------|
| `device_security_level_feature_cred_level` | bundle.json | 信任等级特性 |
| `device_security_level_feature_plugin_path` | bundle.json | 插件路径特性 |
| `device_security_level_feature_secondary_session_name` | bundle.json | 次级会话名称特性 |

---

## 3.7 常见问题定位

### SDK 相关问题

| 问题 | 定位文件 |
|------|----------|
| API 函数未找到 | `interfaces/inner_api/include/device_security_info.h` |
| 错误码定义 | `interfaces/inner_api/include/device_security_defines.h` |
| 异步回调异常 | `interfaces/inner_api/src/standard/device_security_level_callback_helper.h` |
| IPC 调用失败 | `interfaces/inner_api/src/standard/device_security_level_proxy.h` |

### SA 服务问题

| 问题 | 定位文件 |
|------|----------|
| SA 未注册 | `services/sa/standard/dslm_service.cpp:38` |
| IPC 请求处理异常 | `services/sa/standard/dslm_ipc_process.cpp` |
| 服务初始化失败 | `services/sa/common/dslm_rpc_process.c:80` |
| 回调分发异常 | `services/sa/standard/dslm_callback_proxy.cpp` |

### 凭证问题

| 问题 | 定位文件 |
|------|----------|
| 凭证验证失败 | `oem_property/ohos/common/dslm_ohos_verify.c` |
| 签名验证异常 | `oem_property/common/dslm_credential_utils.c:522-577` |
| 证书链验证问题 | `oem_property/ohos/common/external_interface_adapter.c:107-161` |
| HUKS 操作失败 | `oem_property/ohos/common/hks_adapter.c` |

### 状态机问题

| 问题 | 定位文件 |
|------|----------|
| 状态转移异常 | `services/dslm/dslm_fsm_process.c` |
| 设备状态管理 | `services/dslm/dslm_device_list.c` |
| 超时处理 | `services/dslm/dslm_core_process.c` |

---

## 下一章

- [04_Interface.md](./04_Interface.md) - 对外接口文档
- [05_AttackSurface.md](./05_AttackSurface.md) - 攻击面分析
- [06_SecurityReview.md](./06_SecurityReview.md) - 安全风险评估
