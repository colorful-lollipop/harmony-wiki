# 配置项与宏定义

> 本文档记录 app_domain_verify 部件的关键配置项、宏定义和 feature flags。

## 1. Feature Flags

### 1.1 部件特性开关

**文件**: `bundle.json`

```json
{
  "features": [
    "app_domain_verify_feature_target_from_cloud"
  ]
}
```

| 特性名称 | 默认值 | 说明 |
|---------|-------|------|
| `app_domain_verify_feature_target_from_cloud` | true | 是否支持从云端获取校验目标 |

**使用方式**:
```cpp
#if defined(APP_DOMAIN_VERIFY_FEATURE_TARGET_FROM_CLOUD)
    // 从云端获取校验目标
#else
    // 使用本地配置
#endif
```

---

## 2. GN 构建变量

### 2.1 模块路径变量

**文件**: `app_domain_verify.gni`

```gn
# 根路径
app_domain_verify_root_path = "//foundation/bundlemanager/app_domain_verify"

# 客户端路径
app_domain_verify_client_path = "//foundation/bundlemanager/app_domain_verify/interfaces/inner_api/client"

# 服务路径
app_domain_verify_service_path = "//foundation/bundlemanager/app_domain_verify/services"

# 公共接口路径
app_domain_verify_common_path = "//foundation/bundlemanager/app_domain_verify/interfaces/inner_api/common"

# 框架路径
app_domain_verify_frameworks_common_path = "//foundation/bundlemanager/app_domain_verify/frameworks/common"
app_domain_verify_frameworks_verifier_path = "//foundation/bundlemanager/app_domain_verify/frameworks/verifier"
app_domain_verify_frameworks_extension_path = "//foundation/bundlemanager/app_domain_verify/frameworks/extension"
app_domain_verify_frameworks_app_details_rdb_path = "//foundation/bundlemanager/app_domain_verify/frameworks/app_details_rdb"
```

### 2.2 编译器配置

**文件**: `services/BUILD.gn`

```gn
config("app_domain_verify_service_config") {
  # 包含路径
  include_dirs = [
    ".",
    "$app_domain_verify_service_path/include",
    "$app_domain_verify_service_path/include/agent",
    "$app_domain_verify_service_path/include/manager",
    "$app_domain_verify_frameworks_common_path/include",
  ]

  # 编译标志
  cflags = [
    "-fvisibility=hidden",      # 隐藏符号
    "-fstack-protector-strong",  # 栈保护
  ]

  # 发布版本优化
  if (!is_debug) {
    cflags += [ "-Os" ]          # 优化大小
  }
}
```

---

## 3. 错误码定义

### 3.1 通用错误码

**文件**: `interfaces/inner_api/common/include/comm_define.h`

| 错误码 | 值 | 描述 |
|-------|-----|------|
| `E_OK` | 0 | 成功 |
| `E_PERMISSION_DENIED` | 201 | 权限不足 |
| `E_IS_NOT_SYS_APP` | 202 | 非系统应用 |
| `E_PARAM_ERROR` | 401 | 参数错误 |
| `E_INTERNAL_ERR` | 29900001 | 内部错误 |

### 3.2 业务错误码

**文件**: `frameworks/common/include/app_domain_verify_error.h`

| 错误码 | 值 | 描述 |
|-------|-----|------|
| `E_OK` | 0 | 成功 |
| `E_EXTENSIONS_LIB_NOT_FOUND` | 10000001 | 扩展库未找到 |
| `E_EXTENSIONS_INTERNAL_ERROR` | 10000002 | 扩展内部错误 |
| `E_NET_CONNECT_ERR` | 29900002 | 网络连接错误 |
| `E_SERVER_ERR` | 29900003 | 服务器错误 |

### 3.3 校验状态码

**文件**: `interfaces/inner_api/common/include/inner_verify_status.h`

| 状态 | 值 | 描述 |
|-----|-----|------|
| `UNKNOWN` | 0 | 未知 |
| `STATE_SUCCESS` | 1 | 校验成功 |
| `STATE_FAIL` | 2 | 校验失败 |
| `FAILURE_REDIRECT` | 3 | 重定向失败 |
| `FAILURE_CLIENT_ERROR` | 4 | 客户端错误 |
| `FAILURE_REJECTED_BY_SERVER` | 5 | 服务器拒绝 |
| `FAILURE_HTTP_UNKNOWN` | 6 | 未知 HTTP 错误 |
| `FAILURE_TIMEOUT` | 7 | 超时 |
| `FAILURE_CONFIG` | 8 | 配置错误 |
| `FORBIDDEN_FOREVER` | 9 | 永久禁止 |

---

## 4. 常量定义

### 4.1 优先级常量

**文件**: `interfaces/inner_api/common/include/bundle_verify_status_info.h`

```cpp
constexpr int PRIORITY_UNSET = -1000;
constexpr int PRIORITY_MIN = -100;
constexpr int PRIORITY_MAX = 100;
```

### 4.2 Asset URL 路径

**文件**: `frameworks/verifier/include/constant/agent_constants.h`

```cpp
constexpr const char* ASSET_URL_PATH = "/.well-known/applinking.json";
```

### 4.3 JSON 键名

