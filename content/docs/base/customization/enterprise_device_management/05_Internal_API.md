# 内部API

## 目的

本文档说明EDM组件的内部C++接口、模块依赖方向和稳定性说明。

## 适用范围

- 目标读者：EDM组件开发者、插件开发者
- 覆盖内容：内部接口列表、依赖关系、稳定/不稳定接口
- 不包含：公开JS/N-API接口

## 关键结论

- 内部API分为：Proxy接口（跨进程）、Plugin接口（插件系统）
- Proxy层提供访问EDM服务的C++接口
- Plugin接口定义策略插件的标准接口

## 相关跳转

- [03_Architecture.md](03_Architecture.md) - 架构和数据流
- [04_External_API_NAPI.md](04_External_API_NAPI.md) - 公开JS API

---

## 核心接口列表

### 1. EDM服务接口

| 接口 | 文件 | 职责 | 稳定性 |
|------|------|------|--------|
| IEnterpriseDeviceMgr | `interfaces/inner_api/common/include/ienterprise_device_mgr.h` | EDM服务主接口 | 稳定 |
| EnterpriseDeviceMgrProxy | `interfaces/inner_api/common/src/enterprise_device_mgr_proxy.cpp` | EDM服务Proxy | 稳定 |
| EnterpriseDeviceMgrStub | `services/edm/include/enterprise_device_mgr_stub.h` | EDM服务Stub | 稳定 |

证据：`interfaces/inner_api/common/include/ienterprise_device_mgr.h:23-31`

**主要方法**：
```cpp
// IEnterpriseDeviceMgr接口
virtual ErrCode EnableAdmin(const AppExecFwk::ElementName &admin,
    const EntInfo &entInfo, AdminType type, int32_t userId, bool isDebug) = 0;
virtual ErrCode DisableAdmin(const AppExecFwk::ElementName &admin,
    int32_t userId) = 0;
virtual ErrCode GetDevicePolicy(uint32_t code, MessageParcel &data,
    MessageParcel &reply, int32_t userId, int32_t hasUserId) = 0;
virtual ErrCode HandleDevicePolicy(uint32_t code, MessageParcel &data,
    MessageParcel &reply, int32_t userId, int32_t hasUserId) = 0;
// ... 约100+个接口
```

### 2. 插件接口

| 接口 | 文件 | 职责 | 稳定性 |
|------|------|------|--------|
| IPlugin | `interfaces/inner_api/plugin_kits/include/iplugin.h` | 插件基接口 | 稳定 |
| IPluginManager | `interfaces/inner_api/plugin_kits/include/iplugin_manager.h` | 插件管理器接口 | 稳定 |
| IPluginExecuteStrategy | `interfaces/inner_api/plugin_kits/include/iplugin_execute_strategy.h` | 执行策略接口 | 稳定 |
| IPolicyManager | `interfaces/inner_api/plugin_kits/include/ipolicy_manager.h` | 策略管理器接口 | 稳定 |
| IPolicySerializer | `interfaces/inner_api/plugin_kits/include/ipolicy_serializer.h` | 策略序列化接口 | 稳定 |

证据：`interfaces/inner_api/plugin_kits/include/*.h`

### 3. 管理器Proxy接口

| Proxy | 文件 | 目标服务 | 说明 |
|------|------|----------|------|
| AdminManagerProxy | `interfaces/inner_api/common/include/admin_manager_proxy.h` | EDM服务 | 管理员操作 |
| AccountManagerProxy | `interfaces/inner_api/account_manager/include/account_manager_proxy.h` | EDM服务 | 账户管理 |
| ApplicationManagerProxy | `interfaces/inner_api/application_manager/include/application_manager_proxy.h` | EDM服务 | 应用管理 |
| DeviceSettingsProxy | `interfaces/inner_api/device_settings/include/device_settings_proxy.h` | EDM服务 | 设备设置 |
| NetworkManagerProxy | `interfaces/inner_api/network_manager/include/network_manager_proxy.h` | EDM服务 | 网络管理 |

证据：`interfaces/inner_api/*/include/*_proxy.h`

### 4. 策略查询接口

