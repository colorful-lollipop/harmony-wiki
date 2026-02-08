# OpenHarmony Updater 对外 API

## 目的

本文档详细说明 Updater 子系统提供的对外 C++ API，供应用开发者和其他子系统调用。

## 适用范围

- 应用开发者（开发 OTA 应用）
- 子系统开发者（调用 Updater 能力）

## 重要说明

**当前 Updater 不提供 N-API（JavaScript 绑定）**，仅提供 C/C++ 接口。应用需要通过 Native 代码调用。

## API 清单

### 1. UpdaterKits - 升级触发接口

**头文件**: `interfaces/kits/include/updaterkits/updaterkits.h`

**库文件**: 
- 动态库: `libupdater_shared.so`
- 静态库: `libupdaterkits.a`

#### 函数列表

| 函数 | 签名 | 说明 |
|------|------|------|
| `RebootAndInstallUpgradePackage` | `int RebootAndInstallUpgradePackage(const std::string &miscFile, const std::vector<std::string> &packageName, const std::string &upgradeType = UPGRADE_TYPE_OTA)` | 重启并安装 OTA 包 |
| `RebootAndInstallUpgradePackage` | `int RebootAndInstallUpgradePackage(const std::string &miscFile, const std::vector<std::string> &packageName, const std::string &upgradeType, const RebootFunType &rebootFunc)` | 带自定义重启函数 |
| `RebootAndInstallSdcardPackage` | `bool RebootAndInstallSdcardPackage(const std::string &miscFile, const std::vector<std::string> &packageName)` | 重启并安装 SD 卡升级包 |
| `RebootAndCleanUserData` | `bool RebootAndCleanUserData(const std::string &miscFile, const std::string &cmd)` | 重启并清除用户数据 |

#### 常量定义

```cpp
// interfaces/kits/include/updaterkits/updaterkits.h:21-25
constexpr const char *UPGRADE_TYPE_OTA = "ota";
constexpr const char *UPGRADE_TYPE_SD = "sdcard";
constexpr const char *UPGRADE_TYPE_OTA_INTRAL = "ota_intral";
constexpr const char *UPGRADE_TYPE_SD_INTRAL = "sdcard_intral";
constexpr const char *UPGRADE_TYPE_SUBPKG_UPDATE = "subpkg_update";
```

#### 类型定义

```cpp
// interfaces/kits/include/updaterkits/updaterkits.h:20
using RebootFunType = std::function<int()>;
```

#### 调用示例

```cpp
#include "updaterkits/updaterkits.h"

// 触发 OTA 升级
std::vector<std::string> packages = {"/data/update/ota.zip"};
int ret = RebootAndInstallUpgradePackage("/dev/block/misc", packages, UPGRADE_TYPE_OTA);
if (ret != 0) {
    // 处理错误
}

// 触发恢复出厂
bool result = RebootAndCleanUserData("/dev/block/misc", "factory_reset");
```

---

### 2. MiscInfo - Misc 分区操作

**头文件**: `interfaces/kits/include/misc_info/misc_info.h`

**库文件**: `libmiscinfo.a`

#### 数据结构

```cpp
// interfaces/kits/include/misc_info/misc_info.h:49-62
struct UpdateMessage {
    char command[MAX_COMMAND_SIZE];      // 32 bytes
    char status[MAX_STATUS_SIZE];        // 32 bytes
    char update[MAX_UPDATE_SIZE];        // 768 bytes - 升级包路径
    char stage[MAX_STAGE_SIZE];          // 32 bytes
    char faultinfo[MAX_FAULTINFO_SIZE];  // 32 bytes
    char reserved[MAX_RESERVED_SIZE];    // 224 bytes
};

struct UpdaterPara {
    char language[MAX_PARA_SIZE];        // 32 bytes
    char osVersionSuffix[MAX_PARA_SIZE]; // 32 bytes
    char reserved[MAX_RESERVED_SIZE_PARA]; // 192 bytes
};
```

#### 函数列表

| 函数 | 签名 | 说明 |
|------|------|------|
| `WriteUpdaterMessage` | `bool WriteUpdaterMessage(const std::string &path, const UpdateMessage &boot)` | 写入更新消息到指定路径 |
| `ReadUpdaterMessage` | `bool ReadUpdaterMessage(const std::string &path, UpdateMessage &boot)` | 从指定路径读取更新消息 |
| `WriteUpdaterMiscMsg` | `bool WriteUpdaterMiscMsg(const UpdateMessage &boot)` | 写入 Misc 分区（默认路径） |
| `ReadUpdaterMiscMsg` | `bool ReadUpdaterMiscMsg(UpdateMessage &boot)` | 读取 Misc 分区（默认路径） |
| `WriteUpdaterParaMisc` | `bool WriteUpdaterParaMisc(const UpdaterPara &para)` | 写入 Updater 参数 |
| `ReadUpdaterParaMisc` | `bool ReadUpdaterParaMisc(UpdaterPara &para)` | 读取 Updater 参数 |
| `ClearUpdaterParaMisc` | `void ClearUpdaterParaMisc(void)` | 清除 Updater 参数 |

