# OAID 目录结构与代码地图

## 顶层目录结构

```
domains/advertising/oaid/
├── bundle.json                    # Bundle 配置文件
├── LICENSE                        # Apache 2.0 许可证
├── README.md                      # 项目 README（英文）
├── README_zh.md                   # 项目 README（中文）
├── oaid.gni                       # GN 构建配置
├── BUILD.gn                       # 根构建文件
│
├── etc/                           # 配置文件
│   └── init/
│       ├── BUILD.gn
│       └── oaidservice.cfg        # SA 启动配置
│
├── interfaces/                    # 接口层
│   ├── innerkits/                 # C++ 内部接口（Inner API）
│   │   ├── include/               # 头文件
│   │   ├── src/                   # 实现文件
│   │   ├── BUILD.gn
│   │   └── liboaidclient.versionscript
│   │
│   └── kits/                      # 应用接口（JS API）
│       └── js/
│           └── napi/
│               └── oaid/          # N-API 实现
│                   ├── include/
│                   ├── src/
│                   └── BUILD.gn
│
├── profile/                       # SA Profile
│   ├── 6101.json                  # OAID SA 配置文件
│   └── BUILD.gn
│
├── services/                      # 服务层
│   ├── BUILD.gn
│   └── oaid_manager/              # OAID 服务管理器
│       ├── include/               # 头文件
│       ├── src/                   # 实现文件
│       └── resource/
│           └── oaid_service_config.json
│
├── utils/                         # 工具类
│   └── native/
│       ├── include/
│       ├── src/
│       └── BUILD.gn
│
└── test/                          # 测试代码（本 Wiki 忽略）
    └── fuzztest/
```

---

## 目录职责说明

| 目录 | 职责 | 重要程度 |
|------|------|---------|
| `interfaces/innerkits/` | C++ 内部接口，供系统应用/服务调用 | ⭐⭐⭐⭐⭐ |
| `interfaces/kits/js/napi/` | JavaScript N-API 桥接实现 | ⭐⭐⭐⭐⭐ |
| `services/oaid_manager/` | 核心业务服务实现 | ⭐⭐⭐⭐⭐ |
| `utils/native/` | 通用工具类（文件操作、日志） | ⭐⭐⭐⭐ |
| `etc/init/` | 服务启动配置文件 | ⭐⭐⭐ |
| `profile/` | SA Profile（SA 管理器配置） | ⭐⭐⭐ |

---

## 核心文件导航图

### 🔴 关键入口文件

```
┌────────────────────────────────────────────────────────────────────┐
│                        N-API 入口                                   │
│  interfaces/kits/js/napi/oaid/src/oaid_init.cpp:39-45              │
│  ├── nm_modname = "identifier.oaid"                                │
│  └── nm_register_func = Init                                       │
└────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────┐
│                        IPC 接口入口                                 │
│  interfaces/innerkits/include/oaid_service_interface.h:26-46       │
│  ├── GetOAID() = 0                                                 │
│  ├── ResetOAID() = 0                                               │
│  └── RegisterObserver() = 0                                        │
└────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────┐
│                        服务主入口                                   │
│  services/oaid_manager/src/oaid_service.cpp:96                     │
│  └── REGISTER_SYSTEM_ABILITY_BY_ID(OAIDService, 6101, true)        │
└────────────────────────────────────────────────────────────────────┘
```

### 📁 按功能分类

#### 1. N-API 实现 (JS → C++)

| 文件 | 路径 | 行数 | 核心功能 |
|------|------|------|---------|
| **oaid.cpp** | `interfaces/kits/js/napi/oaid/src/` | 261 | N-API 函数实现 |
| **oaid_init.cpp** | `interfaces/kits/js/napi/oaid/src/` | 56 | 模块注册 |
| **oaid.h** | `interfaces/kits/js/napi/oaid/include/` | - | 函数声明 |

**关键函数**:
- `GetOAID()` [行192-220] - 异步获取 OAID
- `ResetOAID()` [行222-247] - 重置 OAID
- `ParseParameters()` [行117-130] - 参数解析
- `OAIDInit()` [行249-258] - 模块初始化