| 接口 | 文件 | 说明 |
|------|------|------|
| IPolicyQuery | `services/edm/include/query_policy/ipolicy_query.h` | 策略查询基接口 |
| IPasswordPolicyQuery | `services/edm/include/query_policy/password_policy_query.h` | 密码策略查询 |
| IAllowedInstallBundlesQuery | `services/edm/include/query_policy/allowed_install_bundles_query.h` | 应用白名单查询 |

证据：`services/edm/include/query_policy/*.h`

---

## IPlugin接口详解

### 插件类型定义

```cpp
// IPlugin::PluginType
enum class PluginType {
    BASIC = 0,        // 基础插件（device_core, communication, sys_service）
    EXTENSION,        // 扩展插件（可增强基础插件）
};

// IPlugin::PermissionType
enum class PermissionType {
    NORMAL_DEVICE_ADMIN = 0,    // 普通设备管理员
    SUPER_DEVICE_ADMIN,        // 超级设备管理员
    BYOD_DEVICE_ADMIN,          // BYOD管理员
    UNKNOWN,
};

// IPlugin::ApiType
enum class ApiType {
    PUBLIC = 0,    // 公开API（三方应用）
    SYSTEM,         // 系统API（系统应用）
    UNKNOWN,
};
```

证据：`interfaces/inner_api/plugin_kits/include/iplugin.h:36-52`

### 核心方法

| 方法 | 说明 | 调用时机 |
|------|------|----------|
| OnHandlePolicy | 处理策略设置请求 | EDM服务设置策略时 |
| OnGetPolicy | 查询策略数据 | EDM服务查询策略时 |
| OnHandlePolicyDone | 策略处理完成回调 | 策略执行后通知 |
| OnAdminRemove | 管理员移除 | 管理员被禁用时 |
| OnAdminRemoveDone | 管理员移除完成 | 所有插件清理后 |
| OnGetPolicy | 获取策略数据 | 查询策略时 |
| GetOthersMergePolicyData | 合并策略数据 | 多管理员策略合并时 |

证据：`interfaces/inner_api/plugin_kits/include/iplugin.h:86-109`

### 权限配置

```cpp
// IPlugin::PolicyPermissionConfig
struct PolicyPermissionConfig {
    // 基于管理类型的权限映射
    std::map<PermissionType, std::string> typePermissions;
    
    // API类型（PUBLIC/SYSTEM）
    ApiType apiType;
    
    // 基于策略标签的权限映射
    std::map<std::string, std::map<PermissionType, std::string>> tagPermissions;
};
```

证据：`interfaces/inner_api/plugin_kits/include/iplugin.h:54-75`

---

## IPluginManager接口详解

### 核心方法

| 方法 | 说明 |
|------|------|
| AddPlugin | 添加插件（静态加载） |
| AddExtensionPlugin | 添加扩展插件（动态增强基础插件） |
| GetPluginByFuncCode | 根据功能码获取插件 |
| HandlePolicy | 分发策略请求到插件 |
| GetPolicy | 从插件查询策略 |
| RemoveAdminItem | 移除管理员相关插件项 |

证据：`interfaces/inner_api/plugin_kits/include/iplugin_manager.h:29-41`

### 插件管理流程

```
EDM服务设置策略
    │
    ▼
PluginManager::HandlePolicy(funcCode, data, reply, userId)
    │
    ├─► GetPluginByFuncCode(funcCode)
    │     │
    │     ▼
    │ ┌──────────────────┐
    │ │ 插件已加载？  │
    │ └──────────────────┘
    │         │           │
    │      否            是
    │         │           │
    │         ▼           ▼
    │ LoadPlugin()  直接调用插件的  直接调用插件
    │         OnHandlePolicy()  OnHandlePolicy()
    │
    └────────────► 返回执行结果
```

---

## IPolicySerializer接口详解

### 序列化工具

| 接口 | 文件 | 数据类型 | 说明 |
|------|------|----------|------|
| StringSerializer | `interfaces/inner_api/plugin_kits/include/utils/string_serializer.h` | std::string | 字符串序列化 |
| ArrayStringSerializer | `interfaces/inner_api/plugin_kits/include/utils/array_string_serializer.h` | std::vector<std::string> | 字符串数组 |
| BoolSerializer | `interfaces/inner_api/plugin_kits/include/utils/bool_serializer.h` | bool | 布尔值 |
| IntSerializer/UIntSerializer/LongSerializer | `interfaces/inner_api/plugin_kits/include/utils/*_serializer.h` | 整数类型 | 数字序列化 |
| MapStringSerializer | `interfaces/inner_api/plugin_kits/include/utils/map_string_serializer.h` | std::map<std::string, std::string> | Map序列化 |
| CjsonSerializer | `interfaces/inner_api/plugin_kits/include/utils/cjson_serializer.h` | JSON | cJSON格式 |

