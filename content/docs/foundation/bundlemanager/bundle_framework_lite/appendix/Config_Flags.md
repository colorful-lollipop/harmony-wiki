# 附录：配置开关

## 目的

本文档汇总 bundle_framework_lite 的所有配置开关（Feature Flags、宏定义、编译选项）。

## GN Feature 开关

### bundle_framework_lite.gni

**位置**: 根目录 `bundle_framework_lite.gni:26-31`

```gn
declare_args() {
  bundle_framework_lite_enable_ohos_bundle_manager_service = false
  bundle_framework_lite_enable_ohos_bundle_manager_service_permission = false
  bundle_framework_lite_enable_ohos_bundle_manager_service_parse_metadata = false
}
```

| 开关 | 默认值 | 说明 | 影响 |
|------|--------|------|------|
| `bundle_framework_lite_enable_ohos_bundle_manager_service` | false | 启用 BMS 服务 | 定义 `_MINI_BMS_` 宏 |
| `bundle_framework_lite_enable_ohos_bundle_manager_service_permission` | false | 启用权限管理 | 定义 `_MINI_BMS_PERMISSION_` 和 `BC_TRANS_ENABLE` |
| `bundle_framework_lite_enable_ohos_bundle_manager_service_parse_metadata` | false | 启用元数据解析 | 定义 `_MINI_BMS_PARSE_METADATA_` |

### 使用方式

**在 product 配置中启用**:
```gn
# 在 product 的 config.json 或 gn 文件中
bundle_framework_lite_enable_ohos_bundle_manager_service = true
bundle_framework_lite_enable_ohos_bundle_manager_service_permission = true
```

## C/C++ 宏定义

### 功能宏

| 宏 | 定义位置 | 说明 | 影响代码 |
|----|---------|------|----------|
| `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER` | BUILD.gn | 启用 BundleManager 功能 | 所有对外 API |
| `_MINI_BMS_` | BUILD.gn (条件) | LiteOS-M 版本 | `gt_*.cpp` 文件 |
| `_MINI_BMS_PERMISSION_` | BUILD.gn (条件) | 启用权限功能 | 权限相关代码 |
| `_MINI_BMS_PARSE_METADATA_` | BUILD.gn (条件) | 启用元数据解析 | 解析相关代码 |
| `BC_TRANS_ENABLE` | BUILD.gn (条件) | 启用权限转换 | 权限转换代码 |
| `JERRY_FOR_IAR_CONFIG` | BUILD.gn (条件) | IAR 编译器配置 | JerryScript 相关 |

### 调试宏

| 宏 | 定义位置 | 说明 | 影响 |
|----|---------|------|------|
| `OHOS_DEBUG` | 编译选项 | 调试模式 | 启用调试接口 |
| `HILOG_MODULE_APP` | bundle_log.h | 应用模块日志 | 日志输出 |
| `HILOG_MODULE_AAFWK` | bundle_log.h | AAFWK 模块日志 | 日志输出 |

## 代码中的条件编译

### 1. 调试模式代码

**证据**: `services/bundlemgr_lite/include/bundle_manager_service.h:60-63`

```cpp
#ifdef OHOS_DEBUG
    uint8_t SetSignMode(bool enable);
    bool IsSignMode() const;
#endif
```

**说明**: 仅在 `OHOS_DEBUG` 定义时可用，用于调试签名验证

### 2. 签名验证开关

**证据**: `services/bundlemgr_lite/src/bundle_installer.cpp:179-213`

```cpp
#ifdef OHOS_DEBUG
    if (ManagerService::GetInstance().IsSignMode()) {
        errorCode = HapSignVerify::VerifySignature(path, signatureInfo);
    }
#else
    errorCode = HapSignVerify::VerifySignature(path, signatureInfo);
#endif
```

**说明**: 
- Release 模式：强制签名验证
- Debug 模式：可通过 `IsSignMode()` 关闭

