# 内部 API

---

## 目的

本文档说明 DBMS 服务的内部 API 定义、模块接口、依赖方向和接口稳定性。

---

## 适用范围

- ✅ IDistributedBms 接口定义
- ✅ Proxy 和 Stub 类
- ✅ 模块依赖关系
- ✅ 接口稳定性标注
- ❌ 详细的实现逻辑（请参考架构文档）

---

## IDistributedBms 接口

### 接口定义

**文件**: `interfaces/inner_api/include/distributed_bms_interface.h`

```cpp
class IDistributedBms : public IRemoteBroker {
public:
    DECLARE_INTERFACE_DESCRIPTOR(u"ohos.appexecfwk.IDistributedbms");

    // 远程 Ability 信息查询（单条）
    virtual int32_t GetRemoteAbilityInfo(const ElementName &elementName, RemoteAbilityInfo &remoteAbilityInfo);
    virtual int32_t GetRemoteAbilityInfo(const ElementName &elementName, const std::string &localeInfo,
                                         RemoteAbilityInfo &remoteAbilityInfo);

    // 远程 Ability 信息查询（批量）
    virtual int32_t GetRemoteAbilityInfos(const std::vector<ElementName> &elementNames,
                                          std::vector<RemoteAbilityInfo> &remoteAbilityInfos);
    virtual int32_t GetRemoteAbilityInfos(const std::vector<ElementName> &elementNames, const std::string &localeInfo,
                                          std::vector<RemoteAbilityInfo> &remoteAbilityInfos);

    // 本地 Ability 信息查询
    virtual int32_t GetAbilityInfo(const ElementName &elementName, RemoteAbilityInfo &remoteAbilityInfo);
    virtual int32_t GetAbilityInfo(const ElementName &elementName, const std::string &localeInfo,
                                   RemoteAbilityInfo &remoteAbilityInfo, DistributedBmsAclInfo *info = nullptr);

    // 批量本地 Ability 信息查询
    virtual int32_t GetAbilityInfos(const std::vector<ElementName> &elementNames,
                                    std::vector<RemoteAbilityInfo> &remoteAbilityInfos);
    virtual int32_t GetAbilityInfos(const std::vector<ElementName> &elementNames, const std::string &localeInfo,
                                    std::vector<RemoteAbilityInfo> &remoteAbilityInfos, DistributedBmsAclInfo *info = nullptr);

    // 分布式 Bundle 信息查询
    virtual bool GetDistributedBundleInfo(const std::string &networkId, const std::string &bundleName,
                                          DistributedBundleInfo &distributedBundleInfo);

    // 根据 AccessToken 获取 Bundle 名称
    virtual int32_t GetDistributedBundleName(const std::string &networkId, uint32_t accessTokenId, std::string &bundleName);
};
```

**证据**: `interfaces/inner_api/include/distributed_bms_interface.h:31-155`

---

## Proxy 类

### DistributedBmsProxy

**文件**: `interfaces/inner_api/include/distributed_bms_proxy.h`

**职责**: IPC 客户端代理，将本地调用转换为 IPC 消息

**继承**: `IRemoteProxy<IDistributedBms>`

**公共接口**:

| 方法 | 说明 | 文件位置 |
|------|------|----------|
| GetRemoteAbilityInfo() | 获取远程 Ability 信息 | distributed_bms_proxy.cpp |
| GetRemoteAbilityInfos() | 批量获取远程 Ability 信息 | distributed_bms_proxy.cpp |
| GetAbilityInfo() | 获取 Ability 信息 | distributed_bms_proxy.cpp |
| GetAbilityInfos() | 批量获取 Ability 信息 | distributed_bms_proxy.cpp |
| GetDistributedBundleInfo() | 获取分布式 Bundle 信息 | distributed_bms_proxy.cpp |
| GetDistributedBundleName() | 根据 Token 获取 Bundle 名称 | distributed_bms_proxy.cpp |

**私有方法**:

| 方法 | 说明 |
|------|------|
| SendRequest() | 发送 IPC 请求，处理序列化和反序列化 |
| WriteParcelableVector() | 序列化 Parcelable 对象数组 |
| GetParcelableInfo() | 获取 Parcelable 对象 |
| GetParcelableInfos() | 获取 Parcelable 对象数组 |
| CheckElementName() | 验证 ElementName 参数 |

