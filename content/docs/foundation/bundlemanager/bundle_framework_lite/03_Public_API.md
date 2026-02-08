# 对外 API

## 目的

本文档详细说明 bundle_framework_lite 提供的对外 API，包括 C API 和 JS API，帮助开发者正确使用包管理功能。

## API 概览

| API 类型 | 文件路径 | 说明 |
|---------|----------|------|
| C API | `interfaces/kits/bundle_lite/bundle_manager.h` | C/C++ 应用接口 |
| JS API | `interfaces/kits/bundle_lite/js/builtin/capability_module.cpp` | JavaScript 接口 |

## C API 详细说明

### 头文件

**位置**: `interfaces/kits/bundle_lite/bundle_manager.h`

```cpp
#include "bundle_manager.h"
```

### 回调函数类型

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:72`

```cpp
typedef void (*InstallerCallback)(const uint8_t resultCode, const void *resultMessage);
```

**说明**: 安装/卸载操作的回调函数类型

| 参数 | 类型 | 说明 |
|------|------|------|
| resultCode | uint8_t | 结果码，参见 AppexecfwkErrors |
| resultMessage | const void* | 结果消息（可能为 NULL） |

### 1. 回调注册接口

#### RegisterCallback

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:85`

```cpp
int32_t RegisterCallback(BundleStatusCallback *BundleStatusCallback);
```

**功能**: 注册应用状态变化回调

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| BundleStatusCallback | BundleStatusCallback* | 回调结构体指针 |

**返回值**:
- `ERR_OK` (0): 成功
- 其他: 错误码（参见 `appexecfwk_errors.h`）

#### UnregisterCallback

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:97`

```cpp
int32_t UnregisterCallback(void);
```

**功能**: 注销状态变化回调

### 2. 安装/卸载接口

#### Install

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:112`

```cpp
bool Install(const char *hapPath, const InstallParam *installParam, InstallerCallback installerCallback);
```

**功能**: 安装或更新应用

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| hapPath | const char* | HAP 包路径（绝对路径） |
| installParam | InstallParam* | 安装参数 |
| installerCallback | InstallerCallback | 安装结果回调 |

**InstallParam 结构** (`install_param.h:24-27`):
```cpp
typedef struct InstallParam {
    int32_t installLocation;  // 安装位置：0=内部，1=外部优先
    bool keepData;           // 卸载时是否保留数据
} InstallParam;
```

**返回值**:
- `true`: 调用成功（注意：不代表安装成功，结果通过回调返回）
- `false`: 调用失败

**调用链**:
```
Install()
  └── frameworks/bundle_lite/src/bundle_manager.cpp:Install()
       └── IPC 到 BMS
            └── services/bundlemgr_lite/src/bundle_installer.cpp:Install()
```

#### Uninstall

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:125`

```cpp
bool Uninstall(const char *bundleName, const InstallParam *installParam, InstallerCallback installerCallback);
```

**功能**: 卸载应用

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| bundleName | const char* | 应用包名 |
| installParam | InstallParam* | 卸载参数（keepData 字段有效） |
| installerCallback | InstallerCallback | 卸载结果回调 |

**限制**: 系统应用（isSystemApp=true）不可卸载

### 3. Ability 查询接口

#### QueryAbilityInfo

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:142`

```cpp
uint8_t QueryAbilityInfo(const Want *want, AbilityInfo *abilityInfo);
```

**功能**: 根据 Want 查询 Ability 信息

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| want | Want* | 查询条件（包含 bundleName, abilityName 等） |
| abilityInfo | AbilityInfo* | 输出参数，Ability 信息 |

**注意**: 调用前需使用 `memset` 初始化 abilityInfo

**返回值**: 错误码（参见 `appexecfwk_errors.h`）

#### QueryAbilityInfos

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:152`

```cpp
uint8_t QueryAbilityInfos(const Want *want, AbilityInfo **abilityInfo, int32_t *len);
```

**功能**: 查询符合条件的所有 Ability

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| want | Want* | 查询条件 |
| abilityInfo | AbilityInfo** | 输出参数，Ability 数组 |
| len | int32_t* | 输出参数，数组长度 |

**注意**: 使用完后需调用 `BundleInfoUtils::FreeAbilityInfos` 释放内存

### 4. Bundle 信息查询接口

#### GetBundleInfo

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:187`

```cpp
uint8_t GetBundleInfo(const char *bundleName, int32_t flags, BundleInfo *bundleInfo);
```

**功能**: 获取指定应用的 BundleInfo

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| bundleName | const char* | 应用包名 |
| flags | int32_t | 标志：1=包含 AbilityInfo，0=不包含 |
| bundleInfo | BundleInfo* | 输出参数，Bundle 信息 |

**注意**: 调用前需使用 `memset` 初始化 bundleInfo