### 3. 系统能力 API

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:245-276`

```cpp
#ifdef OHOS_APPEXECFWK_BMS_BUNDLEMANAGER
bool HasSystemCapability(const char *sysCapName);
SystemCapability *GetSystemAvailableCapabilities();
void FreeSystemAvailableCapabilitiesInfo(SystemCapability *sysCap);
#endif
```

**说明**: 需要定义 `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER` 才能使用系统能力查询 API

### 4. LiteOS-M 特定代码

**证据**: `services/bundlemgr_lite/src/gt_bundle_installer.cpp:108-152`

```cpp
#ifdef _MINI_BMS_
    VerifyResult verifyResult;
    (void) APPVERI_SetDebugMode(true);
    int32_t ret = APPVERI_AppVerify(path, &verifyResult);
```

**说明**: `_MINI_BMS_` 模式下启用调试签名验证

## 编译选项

### 警告选项

**证据**: `services/bundlemgr_lite/BUILD.gn:109-114`

```gn
cflags = [
  "-Wall",
  "-Wno-format",
  "-Wno-format-extra-args",
]
```

### 代码标准

**证据**: `services/bundlemgr_lite/BUILD.gn:17-20`

```gn
config("bundle_config") {
  defines = [ "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER" ]
  cflags_cc = [ "-std=c++14" ]
}
```

### 位置无关代码

**证据**: `frameworks/bundle_lite/BUILD.gn:125-131`

```gn
if (board_toolchain_type != "iccarm") {
  cflags = [
    "-fPIC",
    "-Wall",
    "-Wno-format",
  ]
}
```

## 常量配置

### 路径常量

**证据**: `services/bundlemgr_lite/include/bundle_common.h:62-71`

```cpp
const char INSTALL_PATH[] = "/data/app";
const char DATA_PATH[] = "/data/data";
const char SYSTEM_BUNDLE_PATH[] = "/system/app";
const char THIRD_SYSTEM_BUNDLE_PATH[] = "/system/vendor";
const char EXTEANAL_INSTALL_PATH[] = "/sdcard/app";
const char EXTEANAL_DATA_PATH[] = "/sdcard/data";
const char JSON_PATH[] = "/data/accounts/account_0/applications/";
const char SHARED_LIB_PATH[] = "/data/shared_lib";
```

### UID/GID 常量

**证据**: `services/bundlemgr_lite/include/bundle_common.h:54-60`

```cpp
const uint32_t BASE_SYS_UID = 100;
const uint32_t BASE_SYS_VEN_UID = 1000;
const uint32_t MAX_SYS_VEN_UID = 9999;
const uint32_t BASE_APP_UID = 10000;
const int8_t INVALID_UID = -1;
const int8_t INVALID_GID = -1;
```

### 长度限制

**证据**: `services/bundlemgr_lite/include/bundle_common.h:35-43`

```cpp
const int32_t MAX_BUNDLE_NAME_LEN = 128;
const int32_t MIN_BUNDLE_NAME_LEN = 5;
const int32_t MAX_SYSCAP_NAME_LEN = 64;
const uint32_t PATH_LENGTH = 256;
const uint32_t MAX_IO_SIZE = 8192;
```

## 配置组合建议

### 标准配置（生产环境）

```gn
# 启用 BMS 和权限管理
bundle_framework_lite_enable_ohos_bundle_manager_service = true
bundle_framework_lite_enable_ohos_bundle_manager_service_permission = true

# 不定义 OHOS_DEBUG
# 强制签名验证
```

### 开发配置

```gn
# 启用所有功能
bundle_framework_lite_enable_ohos_bundle_manager_service = true
bundle_framework_lite_enable_ohos_bundle_manager_service_permission = true
bundle_framework_lite_enable_ohos_bundle_manager_service_parse_metadata = true

# 定义 OHOS_DEBUG 以启用调试接口
```

### 最小配置（仅查询）

```gn
# 仅启用基本 BMS
bundle_framework_lite_enable_ohos_bundle_manager_service = true

# 不启用权限管理（如果系统不需要）
bundle_framework_lite_enable_ohos_bundle_manager_service_permission = false
```

## 配置检查清单

发布前请确认：

- [ ] `OHOS_DEBUG` 未定义（生产环境）
- [ ] `bundle_framework_lite_enable_ohos_bundle_manager_service` 已启用
- [ ] 签名验证无法被关闭
- [ ] 路径常量符合目标系统要求
- [ ] UID/GID 范围不与系统冲突

---

**相关链接**:
- [GN 构建目标](../05_GN_Targets.md)
- [编译产物](../06_Build_Artifacts.md)
- [安全风险分析](../07_Security_Analysis.md)
