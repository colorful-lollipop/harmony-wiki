# 对外 API

## 目的

本文档详细说明 `sys_installer` 对外暴露的 C++ API 接口，包括 SysInstallerKits 和 ModuleUpdateKits。

## 适用范围

需要调用 sys_installer 能力的系统组件和应用程序。

## 重要说明

**sys_installer 是纯 Native 服务，不直接暴露 JavaScript/ArkTS API。**

如需在 JS/ArkTS 中使用，需要通过其他组件（如 `@ohos.update`）封装后间接调用。

## API 概览

| Kit 名称 | 功能 | 对应 SA | 头文件 |
|----------|------|---------|--------|
| SysInstallerKits | 系统包更新、VAB、云 ROM | SA 4101 | `sys_installer_kits.h` |
| ModuleUpdateKits | 模块更新、HMP 管理 | SA 4103 | `module_update_kits.h` |

## SysInstallerKits API

### 头文件

```cpp
#include "sys_installer_kits.h"
```

**路径**: `interfaces/innerkits/ipc_client/include/sys_installer_kits.h`

**实现**: `interfaces/innerkits/ipc_client/src/sys_installer_kits_impl.cpp` (662 行)

### 单例获取

```cpp
// 获取 Kit 实例
SysInstallerKits &GetInstance();
```

### 初始化与退出

| 方法 | 签名 | 说明 |
|------|------|------|
| SysInstallerInit | `int32_t SysInstallerInit(const std::string &taskId, bool bStreamUpgrade)` | 初始化安装器 |
| ExitSysInstaller | `int32_t ExitSysInstaller()` | 退出安装器 |

### 包更新接口

| 方法 | 签名 | 说明 |
|------|------|------|
| StartUpdatePackageZip | `int32_t StartUpdatePackageZip(const std::string &taskId, const std::string &pkgPath)` | 开始 ZIP 包更新 |
| StartUpdateParaZip | `int32_t StartUpdateParaZip(...)` | 更新参数 ZIP |
| StartDeleteParaZip | `int32_t StartDeleteParaZip(...)` | 删除参数 ZIP |
| CancelUpdateVabPackageZip | `int32_t CancelUpdateVabPackageZip(const std::string &taskId)` | 取消 VAB 更新 |
| StartUpdateVabPackageZip | `int32_t StartUpdateVabPackageZip(const std::string &taskId, const std::vector<std::string> &pkgPath)` | 开始 VAB 更新 |
| StartUpdateSingularPackageZip | `int32_t StartUpdateSingularPackageZip(...)` | 开始单包更新 |

### VAB 相关接口

| 方法 | 签名 | 说明 |
|------|------|------|
| CreateVabSnapshotCowImg | `int32_t CreateVabSnapshotCowImg(const unsigned int &cowSize, std::vector<std::string> &pkgPath)` | 创建 VAB COW 快照 |
| GetPartitionAvailableSize | `int32_t GetPartitionAvailableSize(const std::string &partName, uint64_t &size)` | 获取分区可用空间 |
| StartVabMerge | `int32_t StartVabMerge(const std::string &taskId)` | 开始 VAB 合并 |
| ClearVabMetadataAndCow | `int32_t ClearVabMetadataAndCow()` | 清除 VAB 元数据 |
| ClearVabPatch | `int32_t ClearVabPatch()` | 清除 VAB 补丁 |
| VabUpdateActive | `int32_t VabUpdateActive(VabActiveMode mode)` | 激活 VAB 更新 |

### 流式更新接口

| 方法 | 签名 | 说明 |
|------|------|------|
| StartStreamUpdate | `int32_t StartStreamUpdate()` | 开始流式更新 |
| StopStreamUpdate | `int32_t StopStreamUpdate()` | 停止流式更新 |
| ProcessStreamData | `int32_t ProcessStreamData(const uint8_t *buffer, uint32_t size)` | 处理流数据 |

### 云 ROM 接口

| 方法 | 签名 | 说明 |
|------|------|------|
| InstallCloudRom | `int32_t InstallCloudRom(const std::string &taskId, const std::string &pkgPath)` | 安装云 ROM |
| UninstallCloudRom | `int32_t UninstallCloudRom(const std::string &taskId, const std::string &pkgPath)` | 卸载云 ROM |
| ClearCloudRom | `int32_t ClearCloudRom(const std::string &taskId, const std::string &pkgPath)` | 清除云 ROM |
| UpdateCloudRomVersion | `int32_t UpdateCloudRomVersion(const std::string &taskId, const std::string &version)` | 更新云 ROM 版本 |

### 其他接口

