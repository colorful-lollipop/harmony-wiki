# 附录：配置开关

## 目的

本文档列出 Data Share 中的关键配置项、宏定义和 Feature Flags。

## 编译时配置

### 安全加固选项

**文件**: `interfaces/inner_api/BUILD.gn`

```gn
# 所有共享库都启用以下安全选项
branch_protector_ret = "pac_ret"
sanitize = {
    ubsan = true              # 未定义行为检测 (UndefinedBehaviorSanitizer)
    boundary_sanitize = true  # 边界检查
    cfi = true                # 控制流完整性 (Control Flow Integrity)
    cfi_cross_dso = true      # 跨 DSO CFI
    debug = false
}
```

**作用**: 防止内存损坏、控制流劫持等攻击

### ARM 32位配置

```gn
if (target_cpu == "arm") {
    cflags += [ "-DBINDER_IPC_32BIT" ]
}
```

**文件**: `interfaces/inner_api/BUILD.gn:29-31`

**作用**: ARM 32位系统使用 32位 Binder IPC

## 运行时配置

### URI 信任列表

**文件**: `/system/etc/distributeddata/conf/config.json`

```json
{
    "uriTrusts": [
        "datashare://com.example.app/",
        "datashareproxy://com.example.provider/"
    ]
}
```

**代码引用**:
```cpp
// frameworks/native/permission/src/data_share_permission.cpp:93-109
bool IsInUriTrusts(Uri &uri) {
    auto config = ConfigFactory::GetInstance().GetDataShareConfig();
    std::string uriStr = uri.ToString();
    for (std::string& item : config->uriTrusts) {
        if (item.length() > uriStr.length() ||
            uriStr.compare(0, item.length(), item) != 0) {
            continue;
        }
        return true;
    }
    return false;
}
```

**作用**: 配置可信的 URI 前缀，用于 URI 安全校验

### 缓存大小配置

**文件**: `interfaces/inner_api/permission/include/data_share_permission.h`

```cpp
static constexpr int32_t CACHE_SIZE = 32;  // Line 91
```

**代码引用**:
```cpp
// frameworks/native/permission/src/data_share_permission.cpp:211-214
if (silentCache_.size() >= CACHE_SIZE) {
    LOG_INFO("silentCache_ full, clear all");
    silentCache_.clear();
}
```

**作用**: Silent 访问权限缓存大小限制

### Provider 白名单

**文件**: `frameworks/native/provider/src/datashare_stub_impl.cpp:40-45`

```cpp
const std::set<std::string> PROVIDER_LIST = {
    "5765880207853551549",
    "5765880207853570539",
    "5765880207853771197",
    "5765880207854616753"
};
```

**作用**: 硬编码的 Provider 签名白名单，用于 `VerifyProvider()` 检查

## Scheme 常量

**文件**: `interfaces/inner_api/permission/include/data_share_permission.h:116-120`

```cpp
static constexpr const char *SCHEME_DATASHARE = "datashare";
static constexpr const char *SCHEME_DATASHARE_PROXY = "datashareproxy";
static constexpr const char *SCHEME_PREFERENCE = "sharepreferences";
static constexpr const char *SCHEME_RDB = "rdb";
static constexpr const char *SCHEME_FILE = "file";
```

**作用**: 支持的 URI Scheme 列表

## 线程池配置

**文件**: `frameworks/native/consumer/controller/service/src/general_controller_service_impl.cpp:33`

```cpp
static constexpr int MAX_THREADS = 2;
```

**作用**: Silent 模式查询线程池最大线程数

## 日志标签

**代码中使用的日志标签**:

| 标签 | 文件 | 说明 |
|------|------|------|
| `native_datashare_module` | `native_datashare_module.cpp:16` | NAPI 模块日志 |
| `napi_datashare_helper` | `napi_datashare_helper.cpp:16` | Helper NAPI 日志 |
| `datashare_stub_impl` | `datashare_stub_impl.cpp` | Provider 端日志 |
| `data_share_permission` | `data_share_permission.cpp` | 权限模块日志 |

## 错误码基准值

**文件**: `interfaces/inner_api/common/include/datashare_errno.h:35`

```cpp
constexpr int E_BASE = 1000;
```

**作用**: 所有错误码的基准值，实际错误码 = E_BASE + 偏移量

## 权限常量

**文件**: `interfaces/inner_api/permission/include/data_share_permission.h:72`

```cpp
static constexpr const char *NO_PERMISSION = "noPermission";
```

**作用**: 表示"不需要权限"的特殊字符串

## 系统 UID 阈值

**文件**: `frameworks/native/permission/src/data_share_called_config.cpp:46`

```cpp
static constexpr int SYSTEM_UID = 10000;
```

**作用**: 区分系统应用和普通应用的 UID 阈值

## Feature Flags（未确认）

以下配置可能存在，但需要进一步确认：

| 配置项 | 可能位置 | 说明 |
|--------|----------|------|
| `ENABLE_SILENT_ACCESS` | TODO | 是否启用 Silent 访问模式 |
| `ENABLE_BATCH_OPERATIONS` | TODO | 是否启用批量操作优化 |
| `ENABLE_SHARED_MEMORY` | TODO | 是否启用共享内存传输 |

**注意**: 以上配置项在当前代码分析中未找到明确的 Feature Flag，可能是编译时硬编码或需要进一步确认。

## 配置修改建议

### 增加配置项

建议将以下硬编码配置改为可配置：

1. **Provider 白名单** - 从代码迁移到配置文件
2. **缓存大小** - 支持运行时调整
3. **线程池大小** - 根据设备性能动态调整

### 配置文件示例

```json
// /system/etc/distributeddata/conf/datashare_config.json
{
    "security": {
        "providerWhitelist": [
            "5765880207853551549",
            "5765880207853570539"
        ],
        "uriTrusts": [
            "datashare://com.example/"
        ]
    },
    "performance": {
        "cacheSize": 32,
        "maxThreadPoolSize": 2
    },
    "features": {
        "enableSilentAccess": true,
        "enableSharedMemory": true
    }
}
```

## 相关文档

- [构建系统](../07_Build_System.md) - 编译配置详情
- [安全分析](../08_Security_Analysis.md) - 安全配置影响