证据：`interfaces/inner_api/plugin_kits/include/utils/*.h`

---

## 模块依赖方向

### 依赖层次

```
┌─────────────────────────────────────────┐
│         interfaces/inner_api              │
│         (内部API - Proxy层）                   │
└──────────────┬───────────────────────────┘
               │ depends
               ▼
┌─────────────────────────────────────────┐
│            services/edm                   │
│         (EDM服务 - 实现层）                │
└──────────────┬───────────────────────────┘
               │ depends
               ▼
┌─────────────────────────────────────────┐
│         interfaces/inner_api/plugin_kits   │
│         (插件接口 - 定义层）                 │
└─────────────────────────────────────────┘
               │ implements
               ▼
┌─────────────────────────────────────────┐
│         services/edm_plugin                 │
│         (插件实现 - 策略层）                  │
└─────────────────────────────────────────┘
```

### 关键依赖说明

| 调用者 | 被调用者 | 依赖类型 | 稳定性 |
|--------|----------|----------|--------|
| 所有Manager Proxy | EnterpriseDeviceMgrProxy | 依赖EDM服务可用 | 稳定 |
| EnterpriseDeviceMgrAbility | AdminManager、PolicyManager、PluginManager | 内部组合依赖 | 稳定 |
| PluginManager | IPlugin实例 | 接口依赖（dlopen动态） | 稳定接口 |

---

## 稳定性与可替换点

### 稳定接口（可安全依赖）

| 接口 | 稳定性 | 说明 |
|------|----------|------|
| IEnterpriseDeviceMgr | ✓ 稳定 | 系统定义，SA接口标准 |
| IPlugin | ✓ 稳定 | 插件标准接口，所有插件实现 |
| IPluginManager | ✓ 稳定 | 插件管理器接口 |
| IPolicySerializer | ✓ 稳定 | 序列化工具接口 |
| 各Manager Proxy | ✓ 稳定 | Proxy层接口 |
| Query Policy接口 | ✓ 稳定 | 策略查询接口 |

### 不稳定接口（内部实现，可能变化）

| 接口 | 稳定性 | 说明 |
|------|----------|------|
| AdminManager内部类 | ⚠️ 部分不稳定 | AdminManager具体实现可能调整 |
| PolicyManager内部类 | ⚠️ 部分不稳定 | 策略存储实现可能优化 |
| PluginManager内部类 | ⚠️ 部分不稳定 | 插件管理机制可能演进 |
| 各Query Policy实现类 | ⚠️ 部分不稳定 | 查询策略实现可能变化 |

### 可替换点

| 模块 | 可替换点 | 替换方式 |
|------|----------|----------|
| 策略插件 | IPlugin实现 | 实现IPlugin接口即可 |
| 序列化工具 | IPolicySerializer实现 | 实现序列化接口 |
| 外部系统适配器 | External Manager Wrapper | 修改common/external下的适配器 |

---

## 内部数据结构

### 策略数据结构

```cpp
// 来自policy_struct.h
struct ManagedPolicy {
    std::string policyName;        // 策略名称
    std::string policyData;         // 策略数据（JSON或序列化）
    int32_t userId;               // 用户ID
    std::string adminName;          // 管理员名称
};

struct AdminInfo {
    std::string packageName_;        // Bundle名称
    std::string className_;          // Ability名称
    std::string entName_;           // 企业名称
    std::vector<std::string> permission_;  // 权限列表
    AdminType adminType_;           // 管理员类型
    bool isDebug_;                // 是否Debug模式
};
```

证据：`interfaces/inner_api/common/include/managed_policy.h`, `interfaces/inner_api/common/include/admin_info.h`

---

## 相关跳转

- [02_Directory_Structure.md](02_Directory_Structure.md) - 完整目录结构
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链详细说明