| 方法 | 签名 | 说明 |
|------|------|------|
| SetUpdateCallback | `int32_t SetUpdateCallback(const std::string &taskId, const sptr<ISysInstallerCallback> &cb)` | 设置更新回调 |
| GetUpdateStatus | `int32_t GetUpdateStatus(const std::string &taskId)` | 获取更新状态 |
| GetUpdateResult | `std::string GetUpdateResult(const std::string &taskId, const std::string &taskType, const std::string &resultType)` | 获取更新结果 |
| AccDecompressAndVerifyPkg | `int32_t AccDecompressAndVerifyPkg(...)` | 解压并验证包 |
| AccDeleteDir | `int32_t AccDeleteDir(const std::string &taskId, const std::string &dstPath)` | 删除目录 |
| StartAbSync | `int32_t StartAbSync()` | 开始 AB 同步 |
| SetCpuAffinity | `int32_t SetCpuAffinity(const std::string &taskId, unsigned int reservedCores)` | 设置 CPU 亲和性 |
| GetFeatureStatus | `int32_t GetFeatureStatus(...)` | 获取特性状态 |
| GetAllFeatureStatus | `int32_t GetAllFeatureStatus(...)` | 获取所有特性状态 |
| GetMetadataResult | `int32_t GetMetadataResult(const std::string &action, bool &result)` | 获取元数据结果 |

## ModuleUpdateKits API

### 头文件

```cpp
#include "module_update_kits.h"
```

**路径**: `interfaces/innerkits/ipc_client/include/module_update_kits.h`

**实现**: `interfaces/innerkits/ipc_client/src/module_update_kits_impl.cpp` (255 行)

### 单例获取

```cpp
// 获取 Kit 实例
ModuleUpdateKits &GetInstance();
```

### 初始化与退出

| 方法 | 签名 | 说明 |
|------|------|------|
| InitModuleUpdate | `int32_t InitModuleUpdate()` | 初始化模块更新 |
| ExitModuleUpdate | `int32_t ExitModuleUpdate()` | 退出模块更新 |

### 模块包管理

| 方法 | 签名 | 说明 |
|------|------|------|
| InstallModulePackage | `int32_t InstallModulePackage(const std::string &pkgPath)` | 安装模块包 |
| UninstallModulePackage | `int32_t UninstallModulePackage(const std::string &hmpName)` | 卸载模块包 |
| GetModulePackageInfo | `int32_t GetModulePackageInfo(const std::string &hmpName, std::list<ModulePackageInfo> &modulePackageInfos)` | 获取模块包信息 |

### HMP 管理

| 方法 | 签名 | 说明 |
|------|------|------|
| GetHmpVersionInfo | `std::vector<HmpVersionInfo> GetHmpVersionInfo()` | 获取 HMP 版本信息 |
| StartUpdateHmpPackage | `int32_t StartUpdateHmpPackage(const std::string &path, sptr<ISysInstallerCallbackFunc> callback)` | 开始 HMP 更新 |
| GetHmpUpdateResult | `std::vector<HmpUpdateInfo> GetHmpUpdateResult()` | 获取 HMP 更新结果 |

## 回调接口

### ISysInstallerCallback

**文件**: `interfaces/innerkits/ipc_client/ISysInstallerCallback.idl`

```cpp
interface ISysInstallerCallback {
    void OnUpgradeProgress(int32_t status, int32_t percent);
};
```

**回调状态码**:

| 状态 | 值 | 说明 |
|------|-----|------|
| UPDATE_STATE_SUCCESS | 0 | 更新成功 |
| UPDATE_STATE_FAIL | 1 | 更新失败 |
| UPDATE_STATE_ONGOING | 2 | 更新进行中 |

### ISysInstallerCallbackFunc

**文件**: `interfaces/inner_api/include/isys_installer_callback_func.h`

```cpp
class ISysInstallerCallbackFunc {
public:
    virtual void OnUpgradeProgress(UpdateStatus status, int32_t percent, 
                                   const std::string &resultMsg) = 0;
    virtual void OnUpgradeHmpProgress(const std::vector<HmpUpdateInfo> &hmpUpdateInfos) = 0;
    virtual void OnFeatureStateChanged(const std::vector<FeatureStatus> &features) = 0;
};
```

## 数据类型

### UpdateStatus (更新状态)

**文件**: `interfaces/innerkits/ipc_client/Types.idl`

```cpp
enum UpdateStatus {
    UPDATE_STATE_SUCCESS = 0,
    UPDATE_STATE_FAIL = 1,
    UPDATE_STATE_ONGOING = 2,
    UPDATE_STATE_SUB_SUCCESS = 3,
    UPDATE_STATE_SUB_FAIL = 4,
    UPDATE_STATE_ALREADY_UP_TO_DATE = 5
};
```

### ModulePackageInfo (模块包信息)

```cpp
struct ModulePackageInfo {
    std::string hmpName;           // HMP 名称
    std::string version;           // 版本号
    std::string path;              // 安装路径
    // ... 其他字段
};
```