#### GetBundleInfos

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:203`

```cpp
uint8_t GetBundleInfos(const int flags, BundleInfo **bundleInfos, int32_t *len);
```

**功能**: 获取所有应用的 BundleInfo

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| flags | int | 标志：1=包含 AbilityInfo，0=不包含 |
| bundleInfos | BundleInfo** | 输出参数，BundleInfo 数组 |
| len | int32_t* | 输出参数，数组长度 |

#### QueryKeepAliveBundleInfos

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:216`

```cpp
uint8_t QueryKeepAliveBundleInfos(BundleInfo **bundleInfos, int32_t *len);
```

**功能**: 获取所有保活应用的 BundleInfo

#### GetBundleInfosByMetaData

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:230`

```cpp
uint8_t GetBundleInfosByMetaData(const char *metaDataKey, BundleInfo **bundleInfos, int32_t *len);
```

**功能**: 根据 MetaData 查询 BundleInfo

### 5. UID 相关接口

#### GetBundleNameForUid

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:243`

```cpp
uint8_t GetBundleNameForUid(int32_t uid, char **bundleName);
```

**功能**: 根据 UID 获取应用包名

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| uid | int32_t | 应用 UID |
| bundleName | char** | 输出参数，包名（需调用者释放） |

### 6. 系统能力接口

#### HasSystemCapability

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:255`

```cpp
bool HasSystemCapability(const char *sysCapName);
```

**功能**: 检查系统是否支持指定的系统能力

**条件**: 需要定义 `OHOS_APPEXECFWK_BMS_BUNDLEMANAGER` 宏

#### GetSystemAvailableCapabilities

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:265`

```cpp
SystemCapability *GetSystemAvailableCapabilities();
```

**功能**: 获取系统所有可用的系统能力

#### FreeSystemAvailableCapabilitiesInfo

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:275`

```cpp
void FreeSystemAvailableCapabilitiesInfo(SystemCapability *sysCap);
```

**功能**: 释放系统能力信息内存

### 7. 其他接口

#### GetBundleSize

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:286`

```cpp
uint32_t GetBundleSize(const char *bundleName);
```

**功能**: 获取应用大小（字节）

**返回值**: 应用大小，0 表示失败

#### RegisterEvent / UnregisterEvent

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:160-168`

```cpp
bool RegisterEvent(InstallerCallback installerCallback);
bool UnregisterEvent(InstallerCallback installerCallback);
```

**功能**: 注册/注销事件回调（用于 HCE 标签等）

## JS API 详细说明

### 模块位置

**文件**: `interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp`

### 接口列表

| 方法名 | 命名空间 | 参数 | 返回值 | 说明 |
|--------|----------|------|--------|------|
| has | capability | sysCapName: string | boolean | 检查系统能力 |

### has 接口

**证据**: `interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp:20-32`

```cpp
JSIValue CapabilityModule::HasCapability(const JSIValue thisVal, const JSIValue *args, uint8_t argsSize)
{
    if (argsSize < 1) {
        return JSI::CreateBoolean(false);
    }
    char *str = JSI::ValueToString(args[0]);
    if (str == nullptr) {
        return JSI::CreateBoolean(false);
    }
    bool hasCap = HasSystemCapability(str);
    ace_free(str);
    return JSI::CreateBoolean(hasCap);
}
```

**JS 使用示例**:
```javascript
import capability from '@system.capability';

// 检查系统是否支持某个能力
const hasFeature = capability.has('SystemCapability.BundleManager.BundleFramework');
console.log('Has feature:', hasFeature);
```

**参数校验**:
- 检查参数数量（argsSize >= 1）
- 检查参数转换结果（非 NULL）

**调用链**:
```
JS: capability.has()
  └── capability_module.cpp:HasCapability()
       └── bundle_manager.h:HasSystemCapability()
            └── IPC 到 BMS
                 └── ManagerService::HasSystemCapability()
                      └── SAMGR::HasSystemCapability()