#### 常量定义

```cpp
// interfaces/kits/include/misc_info/misc_info.h:24-47
constexpr int MAX_COMMAND_SIZE = 32;
constexpr int MAX_STATUS_SIZE = 32;
constexpr int MAX_UPDATE_SIZE = 768;
constexpr int MAX_STAGE_SIZE = 32;
constexpr int MAX_FAULTINFO_SIZE = 32;
constexpr int MAX_RESERVED_SIZE = 224;

constexpr off_t MISC_BASE_OFFSET = 0;
constexpr off_t MISC_UPDATER_PARA_OFFSET = 1024 * 1024; // 1MB 偏移
```

#### 调用示例

```cpp
#include "misc_info/misc_info.h"

// 写入升级命令
Updater::UpdateMessage msg = {};
strncpy(msg.command, "boot_updater", sizeof(msg.command));
strncpy(msg.update, "/data/update/ota.zip", sizeof(msg.update));
bool ret = Updater::WriteUpdaterMiscMsg(msg);

// 读取 Misc 信息
Updater::UpdateMessage readMsg;
if (Updater::ReadUpdaterMiscMsg(readMsg)) {
    // 处理读取到的信息
}
```

---

### 3. Packages - 包管理接口

**头文件**: `interfaces/kits/include/package/package.h`

**库文件**: 
- 动态库: `libpackage_shared.so`
- 静态库: `libpackageExt.a`

#### 数据结构

```cpp
// 包类型
enum PkgPackType {
    PKG_PACK_TYPE_UPGRADE = 0,  // 升级包
    PKG_PACK_TYPE_ZIP,          // ZIP 包
    PKG_PACK_TYPE_LZ4,          // LZ4 包
    PKG_PACK_TYPE_GZIP,         // GZIP 包
};

// 压缩方法
enum PkgCompressMethod {
    PKG_COMPRESS_NONE = -1,
    PKG_COMPRESS_ZSTD = 0,
    PKG_COMPRESS_LZ4,
    PKG_COMPRESS_ZIP,
    PKG_COMPRESS_GZIP,
};

// 摘要方法
enum PkgDigestMethod {
    PKG_DIGEST_TYPE_CRC = 0,
    PKG_DIGEST_TYPE_SHA256,
    PKG_DIGEST_TYPE_SHA384,
    PKG_DIGEST_TYPE_SHA512,
};

// 签名方法
enum PkgSignMethod {
    PKG_SIGN_METHOD_RSA = 0,
    PKG_SIGN_METHOD_ECDSA,
};

// 升级包信息
struct UpgradePkgInfoExt {
    uint32_t pkgType;              // 包类型
    uint32_t pkgFlags;             // 标志
    uint32_t pkgVersion;           // 包版本
    PkgDigestMethod digestMethod;  // 摘要算法
    PkgSignMethod signMethod;      // 签名算法
    size_t signAlgLength;          // 签名算法参数长度
    size_t headSignLength;         // 头部签名长度
    size_t dataSignLength;         // 数据签名长度
    const uint8_t *signAlg;        // 签名算法参数
    const uint8_t *headSign;       // 头部签名
    const uint8_t *dataSign;       // 数据签名
    size_t pkgLength;              // 包长度
};

// 组件信息
struct ComponentInfoExt {
    char *componentAddr;           // 组件地址
    uint32_t componentSize;        // 组件大小
    PkgCompressMethod compressMethod; // 压缩方法
    PkgDigestMethod digestMethod;  // 摘要方法
    uint8_t *digest;               // 摘要值
    size_t digestLength;           // 摘要长度
    uint8_t *version;              // 版本
    uint8_t *id;                   // ID
    uint8_t *originalSize;         // 原始大小
    uint8_t *originalDigest;       // 原始摘要
};
```

#### 函数列表

| 函数 | 签名 | 说明 |
|------|------|------|
| `CreatePackage` | `int32_t CreatePackage(const UpgradePkgInfoExt *pkgInfo, std::vector<ComponentInfoExt> &comp, const char *path, const char *keyPath)` | 创建升级包 |
| `VerifyPackage` | `int32_t VerifyPackage(const char *packagePath, const char *keyPath, const char *version, const uint8_t *digest, size_t size)` | 验证升级包 |
| `VerifyPackageWithCallback` | `int32_t VerifyPackageWithCallback(const std::string &packagePath, const std::string &keyPath, std::function<void(int32_t result, uint32_t percent)> cb)` | 带回调的验证 |
| `ExtraPackageDir` | `int32_t ExtraPackageDir(const char *packagePath, const char *keyPath, const char *dir, const char *outPath)` | 解压目录 |
| `ExtraPackageFile` | `int32_t ExtraPackageFile(const char *packagePath, const char *keyPath, const char *file, const char *outPath)` | 解压单个文件 |

#### 调用示例