#### 2. 客户端 SDK (C++ → IPC)

| 文件 | 路径 | 行数 | 核心功能 |
|------|------|------|---------|
| **oaid_service_client.cpp** | `interfaces/innerkits/src/` | 273 | 客户端单例 |
| **oaid_service_proxy.cpp** | `interfaces/innerkits/src/` | - | IPC 代理 |
| **oaid_service_client.h** | `interfaces/innerkits/include/` | - | 客户端头文件 |
| **oaid_service_proxy.h** | `interfaces/innerkits/include/` | - | 代理头文件 |

**关键函数**:
- `OAIDServiceClient::GetOAID()` [行140-165]
- `OAIDServiceClient::ResetOAID()` [行167-183]
- `OAIDServiceClient::CheckPermission()` [行185-203]
- `OAIDServiceClient::LoadService()` [行98-138]

#### 3. 服务层 (IPC → 业务逻辑)

| 文件 | 路径 | 行数 | 核心功能 |
|------|------|------|---------|
| **oaid_service.cpp** | `services/oaid_manager/src/` | 439 | 主服务实现 |
| **oaid_service_stub.cpp** | `services/oaid_manager/src/` | 356 | IPC Stub |
| **oaid_service.h** | `services/oaid_manager/include/` | 80 | 服务类定义 |
| **oaid_service_stub.h** | `services/oaid_manager/include/` | 65 | Stub 定义 |

**关键函数** (oaid_service.cpp):
- `OAIDService::GetOAID()` [行287-293]
- `OAIDService::ResetOAID()` [行295-312]
- `OAIDService::GainOAID()` [行238-285] - 核心获取逻辑
- `GetUUID()` [行50-93] - UUID v4 生成
- `InitKvStore()` [行327-375]
- `ReadValueFromKvStore()` [行193-214]
- `WriteValueToKvStore()` [行216-236]

**关键函数** (oaid_service_stub.cpp):
- `OnRemoteRequest()` [行162-193] - IPC 请求入口
- `CheckPermission()` [行45-80] - 权限检查
- `CheckSystemApp()` [行82-91] - 系统应用检查
- `OnGetOAID()` [行218-232]
- `OnResetOAID()` [行275-283]
- `LoadAndCheckOaidTrustList()` [行93-141] - 白名单验证

#### 4. 扩展连接层

| 文件 | 路径 | 行数 | 核心功能 |
|------|------|------|---------|
| **connect_ads_stub.cpp** | `services/oaid_manager/src/` | 388 | Ads 服务连接 |
| **connect_ads_stub.h** | `services/oaid_manager/include/` | - | 连接管理器定义 |

**关键函数**:
- `ConnectAdsManager::checkAllowGetOaid()` [行268-311]
- `ConnectAdsManager::notifyKit()` [行339-372]
- `ConnectAdsManager::getWantInfo()` [行216-266] - 配置读取

#### 5. 工具类

| 文件 | 路径 | 行数 | 核心功能 |
|------|------|------|---------|
| **oaid_file_operator.cpp** | `utils/native/src/` | 73 | 文件操作 |
| **oaid_file_operator.h** | `utils/native/include/` | - | 文件操作头文件 |
| **oaid_hilog_wreapper.h** | `utils/native/include/` | - | 日志宏定义 |
| **oaid_common.h** | `utils/native/include/` | - | 错误码定义 |

**关键函数**:
- `OAIDFileOperator::IsFileExsit()` [行28-38]
- `OAIDFileOperator::OpenAndReadFile()` [行40-54]
- `OAIDFileOperator::ClearFile()` [行56-71]

#### 6. 定义与配置

