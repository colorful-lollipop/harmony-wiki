# 对外接口文档 - Bundle Framework Lite

## 目录

- [C/C++ API 清单](#cc-api-清单)
- [IPC 接口](#ipc-接口)
- [JS 绑定](#js-绑定)
- [bm 工具](#bm-工具)

---

## C/C++ API 清单

### 安装/卸载 API

| 函数 | 头文件 | 说明 |
|------|----------|------|
| `Install(hapPath, installParam, callback)` | `bundle_manager.h:112` | 安装或更新应用 |
| `Uninstall(bundleName, installParam, callback)` | `bundle_manager.h:125` | 卸载应用 |

**Install 函数签名**:
```c
bool Install(const char *hapPath, 
           const InstallParam *installParam, 
           InstallerCallback installerCallback);
```

**Uninstall 函数签名**:
```c
bool Uninstall(const char *bundleName, 
             const InstallParam *installParam, 
             InstallerCallback installerCallback);
```

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:72-125`

---

### 查询 API

| 函数 | 头文件 | 说明 |
|------|----------|------|
| `QueryAbilityInfo(want, abilityInfo)` | `bundle_manager.h:142` | 根据 Want 查询 AbilityInfo |
| `QueryAbilityInfos(want, abilityInfo, len)` | `bundle_manager.h:152` | 查询所有匹配的 AbilityInfo |
| `GetBundleInfo(bundleName, flags, bundleInfo)` | `bundle_manager.h:187` | 查询指定应用的 BundleInfo |
| `GetBundleInfos(flags, bundleInfos, len)` | `bundle_manager.h:203` | 查询所有应用的 BundleInfo |
| `QueryKeepAliveBundleInfos(bundleInfos, len)` | `bundle_manager.h:216` | 查询保活应用列表 |
| `GetBundleInfosByMetaData(metaDataKey, bundleInfos, len)` | `bundle_manager.h:230` | 根据元数据查询 BundleInfo |
| `GetBundleNameForUid(uid, bundleName)` | `bundle_manager.h:243` | 根据 UID 查询 bundleName |
| `GetBundleSize(bundleName)` | `bundle_manager.h:286` | 查询应用大小 |

**GetBundleInfo 函数签名**:
```c
uint8_t GetBundleInfo(const char *bundleName, 
                   int32_t flags, 
                   BundleInfo *bundleInfo);
```

**flags 参数说明**:
- `0`: 不包含 AbilityInfo
- `1`: 包含 AbilityInfo

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:171-187`

---

### 回调注册 API

| 函数 | 头文件 | 说明 |
|------|----------|------|
| `RegisterCallback(callback)` | `bundle_manager.h:85` | 注册状态变更回调 |
| `UnregisterCallback()` | `bundle_manager.h:97` | 取消注册回调 |
| `RegisterEvent(callback)` | `bundle_manager.h:160` | 注册安装事件回调 |
| `UnregisterEvent(callback)` | `bundle_manager.h:168` | 取消注册事件回调 |

**回调类型定义**:
```c
typedef void (*InstallerCallback)(const uint8_t resultCode, 
                             const void *resultMessage);
```

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:72`

---

### 系统能力 API

| 函数 | 头文件 | 说明 |
|------|----------|------|
| `HasSystemCapability(sysCapName)` | `bundle_manager.h:255` | 查询系统是否支持特定能力 |
| `GetSystemAvailableCapabilities()` | `bundle_manager.h:265` | 获取所有系统能力列表 |
| `FreeSystemAvailableCapabilitiesInfo(sysCap)` | `bundle_manager.h:275` | 释放系统能力列表内存 |

**证据**: `interfaces/kits/bundle_lite/bundle_manager.h:245-276`

---

## IPC 接口

### BMS 服务接口

**服务名**: `bundlems`
**Feature 名**: 
- `BmsFeature` - 公开查询接口
- `BmsInnerFeature` - 内部安装/卸载接口

**证据**: `interfaces/inner_api/bundlemgr_lite/bundle_service_interface.h:37-40`

### BMS Feature 消息 ID

| 命令 ID | 值 | 说明 | 对应 C API |
|---------|------|------|-----------|
| `QUERY_ABILITY_INFO` | 0 | 查询 AbilityInfo | `QueryAbilityInfo()` |
| `GET_BUNDLE_INFO` | 1 | 查询 BundleInfo | `GetBundleInfo()` |
| `GET_BUNDLE_INFOS` | 4 | 查询所有 BundleInfo | `GetBundleInfos()` |
| `GET_BUNDLE_INFOS_BY_METADATA` | 6 | 根据元数据查询 | `GetBundleInfosByMetaData()` |
| `CHECK_SYS_CAP` | 7 | 查询系统能力 | `HasSystemCapability()` |
| `GET_BUNDLE_SIZE` | 8 | 查询应用大小 | `GetBundleSize()` |
| `GET_SYS_CAP` | 11 | 获取所有能力 | `GetSystemAvailableCapabilities()` |

**证据**: `interfaces/inner_api/bundlemgr_lite/bundle_inner_interface.h`

### BMS Inner Feature 消息 ID

| 命令 ID | 值 | 说明 | 对应 C API |
|---------|------|------|-----------|
| `INSTALL` | 12 | 安装应用 | `Install()` |
| `UNINSTALL` | 13 | 卸载应用 | `Uninstall()` |

**证据**: `interfaces/inner_api/bundlemgr_lite/bundle_inner_interface.h`

### IPC 调用示例

```cpp
// 获取 BMS IPC 代理
IUnknown *iUnknown = SAMGR_GetInstance()->GetFeatureApi(BMS_SERVICE, BMS_FEATURE);
IClientProxy *clientProxy = static_cast<IClientProxy *>(iUnknown);

// 准备请求
IpcIo req;
IpcIo reply;
IpcIoInit(&req, 0, 0, 0);
IpcIoInit(&reply, 0, 0, 0);
WriteUint32(&req, bundleNameLen);
WriteString(&req, bundleName);

// 调用 IPC
clientProxy->Invoke(GET_BUNDLE_INFO, &req, &reply, &callback);

// 读取回复
uint8_t result = ReadUint32(&reply);
```

**证据**: `frameworks/bundle_lite/src/bundle_manager.cpp`

### Bundle Daemon IPC 接口

**服务名**: `bundle_daemon`
**Feature 名**: Default Feature

**消息 ID**:

| 命令 ID | 值 | 说明 |
|---------|------|------|
| `EXTRACT_HAP` | 0 | 提取 HAP 文件 |
| `RENAME_DIR` | 1 | 重命名目录 |
| `CREATE_PERMISSION_DIR` | 2 | 创建权限目录 |
| `CREATE_DATA_DIRECTORY` | 3 | 创建数据目录 |
| `STORE_CONTENT_TO_FILE` | 4 | 写入文件内容 |
| `MOVE_FILE` | 5 | 移动文件 |
| `REMOVE_FILE` | 6 | 删除文件 |
| `REMOVE_INSTALL_DIRECTORY` | 7 | 删除安装目录 |

**证据**: `interfaces/inner_api/bundlemgr_lite/bundle_daemon_interface.h`

---

## JS 绑定

### JSI 模块：@system.capability

**模块路径**: `interfaces/kits/bundle_lite/js/builtin/`
**模块初始化**: `InitCapabilityModule(JSIValue exports)`

**导出函数**:

#### capability.has(sysCapName)

**功能**: 查询系统是否支持特定能力

**参数**:
- `sysCapName` (String): 系统能力名称

**返回**: Boolean

**示例**:
```javascript
import capability from '@system.capability';

const hasBundleManager = capability.has('SystemCapability.BundleManager');
console.log(hasBundleManager); // true or false
```

**证据**: `interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp`

### JSI 函数映射

| JS API | C++ 函数 | 说明 |
|--------|----------|------|
| `capability.has()` | `CapabilityModule::HasCapability()` | 调用 `HasSystemCapability()` C API |

**证据**: `interfaces/kits/bundle_lite/js/builtin/src/capability_module.cpp`

---

## bm 工具

### 命令列表

| 命令 | 说明 | 选项 |
|------|------|------|
| `install` | 安装 HAP 包 | `-p, --happath <path>` |
| `uninstall` | 卸载应用 | `-n, --bundlename <name>` |
| `dump` | 查询应用信息 | `-l` (列表), `-n <name>` (详情), `-m <key>` (元数据) |
| `getudid` | 获取设备 UDID | 无 |
| `set` | 设置调试模式（debug 构建）| `-e` (外部模式), `-d` (调试模式), `-s` (签名模式) |

**证据**: `services/bundlemgr_lite/tools/src/command_parser.cpp`

### bm install 示例

```bash
# 基础安装
./bin/bm install -p /sdcard/app.hap

# 安装到外部存储
./bin/bm install -p /sdcard/app.hap

# 安装并保留数据（更新）
./bin/bm install -p /sdcard/app_v2.hap -r
```

**证据**: `README.md:44-48`

### bm uninstall 示例

```bash
# 基础卸载
./bin/bm uninstall -n com.example.app

# 卸载并保留数据（可能不支持）
./bin/bm uninstall -n com.example.app
```

### bm dump 示例

```bash
# 列出所有已安装应用
./bin/bm dump -l

# 查询指定应用详情
./bin/bm dump -n com.example.app

# 根据元数据查询
./bin/bm dump -m custom.metadata.key
```

---

## 数据结构

### BundleInfo

**定义位置**: `interfaces/kits/bundle_lite/bundle_info.h`

**关键字段**:
```c
typedef struct {
    char *bundleName;        // Bundle 名称
    char *versionName;       // 版本名称
    uint32_t versionCode;     // 版本代码
    char *vendor;           // 供应商
    char *smallIconPath;    // 小图标路径
    char *bigIconPath;      // 大图标路径
    uint8_t isSystemApp;     // 是否系统应用
    uint8_t isKeepAlive;     // 是否保活应用
    AbilityInfo *abilityInfos;  // Ability 信息数组
    int32_t abilityInfoNum;  // Ability 数量
} BundleInfo;
```

**证据**: `interfaces/kits/bundle_lite/bundle_info.h`

### AbilityInfo

**定义位置**: `interfaces/kits/bundle_lite/ability_info.h`

**关键字段**:
```c
typedef struct {
    char *bundleName;        // Bundle 名称
    char *abilityName;       // Ability 名称
    char *label;            // 显示标签
    char *iconPath;          // 图标路径
    char *description;       // 描述
    uint8_t isVisible;       // 是否可见
    uint8_t type;           // Ability 类型（PAGE/SERVICE/PROVIDER）
} AbilityInfo;
```

**证据**: `interfaces/kits/bundle_lite/ability_info.h`

### InstallParam

**定义位置**: `interfaces/kits/bundle_lite/install_param.h`

**关键字段**:
```c
typedef struct {
    uint8_t installLocation;  // 安装位置（INTERNAL_ONLY/PREFER_EXTERNAL）
    uint8_t keepData;       // 是否保留数据
    uint8_t replaceBundleName;// 是否替换 bundleName
} InstallParam;
```

**证据**: `interfaces/kits/bundle_lite/install_param.h`

---

## 错误码

### 常见错误码

| 错误码 | 值 | 说明 |
|--------|------|------|
| `ERR_OK` | 0 | 成功 |
| `ERR_APPEXECFWK_INSTALL_FAILED_PARSE_INVALID_BUNDLENAME` | 1 | BundleName 无效 |
| `ERR_APPEXECFWK_INSTALL_FAILED_PARSE_INVALID_PROFILE` | 2 | Profile 解析失败 |
| `ERR_APPEXECFWK_INSTALL_FAILED_PARSE_INVALID_PERMISSION` | 3 | 权限无效 |
| `ERR_APPEXECFWK_INSTALL_FAILED_PARSE_INVALID_SIGNATURE` | 4 | 签名无效 |
| `ERR_APPEXECFWK_INSTALL_FAILED_FILE_NOT_EXIST` | 5 | 文件不存在 |
| `ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_FOUND` | 6 | 应用未找到 |
| `ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_UNINSTALLABLE` | 7 | 不可卸载（系统应用）|

**证据**: `interfaces/kits/bundle_lite/appexecfwk_errors.h`

---

## 相关文档

- [项目概览](01_Overview.md) - 快速开始示例
- [架构与数据流](03_Architecture.md) - IPC 调用时序
- [目录结构与代码地图](02_CodeMap.md) - API 实现位置

---

**最后更新**: 2026-02-07