```cpp
#include "package/package.h"

// 验证升级包
int32_t ret = VerifyPackage("/data/update/ota.zip", "/system/etc/key.pem", 
                            "1.0.0", nullptr, 0);
if (ret == 0) {
    // 验证成功
}

// 创建升级包
UpgradePkgInfoExt pkgInfo = {};
pkgInfo.pkgType = PKG_PACK_TYPE_UPGRADE;
pkgInfo.digestMethod = PKG_DIGEST_TYPE_SHA256;
pkgInfo.signMethod = PKG_SIGN_METHOD_RSA;
// ... 填充其他字段

std::vector<ComponentInfoExt> components;
// ... 填充组件

int32_t result = CreatePackage(&pkgInfo, components, "/data/out.pkg", "/keys/private.pem");
```

---

### 4. SlotInfo - AB 分区槽管理

**头文件**: `interfaces/kits/include/slot_info/slot_info.h`

**库文件**: `libslotinfo.a`

#### 函数列表

| 函数 | 签名 | 说明 |
|------|------|------|
| `GetPartitionSuffix` | `void GetPartitionSuffix(std::string &suffix)` | 获取非活跃分区后缀（如 "_a" 或 "_b"） |
| `GetActivePartitionSuffix` | `void GetActivePartitionSuffix(std::string &suffix)` | 获取活跃分区后缀 |
| `SetActiveSlot` | `void SetActiveSlot()` | 切换活跃分区槽 |

#### 条件编译

```cpp
// interfaces/kits/include/slot_info/slot_info.h
#ifdef UPDATER_AB_SUPPORT
    // 函数实现
#else
    // 空实现（无 AB 支持时）
#endif
```

#### 调用示例

```cpp
#include "slot_info/slot_info.h"

// 获取当前非活跃槽（用于升级目标）
std::string suffix;
Updater::GetPartitionSuffix(suffix);
// suffix = "_b" 或 "_a"

// 切换活跃槽（升级完成后）
Updater::SetActiveSlot();
```

---

### 5. DiffPatch - 差分补丁接口

**头文件**: `interfaces/kits/include/diff_patch/diff_patch_interface.h`

**库文件**: 
- 动态库: `libdiff_patch_shared.so`
- 静态库: `libdiff_patch.a`

#### 函数列表

| 函数 | 签名 | 说明 |
|------|------|------|
| `ApplyPatch` | `int32_t ApplyPatch(const std::string &patchFile, const std::string &oldfile, const std::string &newFile)` | 应用差分补丁 |

#### 调用示例

```cpp
#include "diff_patch/diff_patch_interface.h"

// 应用差分补丁生成新文件
int32_t ret = ApplyPatch("/data/patch.diff", "/system/app.old", "/data/app.new");
if (ret == 0) {
    // 补丁应用成功
}
```

---

## API 汇总表

| Kit | 共享库 | 静态库 | 头文件 | 主要用途 |
|-----|--------|--------|--------|---------|
| updaterkits | `libupdater_shared.so` | `libupdaterkits.a` | `updaterkits/updaterkits.h` | 触发升级、重启 |
| misc_info | - | `libmiscinfo.a` | `misc_info/misc_info.h` | Misc 分区操作 |
| packages | `libpackage_shared.so` | `libpackageExt.a` | `package/package.h` | 包创建、验证 |
| slot_info | - | `libslotinfo.a` | `slot_info/slot_info.h` | AB 分区管理 |
| diff_patch | `libdiff_patch_shared.so` | `libdiff_patch.a` | `diff_patch/diff_patch_interface.h` | 差分补丁 |

## 错误码

常用错误码定义在 `utils/include/error_code.h`：

```cpp
enum UpdaterErrorCode {
    CODE_SUCCESS = 0,
    CODE_VERIFY_FAIL = 1,
    CODE_MOUNT_FAIL = 2,
    CODE_INSTALL_FAIL = 3,
    CODE_REBOOT_FAIL = 4,
    CODE_PACKAGE_INVALID = 5,
    CODE_SPACE_NOTENOUGH = 6,
    CODE_BATTERY_LOW = 7,
    CODE_NETWORK_FAIL = 8,
};
```

## 权限要求

| API | 权限要求 |
|-----|---------|
| `RebootAndInstallUpgradePackage` | system 或 root |
| `WriteUpdaterMiscMsg` | system 或 root |
| `CreatePackage` | 无特殊要求 |
| `VerifyPackage` | 无特殊要求 |
| `SetActiveSlot` | system 或 root |
| `ApplyPatch` | 无特殊要求 |

## 关键结论

1. **无 N-API**: 当前 Updater 只提供 C++ 接口，应用需要通过 Native 代码调用。

2. **分层设计**: `updaterkits` 提供高层接口，`misc_info` 提供底层控制。

3. **AB 分区支持**: `slot_info` 提供 AB 分区切换能力，但需在支持 AB 的设备上使用。

4. **包管理完整**: `packages` 提供完整的包创建、验证、提取能力。

5. **权限敏感**: 涉及重启、分区写入的操作需要 system 或 root 权限。

## 相关跳转

- [项目概览](./00_Overview.md)
- [架构说明](./01_Architecture.md)
- [内部接口 - PkgManager](./04_Inner_API.md#pkg_manager)
- [安全分析](./06_Security_Analysis.md)