| 文件 | 路径 | 行数 | 核心内容 |
|------|------|------|---------|
| **oaid_service_define.h** | `services/oaid_manager/include/` | 76 | 常量定义 |
| **oaid_service_interface.h** | `interfaces/innerkits/include/` | 49 | 服务接口定义 |
| **oaid_service_ipc_interface_code.h** | `interfaces/innerkits/include/` | - | IPC 命令码 |
| **6101.json** | `profile/` | - | SA Profile |
| **oaidservice.cfg** | `etc/init/` | - | 启动配置 |
| **oaid_service_config.json** | `services/oaid_manager/resource/` | - | 服务配置 |

**关键定义** (oaid_service_define.h):
- `OAID_SYSTME_ID = 6101` [行30]
- `OAID_TRACKING_CONSENT_PERMISSION` [行27]
- `OAID_TRUSTLIST_CONFIG_PATH` [行62-63]
- 数据库配置 [行55-66]

---

## 类继承关系

```
SystemAbility (OpenHarmony 框架)
    │
    ├──► OAIDService ─────────────────┐
    │   [oaid_service.h:32]            │
    │   ├── 继承 SystemAbility         │
    │   └── 继承 OAIDServiceStub       │
    │                                  │
    └──► OAIDServiceStub ─────────────┘
        [oaid_service_stub.h:33]
        └── 继承 IRemoteStub<IOAIDService>

IOAIDService (IRemoteBroker)
    [oaid_service_interface.h:26]
    ├── GetOAID() = 0
    ├── ResetOAID() = 0
    └── RegisterObserver() = 0

OAIDServiceClient (RefBase)
    [oaid_service_client.h:36]
    ├── 单例模式
    ├── 管理 OAIDServiceProxy
    └── 死亡监听 (OAIDSaDeathRecipient)

DelayedSingleton<T>
    │
    ├──► BundleMgrHelper
    │
    └──► OaidObserverManager

ConnectAdsManager (单例)
    [connect_ads_stub.h:94]
    └── 管理 ConnectAdsStub

ConnectAdsStub
    [connect_ads_stub.h:40]
    └── 继承 AbilityConnectionStub
```

---

## 调用链导航

### GetOAID 调用链

```
JS: identifier.getOAID()
    │
    ▼
NAPI: GetOAID() [oaid.cpp:192]
    │
    ├── ParseParameters() [oaid.cpp:117]
    ├── Create AsyncWork
    └── GetOAIDExecuteCallBack()
        │
        ▼
Client: OAIDServiceClient::GetOAID() [oaid_service_client.cpp:140]
    │
    ├── CheckPermission() [oaid_service_client.cpp:185]
    ├── LoadService() [oaid_service_client.cpp:98]
    └── oaidServiceProxy_>GetOAID()
        │
        ▼
Proxy: OAIDServiceProxy::GetOAID()
    │
    └── SendRequest(GET_OAID)
        │
        ▼
Stub: OAIDServiceStub::OnRemoteRequest() [oaid_service_stub.cpp:162]
    │
    ├── GetCallingUid()
    ├── GetBundleNameByUid()
    ├── CheckPermission() [oaid_service_stub.cpp:45]
    ├── Verify InterfaceToken
    └── OnGetOAID() [oaid_service_stub.cpp:218]
        │
        ▼
Service: OAIDService::GetOAID() [oaid_service.cpp:287]
    │
    └── GainOAID() [oaid_service.cpp:238]
        │
        ├── Read update_check.json
        ├── checkAllowGetOaid() ──► ConnectAdsManager
        ├── ReadValueFromKvStore() ──► KVStore
        └── GetUUID() ──► OpenSSL RAND_bytes
```

### ResetOAID 调用链

