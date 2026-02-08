# 分布式包管理服务 (DBMS) - 概览

---

## 目的

本文档提供分布式包管理服务 (DBMS, Distributed Bundle Management Service) 的整体概览，帮助新人快速理解项目的定位、核心能力、运行环境和关键概念。

---

## 适用范围

- ✅ DBMS 服务的核心功能定位
- ✅ 系统能力 (System Capability) 定义
- ✅ 主要依赖组件
- ✅ 核心数据结构
- ❌ 详细的实现细节（请参考其他文档）
- ❌ 测试用例说明

---

## 项目定位

分布式包管理服务 (DBMS) 是 OpenHarmony 系统中负责 **跨设备组件调度和任务管理** 的系统服务，核心职责包括：

### 核心能力

1. **跨设备 RPC 能力**：支持应用通过 JS API 查询远程设备上的组件信息
2. **按需获取资源**：支持跨设备获取指定语言的 Ability 资源（如图标、标签）
3. **分布式数据同步**：监听设备状态变化，维护分布式组件信息

### 在系统中的角色

DBMS 是 OpenHarmony **bundlemanager 子系统** 的一部分，作为系统服务 (System Ability) 运行：

- **SA ID**: 402 (DISTRIBUTED_BUNDLE_MGR_SERVICE_SYS_ABILITY_ID)
- **进程名**: d-bms
- **子系统**: bundlemanager
- **系统能力**: SystemCapability.BundleManager.DistributedBundleFramework

---

## 运行环境

### 支持的设备类型

- **系统**: standard (标准系统)
- **架构**: 无特定限制（与 OpenHarmony 支持的架构一致）

### 资源占用

| 资源类型 | 占用量 | 说明 |
|----------|--------|------|
| ROM | ~400KB | 存储占用（库文件和配置） |
| RAM | ~6577KB | 运行时内存占用 |

---

## 核心依赖组件

DBMS 依赖以下外部组件（来自 `bundle.json`）：

| 组件名 | 用途 | 关键能力 |
|--------|------|----------|
| ability_base | Want 参数 | 跨组件通信 |
| access_token | 访问控制 | 权限验证 |
| bundle_framework | 包管理框架 | Bundle 信息查询 |
| dsoftbus | 分布式软总线 | 跨设备通信 |
| ipc | IPC 通信 | 系统服务间通信 |
| napi | N-API | JavaScript 绑定 |
| safwk | 系统服务框架 | SA 生命周期管理 |
| samgr | 系统能力管理 | SA 注册和发现 |
| device_manager | 设备管理 | 跨设备设备发现 |
| image_framework | 图片框架 | 图标资源处理 |
| resource_management | 资源管理 | 多语言资源 |
| kv_store | 分布式数据存储 | 本地缓存 |

### 依赖的 SA

| SA ID | SA 名称 | 用途 |
|--------|---------|------|
| 401 | Bundle Manager Service (BMS) | 查询本地和远程 Bundle 信息 |
| - | Device Manager Service | 设备发现和状态管理 |

---

## 关键概念

### 1. System Ability (SA)

**定义**: OpenHarmony 系统服务的基本单元，提供系统级能力给应用使用。

**DBMS SA**:
- **SA ID**: 402
- **进程**: d-bms
- **启动模式**: 按需启动（deviceonline 事件触发）
- **分布式**: true（支持跨设备查询）

**证据**: `services/dbms/sa_profile/402.json:4-5`

### 2. Remote Ability Info

**定义**: 远程设备上 Ability 组件的信息，包含标签、图标和组件名称。

**结构**:
```cpp
struct RemoteAbilityInfo {
    ElementName elementName;  // 设备ID、包名、模块名、能力名
    std::string label;        // 本地化标签
    std::string icon;         // 图标（Base64 编码）
};
```

**证据**: `interfaces/inner_api/include/distributed_bms_interface.h:27`

### 3. ElementName

**定义**: 唯一标识一个 Ability 组件的名称，包含设备信息。

**结构**:
```cpp
struct ElementName {
    std::string deviceId;    // 设备 ID（远程设备或本地设备）
    std::string bundleName;  // 应用包名
    std::string moduleName;  // 模块名（可选）
    std::string abilityName; // 能力名称
};
```

**证据**: `services/dbms/src/distributed_bms.cpp:46-48`

### 4. ACL (Access Control List)

**定义**: 访问控制列表，用于跨设备场景下的权限验证。

**结构**:
```cpp
struct DistributedBmsAclInfo {
    std::string networkId;   // 网络 ID
    int32_t userId;        // 用户 ID
    std::string accountId;    // 账号 ID
    uint64_t tokenId;       // 访问令牌 ID
    std::string pkgName;      // 包名
};
```

**证据**: `interfaces/inner_api/include/distributed_bms_acl_info.h:16-23`

---

## 主要功能模块

### JS API 层（应用接口）

提供 JavaScript 应用编程接口：

| 命名空间 | 模块名 | 方法 |
|----------|--------|------|
| distributedBundle | libdistributedbundle.z.so | getRemoteAbilityInfo, getRemoteAbilityInfos |
| bundle.distributedBundleManager | libdistributedbundlemanager.z.so | getRemoteAbilityInfo |