**证据**: `interfaces/inner_api/include/distributed_bms_proxy.h:27-131`

---

## Stub 类

### DistributedBmsHost

**文件**: `services/dbms/include/distributed_bms_host.h`

**职责**: IPC 服务端 Stub，接收 IPC 消息并分发到处理函数

**继承**: `IRemoteStub<IDistributedBms>`

**公共接口**:

| 方法 | 说明 |
|------|------|
| DistributedBmsHost() | 构造函数 |
| ~DistributedBmsHost() | 析构函数 |
| OnRemoteRequest() | 接收 IPC 请求，根据命令码分发 |

**私有方法**:

| 方法 | 说明 |
|------|------|
| HandleGetRemoteAbilityInfo() | 处理 GET_REMOTE_ABILITY_INFO 命令 |
| HandleGetRemoteAbilityInfos() | 处理 GET_REMOTE_ABILITY_INFOS 命令 |
| HandleGetAbilityInfo() | 处理 GET_ABILITY_INFO 命令 |
| HandleGetAbilityInfos() | 处理 GET_ABILITY_INFOS 命令 |
| HandleGetDistributedBundleInfo() | 处理 GET_DISTRIBUTED_BUNDLE_INFO 命令 |
| HandleGetDistributedBundleName() | 处理 GET_DISTRIBUTED_BUNDLE_NAME 命令 |
| GetParcelableInfos() | 通用 Parcelable 数组获取方法 |
| WriteParcelableVector() | 通用 Parcelable 数组写入方法 |

**证据**: `services/dbms/include/distributed_bms_host.h:26-44`

---

## IPC 命令码

### DistributedInterfaceCode 枚举

**文件**: `interfaces/inner_api/include/distributed_bundle_ipc_interface_code.h`

| 命令码 | 值 | 对应方法 | Proxy | Stub 处理 |
|--------|-----|----------|--------|--------------|
| GET_REMOTE_ABILITY_INFO | 0 | GetRemoteAbilityInfo() | HandleGetRemoteAbilityInfo() |
| GET_REMOTE_ABILITY_INFOS | 1 | GetRemoteAbilityInfos() | HandleGetRemoteAbilityInfos() |
| GET_ABILITY_INFO | 2 | GetAbilityInfo() | HandleGetAbilityInfo() |
| GET_ABILITY_INFOS | 3 | GetAbilityInfos() | HandleGetAbilityInfos() |
| GET_ABILITY_INFO_WITH_LOCALE | 6 | GetAbilityInfo(locale) | HandleGetAbilityInfo() |
| GET_ABILITY_INFOS_WITH_LOCALE | 7 | GetAbilityInfos(locale) | HandleGetAbilityInfos() |
| GET_DISTRIBUTED_BUNDLE_INFO | 8 | GetDistributedBundleInfo() | HandleGetDistributedBundleInfo() |
| GET_DISTRIBUTED_BUNDLE_NAME | 9 | GetDistributedBundleName() | HandleGetDistributedBundleName() |

---

## 数据结构

### DistributedBmsAclInfo

**文件**: `interfaces/inner_api/include/distributed_bms_acl_info.h`

```cpp
struct DistributedBmsAclInfo : public Parcelable {
    std::string networkId;   // 网络 ID
    int32_t userId = 0;        // 用户 ID
    std::string accountId;    // 账号 ID
    uint64_t tokenId = 0;       // 访问令牌 ID
    std::string pkgName;      // 包名

    // Parcelable 方法
    bool Marshalling(Parcel &parcel) const override;
    bool Unmarshalling(Parcel &parcel) override;
};
```

**证据**: `interfaces/inner_api/include/distributed_bms_acl_info.h:16-23`

### RemoteAbilityInfo

**文件**: 来自 bundle_framework（外部依赖）

```cpp
struct RemoteAbilityInfo : public Parcelable {
    ElementName elementName;  // Ability 组件标识
    std::string label;        // 本地化标签
    std::string icon;         // 图标（Base64 编码）
};
```

**证据**: `services/dbms/include/distributed_bms.h:47-48` (include 语句)

---

## 模块依赖关系

### 依赖方向