```
JS: identifier.resetOAID()
    │
    ▼
NAPI: ResetOAID() [oaid.cpp:222]
    │
    ▼
Client: OAIDServiceClient::ResetOAID() [oaid_service_client.cpp:167]
    │
    ▼
Proxy: OAIDServiceProxy::ResetOAID()
    │
    └── SendRequest(RESET_OAID)
        │
        ▼
Stub: OAIDServiceStub::OnRemoteRequest() [oaid_service_stub.cpp:162]
    │
    ├── GetBundleNameByUid()
    ├── LoadAndCheckOaidTrustList() [oaid_service_stub.cpp:93]
    ├── CheckSystemApp() [oaid_service_stub.cpp:82]
    ├── Verify InterfaceToken
    └── OnResetOAID() [oaid_service_stub.cpp:275]
        │
        ▼
Service: OAIDService::ResetOAID() [oaid_service.cpp:295]
    │
    ├── GetUUID() [oaid_service.cpp:50]
    ├── WriteValueToKvStore()
    ├── notifyKit() ──► ConnectAdsManager
    └── OnUpdateOaid() ──► OaidObserverManager
```

---

## 文件定位速查表

### 按功能定位

| 功能 | 文件路径 | 关键函数/类 |
|------|---------|------------|
| **获取 OAID** | `services/oaid_manager/src/oaid_service.cpp:238` | `GainOAID()` |
| **重置 OAID** | `services/oaid_manager/src/oaid_service.cpp:295` | `ResetOAID()` |
| **UUID 生成** | `services/oaid_manager/src/oaid_service.cpp:50` | `GetUUID()` |
| **权限检查** | `services/oaid_manager/src/oaid_service_stub.cpp:45` | `CheckPermission()` |
| **白名单检查** | `services/oaid_manager/src/oaid_service_stub.cpp:93` | `LoadAndCheckOaidTrustList()` |
| **系统应用检查** | `services/oaid_manager/src/oaid_service_stub.cpp:82` | `CheckSystemApp()` |
| **KVStore 读取** | `services/oaid_manager/src/oaid_service.cpp:193` | `ReadValueFromKvStore()` |
| **KVStore 写入** | `services/oaid_manager/src/oaid_service.cpp:216` | `WriteValueToKvStore()` |
| **文件读取** | `utils/native/src/oaid_file_operator.cpp:40` | `OpenAndReadFile()` |
| **N-API 实现** | `interfaces/kits/js/napi/oaid/src/oaid.cpp:192` | `GetOAID()` |
| **常量定义** | `services/oaid_manager/include/oaid_service_define.h` | 所有常量 |
| **IPC 接口** | `interfaces/innerkits/include/oaid_service_interface.h` | `IOAIDService` |

### 按代码行数排序

| 文件 | 行数 | 说明 |
|------|------|------|
| `oaid_service.cpp` | 439 | 核心业务实现 |
| `connect_ads_stub.cpp` | 388 | 扩展服务连接 |
| `oaid_service_stub.cpp` | 356 | IPC 请求处理 |
| `oaid_service_client.cpp` | 273 | 客户端 SDK |
| `oaid.cpp` | 261 | N-API 实现 |
| `oaid_init.cpp` | 56 | 模块注册 |
| `oaid_file_operator.cpp` | 73 | 文件操作 |

---

## 配置与资源文件

### 配置文件清单

| 文件 | 路径 | 用途 |
|------|------|------|
| **bundle.json** | 项目根目录 | Bundle 配置，依赖声明 |
| **oaid_service_config.json** | `services/oaid_manager/resource/` | 服务运行时配置 |
| **6101.json** | `profile/` | SA Profile（SA 管理器配置）|
| **oaidservice.cfg** | `etc/init/` | 服务启动配置（uid/gid/权限）|
| **BUILD.gn** | 各模块 | 构建配置 |

### 运行时文件路径

| 路径 | 类型 | 说明 |
|------|------|------|
| `/etc/advertising/oaid/oaid_service_config.json` | 配置 | 主配置文件 |
| `/etc/advertising/oaid/oaid_service_config_ext.json` | 配置 | 扩展配置文件 |
| `/data/service/el1/public/database/oaid_service_manager/` | 数据 | 数据库存储目录 |
| `/data/service/el1/public/database/oaid_service_manager/update_check.json` | 数据 | 更新检查文件 |

---

## 相关文档

- [项目概览](01_Overview.md)
- [架构分析](02_Architecture.md)
- [接口文档](04_Interface.md)
- [安全风险评估](06_SecurityReview.md)