```cpp
constexpr const char* JSON_KEY_APPLINKING = "applinking";
constexpr const char* JSON_KEY_APPS = "apps";
constexpr const char* JSON_KEY_BUNDLE_NAME = "bundleName";
constexpr const char* JSON_KEY_APP_IDENTIFIER = "appIdentifier";
constexpr const char* JSON_KEY_FINGERPRINT = "fingerprint";
constexpr const char* JSON_KEY_INDEX = "index";
```

---

## 5. API 导出配置

### 5.1 版本脚本

**文件**: `interfaces/inner_api/client/mgr.versionscript`

```text
{
  global:
    *AppDomainVerifyMgrClient*;
    *AppDomainVerifyMgrServiceProxy*;
};
```

**文件**: `interfaces/inner_api/client/agent.versionscript`

```text
{
  global:
    *AppDomainVerifyAgentClient*;
    *AppDomainVerifyAgentServiceProxy*;
};
```

**文件**: `interfaces/inner_api/common/common.versionscript`

```text
{
  global:
    AppVerifyBaseInfo;
    SkillUri;
    VerifyResultInfo;
    UrlUtil;
    AppDomainVerifyTaskMgr;
    IHttpTask;
    BundleVerifyStatusInfo;
    BundleInfoQuery;
    ConvertCallbackStub;
    ConvertCallbackProxy;
    WhiteListChecker;
    WhiteListUpdater;
    WhiteListConfigMgr;
    MaskStr;
    TargetInfo;
};
```

---

## 6. 线程池配置

### 6.1 FFRT 线程池

**文件**: `frameworks/common/src/httpsession/app_domain_verify_task_mgr.cpp`

```cpp
constexpr int THREAD_POOL_SIZE = 4;        // 线程池大小
constexpr int MAX_TASK_QUEUE_SIZE = 1000; // 最大任务队列
```

---

## 7. 超时配置

### 7.1 HTTP 超时

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `HTTP_CONNECT_TIMEOUT` | 10s | 连接超时 |
| `HTTP_READ_TIMEOUT` | 60s | 读取超时 |
| `HTTP_WRITE_TIMEOUT` | 60s | 写入超时 |

### 7.2 重试配置

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `MAX_RETRY_COUNT` | 7 | 最大重试次数 |
| `INITIAL_BACKOFF` | 1h | 初始退避时间 |
| `MAX_BACKOFF` | 24h | 最大退避时间 |

---

## 8. 安全配置

### 8.1 网络安全

**强制 HTTPS**:
```cpp
// 只允许 HTTPS 请求
if (url.rfind("https://", 0) != 0) {
    return ERROR_INVALID_URL;
}
```

**证书固定** (可选):
```cpp
constexpr const char* PINNED_CERT_FINGERPRINT = "";
```

### 8.2 数据大小限制

```cpp
constexpr size_t MAX_RESPONSE_SIZE = 20 * 1024;  // 20KB
constexpr size_t MAX_URL_LENGTH = 512;           // 512 字符
constexpr size_t MAX_BUNDLE_NAME_LENGTH = 256;    // 256 字符
```

---

## 9. 数据库配置

### 9.1 RDB 配置

**文件**: `services/include/manager/rdb/app_domain_verify_rdb_config.h`

```cpp
constexpr const char* DATABASE_NAME = "app_domain_verify.db";
constexpr int DATABASE_VERSION = 1;
constexpr int32_t DB_OPS_TIMEOUT = 30;  // 秒
```

### 9.2 表名配置

```cpp
constexpr const char* TABLE_APP_VERIFY_STATUS = "app_verify_status";
constexpr const char* TABLE_DOMAIN_RESULT = "domain_verify_result";
constexpr const char* TABLE_BUNDLE_MAPPING = "bundle_domain_mapping";
constexpr const char* TABLE_DEFERRED_LINK = "deferred_link";
constexpr const char* TABLE_META_DATA = "meta_data";
```

---

## 10. 日志配置

### 10.1 Hilog 模块

**文件**: `frameworks/common/include/app_domain_verify_hilog.h`

```cpp
#define APP_DOMAIN_VERIFY_MGR_MODULE_SERVICE 0xD002B01
#define APP_DOMAIN_VERIFY_MGR_MODULE_CLIENT 0xD002B02
#define APP_DOMAIN_VERIFY_MGR_MODULE_AGENT 0xD002B03

#define APP_DOMAIN_VERIFY_HILOGI(module, format, ...)
#define APP_DOMAIN_VERIFY_HILOGE(module, format, ...)
#define APP_DOMAIN_VERIFY_HILOGD(module, format, ...)
```

### 10.2 Hisysevent 配置

**文件**: `hisysevent.yaml`

```yaml
domain_verify_success:
  type: STATISTIC
  action: ALARM
domain_verify_failed:
  type: STATISTIC
  action: ALARM
http_request_failed:
  type: SECURITY
  action: ALARM
```

---

## 11. 相关文档

| 文档 | 链接 |
|-----|------|
| GN 构建 | [04_GN_Build.md](../04_GN_Build.md) |
| 架构说明 | [01_Architecture.md](../01_Architecture.md) |
| 安全风险评审 | [06_Security.md](../06_Security.md) |