```
interfaces/inner_api (dbms_fwk)
    │
    ├─→ 无（被依赖）
    │
    └──→ services/dbms (libdbms)
              │
              ├─→ bundle_framework (IBundleMgr SA)
              ├─→ device_manager (DeviceManager SA)
              ├─→ access_token (权限验证）
              ├─→ account_manager (账号服务）
              ├─→ common_event_service (事件订阅）
              └─→ 其他基础设施（hilog, ipc, safwk, samgr 等）

interfaces/kits/js (distributed_bundle, distributedbundlemanager)
    │
    └──→ interfaces/inner_api:dbms_fwk (通过 IPC)
```

### 依赖层次

| 层级 | 模块 | 依赖的上层 | 被依赖的下层 |
|------|------|------------|------------|
| 1. JS 应用层 | interfaces/kits/js | 无 | interfaces/inner_api |
| 2. N-API 绑定层 | interfaces/kits/js | 无 | 无（JS 层调用）|
| 3. IPC 代理层 | interfaces/inner_api | JS 层 | services/dbms |
| 4. 服务层 | services/dbms | IPC 代理层 | 外部系统服务 |
| 5. 外部系统服务 | bundle_framework, device_manager 等 | services/dbms | 无 |

---

## 接口稳定性

### 稳定接口（公共 API）

| 接口 | 稳定性 | 说明 | 证据 |
|------|--------|------|--------|
| IDistributedBms | **稳定** | interfaces/inner_api/include | 定义明确的 IPC 接口，外部模块依赖此接口 |
| DistributedBmsProxy | **内部** | interfaces/inner_api/include | Proxy 实现细节可能变化 |
| DistributedBmsHost | **内部** | services/dbms/include | IPC Stub 实现细节可能变化 |

### 不稳定接口（内部实现）

| 接口 | 稳定性 | 说明 | 证据 |
|------|--------|------|--------|
| DistributedBms | **内部** | services/dbms/include | 服务实现类，内部逻辑可能变化 |
| DbmsDeviceManager | **内部** | services/dbms/include | 设备管理器，内部实现 |
| DistributedDataStorage | **内部** | services/dbms/include | 数据存储，内部实现 |

### 判断依据

**稳定接口判断标准**:
1. **接口定义在 include 目录下**
2. **被其他模块或外部依赖引用**
3. **命名规范，包含明确的接口描述符**
4. **不依赖内部实现细节**

**不稳定接口判断标准**:
1. **实现在 src 目录下**
2. **包含业务逻辑**
3. **可能因需求变化而修改**

**证据**: 目录结构分析（`interfaces/inner_api/include/` vs `services/dbms/src/`）

---

## 使用示例

### 客户端使用 Proxy

```cpp
#include "distributed_bms_proxy.h"
#include "if_system_ability_manager.h"

using namespace OHOS::AppExecFwk;

// 获取 DBMS 服务代理
auto samgr = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
auto remoteObject = samgr->GetSystemAbility(DISTRIBUTED_BUNDLE_MGR_SERVICE_SYS_ABILITY_ID);
auto distributedBmsProxy = OHOS::iface_cast<IDistributedBms>(remoteObject);

// 调用远程方法
RemoteAbilityInfo remoteAbilityInfo;
ElementName elementName = {...};
int32_t result = distributedBmsProxy->GetRemoteAbilityInfo(elementName, remoteAbilityInfo);
```

**证据**: `interfaces/kits/js/distributebundlemgr/distributed_bundle_mgr.cpp:71-81`

### 服务端实现接口

```cpp
#include "distributed_bms.h"

using namespace OHOS::AppExecFwk;

class DistributedBms : public SystemAbility, public DistributedBmsHost {
public:
    // 实现 IDistributedBms 的所有纯虚函数
    int32_t GetRemoteAbilityInfo(const ElementName &elementName, RemoteAbilityInfo &remoteAbilityInfo) override;
    // ... 其他方法实现
};
```

**证据**: `services/dbms/include/distributed_bms.h:33-168`

---

## 证据索引

| 结论 | 证据来源 |
|------|----------|
| IDistributedBms 接口 | interfaces/inner_api/include/distributed_bms_interface.h:31-155 |
| Proxy 类定义 | interfaces/inner_api/include/distributed_bms_proxy.h:27-131 |
| Stub 类定义 | services/dbms/include/distributed_bms_host.h:26-44 |
| IPC 命令码 | interfaces/inner_api/include/distributed_bundle_ipc_interface_code.h |
| ACL 信息结构 | interfaces/inner_api/include/distributed_bms_acl_info.h:16-23 |
