# Inner API 参考

> 本文档描述 app_domain_verify 部件对系统内部暴露的 Inner API，主要供包管理子系统内部调用。

## 1. 客户端接口

### 1.1 AppDomainVerifyMgrClient

**头文件**: `interfaces/inner_api/client/include/app_domain_verify_mgr_client.h`

**功能**: 提供给 BundleManagerService 调用的客户端接口

```cpp
#include "app_domain_verify_mgr_client.h"
```

#### API 清单

| 方法 | 描述 | 同步/异步 | 权限要求 |
|-----|------|---------|---------|
| `VerifyDomain()` | 触发域名校验任务 | 异步 | SA 调用 |
| `ClearDomainVerifyStatus()` | 清除校验状态 | 同步 | SA 调用 |
| `FilterAbilities()` | 过滤 ability | 同步 | SA 调用 |
| `QueryDomainVerifyStatus()` | 查询校验状态 | 同步 | SA 调用 |
| `QueryAllDomainVerifyStatus()` | 查询所有状态 | 同步 | SA 调用 |
| `SaveDomainVerifyStatus()` | 保存校验状态 | 同步 | SA 调用 |
| `IsAtomicServiceUrl()` | 判断原子服务 URL | 同步 | SA 调用 |
| `ConvertToExplicitWant()` | 转换显式 want | 异步 | SA 调用 |
| `UpdateWhiteListUrls()` | 更新白名单 URL | 同步 | SA 调用 |
| `QueryAssociatedDomains()` | 查询关联域名 | 同步 | `ohos.permission.GET_APP_DOMAIN_BUNDLE_INFO` |
| `QueryAssociatedBundleNames()` | 查询关联包名 | 同步 | `ohos.permission.GET_APP_DOMAIN_BUNDLE_INFO` |
| `QueryAppDetailsWant()` | 查询应用详情 want | 同步 | SA 调用 |
| `PopDeferredLink()` | 获取延迟链接 | 同步 | SA 调用 |
| `GetDeferredLink()` | 获取延迟链接 | 同步 | SA 调用 |
| `QueryAbilityInfos()` | 查询 ability 信息 | 同步 | SA 调用 |

#### 关键数据结构

**SkillUri** (skill_uri.h)
```cpp
struct SkillUri {
    std::string scheme;       // URI 协议
    std::string host;         // 主机地址
    std::string port;         // 端口
    std::string path;         // 路径
    std::string pathStartWith;// 路径前缀
    std::string pathRegex;    // 路径正则
    std::string type;         // MIME 类型
};
```

**DomainVerifyStatus**
```cpp
enum class DomainVerifyStatus {
    NOT_VERIFIED = 0,
    VERIFY_SUCCESS,
    VERIFY_FAILED,
    EXPIRED
};
```

**BundleVerifyStatusInfo**
```cpp
struct BundleVerifyStatusInfo {
    std::string bundleName;
    std::vector<DomainVerifyStatus> domainStatuses;
};
```

#### 使用示例

```cpp
// 触发域名校验
auto& client = AppDomainVerifyMgrClient::GetInstance();
client.VerifyDomain(appIdentifier, bundleName, fingerprint, skillUris);

// 过滤 ability
std::vector<AbilityInfo> filtered;
client.FilterAbilities(want, originAbilityInfos, filtered);
```

### 1.2 AppDomainVerifyAgentClient

**头文件**: `interfaces/inner_api/client/include/app_domain_verify_agent_client.h`

**功能**: 提供给 Agent Service 调用的客户端接口

#### API 清单

| 方法 | 描述 |
|-----|------|
| `SingleVerify()` | 执行单次校验 |
| `CommonTransact()` | 通用事务处理 |

## 2. 公共数据结构

### 2.1 通用数据结构

**AppVerifyBaseInfo** (app_verify_base_info.h)
```cpp
struct AppVerifyBaseInfo {
    std::string appIdentifier;      // 应用标识
    std::string bundleName;         // 包名
    std::string fingerprint;        // 签名指纹
    std::vector<SkillUri> skillUris;// URL 能力列表
};
```

**InnerVerifyStatus** (inner_verify_status.h)
```cpp
enum class InnerVerifyStatus {
    SUCCESS = 0,
    FAIL,
    PENDING,
    EXPIRED,
    NOT_FOUND
};
```

**TargetInfo** (zidl/target_info.h)
```cpp
struct TargetInfo {
    std::string packageName;
    std::vector<std::string> fingerprints;
    std::vector<std::string> paths;
};
```

### 2.2 IPC 接口定义

**IConvertCallback** (zidl/i_convert_callback.h)
```cpp
class IConvertCallback : public IRemoteBroker {
public:
    virtual void OnConvertFinished(const Want& result) = 0;
};
```

## 3. 扩展框架接口

### 3.1 AppDomainVerifyExtensionMgr

**头文件**: `frameworks/extension/include/app_domain_verify_extension_mgr.h`