**证据**: `interfaces/kits/js/distributedBundle/native_module.cpp:54`, `interfaces/kits/js/distributebundlemgr/native_module.cpp:53`

### 内部 API 层（服务接口）

定义 DBMS 服务的 IPC 接口：

**接口**: `IDistributedBms`

**证据**: `interfaces/inner_api/include/distributed_bms_interface.h:31-42`

### 服务实现层（核心逻辑）

实现 DBMS 系统服务的核心逻辑：

**类**: `DistributedBms` (继承自 `SystemAbility` 和 `DistributedBmsHost`)

**证据**: `services/dbms/include/distributed_bms.h:33-35`

---

## 系统架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (JavaScript)                   │
│  ┌───────────────────────────────────────────┐      │
│  │ distributedBundle                           │      │
│  │ - getRemoteAbilityInfo()                 │      │
│  │ - getRemoteAbilityInfos()                │      │
│  └───────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                  N-API 绑定层                        │
│  ┌───────────────────────────────────────────┐      │
│  │ native_module.cpp                        │      │
│  │ distributed_bundle_mgr.cpp              │      │
│  │ distributed_bundle.cpp                 │      │
│  └───────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│              IPC 代理层 (DistributedBmsProxy)         │
│  ┌───────────────────────────────────────────┐      │
│  │ GetRemoteAbilityInfo()                 │      │
│  │ GetRemoteAbilityInfos()                │      │
│  └───────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│         系统服务层 (DistributedBms SA = 402)        │
│  ┌───────────────────────────────────────────┐      │
│  │ GetRemoteAbilityInfo()                 │      │
│  │ 权限验证：VerifyCallingPermission      │      │
│  │ 设备查询：DeviceManager               │      │
│  │ Bundle查询：BundleManager SA           │      │
│  └───────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│           外部系统服务                                  │
│  ┌─────────────────┐  ┌─────────────────┐    │
│  │ Bundle Manager   │  │ Device Manager  │    │
│  │ SA = 401        │  │ Service         │    │
│  └─────────────────┘  └─────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 数据流概览

### 典型调用流程：查询远程 Ability 信息

```
1. 应用调用 JS API
   distributedBundle.getRemoteAbilityInfo(elementName, callback)

2. N-API 参数解析
   ParseElementName() → 提取 deviceId, bundleName, abilityName

3. 异步工作队列
   napi_queue_async_work() → 工作线程

4. IPC 调用 DBMS 服务
   GetDistributedBundleMgr() → GetRemoteAbilityInfo()

5. DBMS 服务处理
   - 权限验证：VerifyCallingPermission()
   - ACL 检查：CheckAclData()（跨设备）
   - 查询 Bundle Manager (SA = 401)

6. 返回结果
   ConvertRemoteAbilityInfo() → 构造 JS 对象

7. 回调/Promise 解析
   callback(err, data) 或 Promise.resolve(data)
```

---

## 关键配置

### Feature Flags

| Flag | 默认值 | 说明 |
|------|--------|------|
| distributed_bundle_framework_graphics | true | 图形功能支持 |
| ability_runtime_enable_dbms | true | Ability 运行时集成 |
| account_enable_dbms | true | 账号系统集成 |
| hisysevent_enable_dbms | true | HiSysEvent 事件上报 |
| distributed_bundle_image_framework_enable | true | 图片框架支持 |

**证据**: `dbms.gni:28-59`

### SA 配置

**启动配置** (`services/dbms/sa_profile/distributedbms.cfg:11-29`):
- 进程: d-bms
- 启动方式: /system/bin/sa_main /system/profile/d-bms.json
- 按需启动: true（deviceonline 事件触发）
- 权限:
  - ohos.permission.DISTRIBUTED_DATASYNC
  - ohos.permission.GET_BUNDLE_INFO_PRIVILEGED
  - ohos.permission.GET_INSTALLED_BUNDLE_LIST
  - ohos.permission.ACCESS_SERVICE_DM
  - ohos.permission.MANAGE_LOCAL_ACCOUNTS
  - ohos.permission.GET_BUNDLE_RESOURCES

---

## 相关文档链接

- [目录结构](./02_Directory_Structure.md) - 详细的代码组织说明
- [架构设计](./03_Architecture.md) - 完整的架构分析
- [JS API 参考](./04_JS_API.md) - 完整的 API 清单
- [GN 构建系统](./06_GN_Build.md) - 编译配置和产物

---

## 证据索引

| 结论 | 证据来源 |
|------|----------|
| SA ID = 402 | services/dbms/sa_profile/402.json:5 |
| 进程名 = d-bms | services/dbms/sa_profile/distributedbms.cfg:11 |
| JS 模块注册 | interfaces/kits/js/distributedBundle/native_module.cpp:49-56 |
| IPC 接口定义 | interfaces/inner_api/include/distributed_bms_interface.h:31-42 |
| 权限检查函数 | services/dbms/include/distributed_bms.h:166-167 |