### HmpVersionInfo (HMP 版本信息)

```cpp
struct HmpVersionInfo {
    std::string hmpName;           // HMP 名称
    std::string version;           // 当前版本
    std::string path;              // 安装路径
    // ... 其他字段
};
```

### HmpUpdateInfo (HMP 更新信息)

```cpp
struct HmpUpdateInfo {
    std::string hmpName;           // HMP 名称
    int32_t result;                // 更新结果
    std::string message;           // 结果消息
    // ... 其他字段
};
```

## 错误码

### 系统错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| ERR_OK | 0 | 成功 |
| ERR_INVALID_VALUE | -1 | 无效参数 |
| ERR_INVALID_OPERATION | -2 | 无效操作 |
| ERR_PERMISSION_DENIED | -3 | 权限拒绝 |
| ERR_NOT_SUPPORTED | -4 | 不支持的操作 |

### 模块更新错误码

**文件**: `services/module_update/util/include/module_error_code.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| ERR_OK | 0 | 成功 |
| ERR_VERIFY_FAIL | 1 | 验证失败 |
| ERR_EXTRACT_FAIL | 2 | 解压失败 |
| ERR_MOUNT_FAIL | 3 | 挂载失败 |
| ERR_DM_CREATE_FAIL | 4 | Device Mapper 创建失败 |
| ERR_LOOP_CREATE_FAIL | 5 | Loop 设备创建失败 |
| ERR_FILE_OPERATION_FAIL | 6 | 文件操作失败 |
| ERR_HVB_VERIFY_FAIL | 7 | HVB 验证失败 |
| ERR_INVALID_PACKAGE | 8 | 无效包 |
| ERR_NO_SPACE | 9 | 空间不足 |

## 权限要求

### 必需权限

调用 sys_installer API 需要以下权限：

```cpp
// 权限名称
const std::string PERMISSION_UPDATE_SYSTEM = "ohos.permission.UPDATE_SYSTEM";
```

### UID 限制

```cpp
// 允许的 UID
const int32_t USER_UPDATE_AUTHORITY = 6666;  // Update 服务 UID
const int32_t ROOT_UID = 0;                  // Root UID
```

**权限检查逻辑** (来自 `frameworks/ipc_server/src/sys_installer_server.cpp:329-350`):

```cpp
bool CheckCallingPerm() {
    int32_t callingUid = OHOS::IPCSkeleton::GetCallingUid();
    if (callingUid == 0) {
        return true;  // Root 绕过检查
    }
    return callingUid == USER_UPDATE_AUTHORITY && IsPermissionGranted();
}

bool IsPermissionGranted() {
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    std::string permission = "ohos.permission.UPDATE_SYSTEM";
    int verifyResult = Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        callerToken, permission);
    return verifyResult == Security::AccessToken::PERMISSION_GRANTED;
}
```

## 调用示例

### 系统包更新示例

```cpp
#include "sys_installer_kits.h"

using namespace OHOS::SysInstaller;

// 定义回调
class MyCallback : public ISysInstallerCallback {
public:
    void OnUpgradeProgress(int32_t status, int32_t percent) override {
        // 处理进度更新
    }
};

// 执行更新
void DoUpdate(const std::string& pkgPath) {
    std::string taskId = "update_001";
    
    // 设置回调
    sptr<ISysInstallerCallback> callback = new MyCallback();
    SysInstallerKits::GetInstance().SetUpdateCallback(taskId, callback);
    
    // 开始更新
    int32_t ret = SysInstallerKits::GetInstance().StartUpdatePackageZip(taskId, pkgPath);
    if (ret != 0) {
        // 处理错误
    }
}
```

### 模块更新示例

```cpp
#include "module_update_kits.h"

using namespace OHOS::SysInstaller;

// 安装模块包
void InstallModule(const std::string& pkgPath) {
    int32_t ret = ModuleUpdateKits::GetInstance().InstallModulePackage(pkgPath);
    if (ret != 0) {
        // 处理错误
    }
}

// 查询模块版本
void QueryVersions() {
    auto versions = ModuleUpdateKits::GetInstance().GetHmpVersionInfo();
    for (const auto& info : versions) {
        // 处理版本信息
    }
}
```

## 相关链接

- [架构说明](02_Architecture.md)
- [内部 API](04_Internal_APIs.md)
- [安全风险分析](06_Security_Analysis.md)

---

*证据来源*:
- `interfaces/innerkits/ipc_client/include/sys_installer_kits.h`: SysInstallerKits 接口
- `interfaces/innerkits/ipc_client/include/module_update_kits.h`: ModuleUpdateKits 接口
- `interfaces/innerkits/ipc_client/src/sys_installer_kits_impl.cpp`: 实现
- `frameworks/ipc_server/src/sys_installer_server.cpp`: 权限检查