**功能**: 管理 AppDomainVerifyExtensionAbility

#### API 清单

| 方法 | 描述 |
|-----|------|
| `GetExtension()` | 获取扩展实例 |
| `RegisterExtension()` | 注册扩展 |
| `UnregisterExtension()` | 注销扩展 |

### 3.2 AppDomainVerifyExtBase

**头文件**: `frameworks/extension/include/app_domain_verify_ext_base.h`

**功能**: ExtensionAbility 基类

## 4. 框架公共接口

### 4.1 HTTP 任务管理

**IHttpTask** (httpsession/i_http_task.h)
```cpp
class IHttpTask {
    virtual int Execute() = 0;
    virtual void SetCallback(VerifyResultCallback callback) = 0;
};
```

**AppDomainVerifyTaskMgr** (httpsession/app_domain_verify_task_mgr.h)
```cpp
class AppDomainVerifyTaskMgr {
    void AddTask(std::shared_ptr<IHttpTask> task);
    void RemoveTask(const std::string& taskId);
    std::shared_ptr<IHttpTask> GetTask(const std::string& taskId);
};
```

### 4.2 权限管理

**PermissionManager** (frameworks/common/include/permission/permission_manager.h)
```cpp
class PermissionManager {
    static bool CheckPermission(const std::string& permission);
    static bool IsSystemAppCall();
    static bool IsSACall();
    static bool IsAgentCall();
};
```

### 4.3 工具类

**DomainUrlUtil** (utils/domain_url_util.h)
```cpp
class DomainUrlUtil {
    static bool IsValidUrl(const std::string& url);
    static std::string ExtractDomain(const std::string& url);
    static bool IsShortUrl(const std::string& url);
};
```

## 5. 校验器接口

### 5.1 VerifyTask

**头文件**: `frameworks/verifier/include/verify_task.h`

**功能**: 校验任务基类

```cpp
class VerifyTask {
    virtual void Execute() = 0;
    virtual void Cancel() = 0;
    virtual VerifyResult GetResult() = 0;
};
```

### 5.2 DomainVerifier

**头文件**: `frameworks/verifier/include/domain_verifier.h`

**功能**: 域名校验器

```cpp
class DomainVerifier {
    VerifyResult Verify(const std::string& domain,
                       const AppVerifyBaseInfo& appInfo);
    bool ParseAssetLinks(const std::string& json,
                         std::vector<TargetInfo>& targets);
    bool VerifySignature(const std::string& fingerprint,
                        const std::vector<TargetInfo>& targets);
};
```

### 5.3 DomainJsonUtil

**头文件**: `frameworks/verifier/include/domain_json_util.h`

**功能**: JSON 解析工具

```cpp
class DomainJsonUtil {
    static bool ParseAssetLinks(const std::string& jsonStr,
                                std::vector<AssetEntry>& entries);
    static std::string SerializeVerifyResult(const VerifyResult& result);
};
```

## 6. RDB 数据管理

### 6.1 AppDomainVerifyRdbDataManager

**头文件**: `services/include/manager/rdb/app_domain_verify_rdb_data_manager.h`

**功能**: RDB 持久化管理

#### API 清单

| 方法 | 描述 |
|-----|------|
| `InsertVerifyStatus()` | 插入校验状态 |
| `UpdateVerifyStatus()` | 更新校验状态 |
| `DeleteVerifyStatus()` | 删除校验状态 |
| `QueryVerifyStatus()` | 查询校验状态 |
| `QueryAllVerifyStatus()` | 查询所有状态 |
| `CleanAll()` | 清除所有数据 |

## 7. 接口稳定性标注

### 7.1 稳定接口（System Internal）

| 接口 | 稳定性 | 调用方 |
|-----|-------|-------|
| `AppDomainVerifyMgrClient` | 稳定 | BundleManagerService |
| `AppDomainVerifyAgentClient` | 稳定 | Manager Service |
| `IAppDomainVerifyMgrService` | 稳定 | 系统服务 |
| `IAppDomainVerifyAgentService` | 稳定 | 系统服务 |

### 7.2 接口依赖方向

```
BundleManagerService
        │
        │ Inner API 调用
        ▼
┌───────────────────┐
│ AppDomainVerify   │
│ MgrClient         │
└─────────┬─────────┘
          │ IPC
          ▼
┌───────────────────┐
│ Manager Service   │ (SA 6200)
│ (常驻进程)         │
└─────────┬─────────┘
          │
          │ Inner API 调用
          ▼
┌───────────────────┐
│ AppDomainVerify   │
│ AgentClient       │
└─────────┬─────────┘
          │ IPC
          ▼
┌───────────────────┐
│ Agent Service     │ (SA 6201)
│ (按需启动)         │
└───────────────────┘
```

## 8. 相关文档

| 文档 | 链接 |
|-----|------|
| N-API 参考 | [03_N_API.md](./03_N_API.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| 安全风险评审 | [06_Security.md](./06_Security.md) |