```

## 数据结构

### AbilityInfo

**证据**: `interfaces/kits/bundle_lite/ability_info.h:46-67`

```cpp
typedef struct AbilityInfo {
    char *bundleName;           // 包名
    char *name;                 // Ability 名
    char *srcPath;              // 源码路径
    char *iconPath;             // 图标路径
    char *label;                // 标签
    char *description;          // 描述
    uint8_t iconId;             // 图标 ID
    uint8_t labelId;            // 标签 ID
    uint8_t descriptionId;      // 描述 ID
    uint8_t type;               // 类型：0=Page，1=Service
    bool isVisible;             // 是否可见
    char **permissions;         // 权限数组
    uint8_t permissionsSize;    // 权限数量
} AbilityInfo;
```

### BundleInfo

**证据**: `interfaces/kits/bundle_lite/bundle_info.h:29-48`

```cpp
typedef struct BundleInfo {
    char *bundleName;           // 包名
    char *appId;                // 应用 ID
    char *versionName;          // 版本名
    uint32_t versionCode;       // 版本号
    char *iconPath;             // 图标路径
    char *bigIconPath;          // 大图标路径
    char *smallIconPath;        // 小图标路径
    char *label;                // 标签
    char *description;          // 描述
    char *codePath;             // 代码路径
    char *dataPath;             // 数据路径
    char *moduleName;           // 模块名
    AbilityInfo *abilityInfos;  // Ability 数组
    uint8_t abilityInfosSize;   // Ability 数量
    ModuleInfo *moduleInfos;    // 模块数组
    uint8_t moduleInfosSize;    // 模块数量
    bool isSystemApp;           // 是否系统应用
    int32_t uid;                // UID
    int32_t gid;                // GID
} BundleInfo;
```

### ModuleInfo

**证据**: `interfaces/kits/bundle_lite/module_info.h:29-42`

```cpp
typedef struct ModuleInfo {
    char *moduleName;           // 模块名
    char *name;                 // 名
    char *description;          // 描述
    char **deviceType;          // 设备类型数组
    uint8_t deviceTypeSize;     // 设备类型数量
    uint8_t moduleType;         // 模块类型
    bool isDeliveryInstall;     // 是否延迟安装
    char **metaData;            // 元数据数组
    uint8_t metaDataSize;       // 元数据数量
} ModuleInfo;
```

## 错误码

**位置**: `interfaces/kits/bundle_lite/appexecfwk_errors.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| ERR_OK | 0 | 成功 |
| ERR_APPEXECFWK_OBJECT_NULL | 1 | 对象为空 |
| ERR_APPEXECFWK_INSTALL_FAILED_PARAM_ERROR | 0x41 | 安装参数错误 |
| ERR_APPEXECFWK_INSTALL_FAILED_FILE_PATH_INVALID | 0x42 | 文件路径无效 |
| ERR_APPEXECFWK_INSTALL_FAILED_INVALID_FILE_NAME | 0x43 | 文件名无效 |
| ERR_APPEXECFWK_INSTALL_FAILED_FILE_NOT_EXISTS | 0x44 | 文件不存在 |
| ERR_APPEXECFWK_INSTALL_FAILED_BAD_FILE | 0x45 | 文件损坏 |
| ERR_APPEXECFWK_INSTALL_FAILED_SIGNATURE_VERIFICATION | 0x46 | 签名验证失败 |
| ERR_APPEXECFWK_INSTALL_FAILED_VERSION_DOWNGRADE | 0x47 | 版本降级 |
| ERR_APPEXECFWK_INSTALL_FAILED_INCOMPATIBLE_SIGNATURE | 0x48 | 签名不兼容 |
| ERR_APPEXECFWK_UNINSTALL_FAILED_PARAM_ERROR | 0x61 | 卸载参数错误 |
| ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_EXISTS | 0x62 | 应用不存在 |
| ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_UNINSTALLABLE | 0x63 | 应用不可卸载 |
| ERR_APPEXECFWK_QUERY_PARAMETER_ERROR | 0x81 | 查询参数错误 |
| ERR_APPEXECFWK_PERMISSION_DENIED | 0x91 | 权限拒绝 |

## 使用示例

### C++ 示例：安装应用

```cpp
#include "bundle_manager.h"
#include <iostream>

void InstallCallback(uint8_t resultCode, const void *resultMessage) {
    if (resultCode == ERR_OK) {
        std::cout << "Install success!" << std::endl;
    } else {
        std::cout << "Install failed, code: " << (int)resultCode << std::endl;
    }
}

int main() {
    const char *hapPath = "/data/app/example.hap";
    InstallParam param = {
        .installLocation = 0,  // 内部存储
        .keepData = false
    };
    
    bool result = Install(hapPath, &param, InstallCallback);
    if (!result) {
        std::cout << "Install call failed!" << std::endl;
        return -1;
    }
    
    // 等待回调...
    return 0;
}
```

### C++ 示例：查询 Bundle 信息

```cpp
#include "bundle_manager.h"
#include <iostream>
#include <cstring>

int main() {
    const char *bundleName = "com.example.app";
    BundleInfo info;
    memset(&info, 0, sizeof(BundleInfo));
    
    uint8_t ret = GetBundleInfo(bundleName, 1, &info);
    if (ret != ERR_OK) {
        std::cout << "GetBundleInfo failed, code: " << (int)ret << std::endl;
        return -1;
    }
    
    std::cout << "Bundle name: " << info.bundleName << std::endl;
    std::cout << "Version: " << info.versionName << std::endl;
    std::cout << "UID: " << info.uid << std::endl;
    
    // 释放资源
    BundleInfoUtils::FreeBundleInfo(&info);
    return 0;
}
```

---

**相关链接**:
- [架构说明](02_Architecture.md)
- [内部 API](04_Internal_API.md)
- [安全风险分析](07_Security_Analysis.md)
