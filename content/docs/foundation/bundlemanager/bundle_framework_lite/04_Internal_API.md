# 内部 API

## 目的

本文档说明 bundle_framework_lite 的内部模块接口，帮助开发者理解模块间的依赖关系和调用方式。

## 接口分层

```
┌─────────────────────────────────────────────────────────────┐
│                      对外 API 层                             │
│              interfaces/kits/bundle_lite                     │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                      内部 API 层                             │
│              interfaces/inner_api/bundlemgr_lite             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │bundle_service│  │bundle_inner_│  │bundle_daemon_       │  │
│  │_interface.h │  │interface.h  │  │interface.h          │  │
│  │(服务接口)    │  │(内部接口)    │  │(Daemon 接口)        │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## IPC 接口定义

### 1. 服务接口 (bundle_service_interface.h)

**位置**: `interfaces/inner_api/bundlemgr_lite/bundle_service_interface.h`

**功能**: 定义 BMS 对外提供的服务接口

```cpp
struct BmsSliteInterface {
    INHERIT_IUNKNOWN;
    uint8_t (*Install)(const char *path, uint8_t bundleStyle);
    uint8_t (*Uninstall)(const char *bundleName);
    uint8_t (*GetBundleInfo)(const char *bundleName, int32_t flags, BundleInfo *bundleInfo);
    uint8_t (*GetBundleInfos)(int32_t flags, BundleInfo **bundleInfos, int32_t *len);
    // ...
};
```

**服务标识**:
```cpp
static const char BMS_SERVICE[] = "bundlems";           // 服务名
static const char BMS_FEATURE[] = "BmsFeature";         // 对外特性名
static const char BMS_SLITE_FEATURE[] = "BmsSliteFeature";  // Slite 特性名
```

### 2. 内部接口 (bundle_inner_interface.h)

**位置**: `interfaces/inner_api/bundlemgr_lite/bundle_inner_interface.h`

**功能**: 定义 BMS 内部 IPC 命令和代理

**IPC 命令枚举** (`BmsCmd`):

| 命令 | 值 | 说明 |
|------|-----|------|
| QUERY_ABILITY_INFO | 0 | 查询 Ability 信息 |
| GET_BUNDLE_INFO | 1 | 获取 Bundle 信息 |
| CHANGE_CALLBACK_SERVICE_IDENTITY | 2 | 变更回调服务标识 |
| GET_BUNDLENAME_FOR_UID | 3 | 通过 UID 获取包名 |
| GET_BUNDLE_INFOS | 4 | 获取所有 Bundle 信息 |
| QUERY_KEEPALIVE_BUNDLE_INFOS | 5 | 查询保活应用 |
| GET_BUNDLE_INFOS_BY_METADATA | 6 | 通过元数据查询 |
| CHECK_SYS_CAP | 7 | 检查系统能力 |
| GET_BUNDLE_SIZE | 8 | 获取应用大小 |
| GET_BUNDLE_INFO_LENGTH | 9 | 获取 BundleInfo 长度 |
| GET_BUNDLE_INFO_BY_INDEX | 10 | 通过索引获取 |
| GET_SYS_CAP | 11 | 获取系统能力 |
| INSTALL | 12 | 安装应用 |
| UNINSTALL | 13 | 卸载应用 |
| SET_EXTERNAL_INSTALL_MODE | 14 | 设置外部安装模式 |
| SET_SIGN_DEBUG_MODE | 15 | 设置签名调试模式 |
| SET_SIGN_MODE | 16 | 设置签名模式 |

**代理结构**:
```cpp
// 服务端代理
typedef struct BmsServerProxy {
    INHERIT_SERVER_IPROXY;
    // 方法指针...
} BmsServerProxy;

// 内部服务端代理
typedef struct BmsInnerServerProxy {
    INHERIT_SERVER_IPROXY;
    // 方法指针...
} BmsInnerServerProxy;
```

### 3. Daemon 接口 (bundle_daemon_interface.h)

**位置**: `interfaces/inner_api/bundlemgr_lite/bundle_daemon_interface.h`

**功能**: 定义与 Bundle Daemon 通信的 IPC 接口

**服务标识**:
```cpp
static const char BDS_SERVICE[] = "bundle_daemon";  // Daemon 服务名
```

**Daemon 命令枚举** (`BdsCmd`):

| 命令 | 值 | 说明 |
|------|-----|------|
| EXTRACT_HAP | 0 | 解压 HAP 包 |
| RENAME_DIR | 1 | 重命名目录 |
| CREATE_PERMISSION_DIR | 2 | 创建权限目录 |
| CREATE_DATA_DIRECTORY | 3 | 创建数据目录 |
| STORE_CONTENT_TO_FILE | 4 | 存储内容到文件 |
| MOVE_FILE | 5 | 移动文件 |
| REMOVE_FILE | 6 | 删除文件 |
| REMOVE_INSTALL_DIRECTORY | 7 | 删除安装目录 |
| REGISTER_CALLBACK | 8 | 注册回调 |
| BDS_CALLBACK | 9 | Daemon 回调 |
| BDS_BUTT | 10 | 命令上限 |

## 核心模块接口

### ManagerService (服务管理)

**位置**: `services/bundlemgr_lite/include/bundle_manager_service.h:33`

**类定义**:
```cpp
class ManagerService {
public:
    static ManagerService &GetInstance();
    
    // 消息处理
    void ServiceMsgProcess(Request *request);
    
    // Bundle 信息管理
    BundleInfo *QueryBundleInfo(const char *bundleName);
    void RemoveBundleInfo(const char *bundleName);
    void AddBundleInfo(BundleInfo *info);
    bool UpdateBundleInfo(BundleInfo *info);
    uint8_t GetBundleInfo(const char *bundleName, int32_t flags, BundleInfo& bundleInfo);
    uint8_t GetBundleInfos(int32_t flags, BundleInfo **bundleInfos, int32_t *len);
    
    // UID 管理
    int32_t GenerateUid(const char *bundleName, int8_t bundleStyle);
    void RecycleUid(const char *bundleName);
    
    // 路径获取
    std::string GetCodeDirPath() const;
    std::string GetDataDirPath() const;
    
    // 模式设置
    uint8_t SetExternalInstallMode(bool enable);
    bool IsExternalInstallMode() const;
    uint8_t SetDebugMode(bool enable);
    bool IsDebugMode() const;
    
    // 系统能力
    bool HasSystemCapability(const char *bundleName);
    uint8_t GetSystemAvailableCapabilities(char syscap[][MAX_SYSCAP_NAME_LEN], int32_t *len);
    
    // AMS 接口
    static bool GetAmsInterface(AmsInnerInterface **amsInterface);
    
private:
    ManagerService();
    ~ManagerService();
    // ...
};
```

**依赖关系**:
- 依赖 `BundleInstaller` 进行安装/卸载
- 依赖 `BundleMap` 进行 Bundle 信息存储
- 依赖 SAMGR 进行服务管理

### BundleInstaller (安装器)

**位置**: `services/bundlemgr_lite/include/bundle_installer.h:36`

**类定义**:
```cpp
class BundleInstaller {
public:
    BundleInstaller(const std::string &codeDirPath, const std::string &dataDirPath);
    ~BundleInstaller();
    
    // 安装/卸载
    uint8_t Install(const char *path, const InstallParam &installParam);
    uint8_t Uninstall(const char *bundleName, const InstallParam &installParam);
    
    // 路径获取
    std::string GetCodeDirPath() const;
    std::string GetDataDirPath() const;
    
private:
    // 安装流程
    uint8_t ProcessBundleInstall(const std::string &path, const char *randStr,
        InstallRecord &installRecord, uint8_t hapType);
    
    // 验证
    uint8_t CheckInstallFileIsValid(const char *path);
    uint8_t CheckVersionAndSignature(const char *bundleName, BundleInfo *bundleInfo);
    uint8_t CheckProvisionInfoIsValid(const SignatureInfo &signatureInfo, 
        const Permissions &permissions, const char *bundleName);
    
    // 权限
    uint8_t StorePermissions(const char *bundleName, PermissionTrans *permissions, 
        int32_t permNum, bool isUpdate);
    
    // 文件操作
    bool BackUpInstallRecord(const InstallRecord &record, const char *jsonPath);
    bool RenameJsonFile(const char *fileName, const char *randStr);
    
    std::string codeDirPath_;
    std::string dataDirPath_;
};
```

**依赖关系**:
- 依赖 `BundleParser` 解析 HAP
- 依赖 `HapSignVerify` 验证签名
- 依赖 `BundleDaemonClient` 执行文件操作
- 依赖 `BundleResTransform` 转换资源

### BundleMap (Bundle 存储)

**位置**: `services/bundlemgr_lite/include/bundle_map.h:28`

**类定义**:
```cpp
class BundleMap {
public:
    static BundleMap *GetInstance();
    
    // CRUD 操作
    void Add(BundleInfo *info);
    BundleInfo *Get(const char *bundleName);
    void Erase(const char *bundleName);
    bool Update(BundleInfo *info);
    void Clear();
    
    // 批量查询
    uint8_t GetBundleInfos(int32_t flags, BundleInfo **bundleInfos, int32_t *len);
    uint8_t GetBundleInfo(const char *bundleName, int32_t flags, BundleInfo &bundleInfo);
    
    // 特殊查询
    uint8_t QueryKeepAliveBundleInfos(BundleInfo **bundleInfos, int32_t *len);
    uint8_t GetBundleInfosByMetaData(const char *metaDataKey, BundleInfo **bundleInfos, int32_t *len);
    
private:
    BundleMap();
    ~BundleMap();
    // ...
};
```

**存储结构**:
```cpp
std::map<std::string, BundleInfo*> bundleMap_;  // bundleName -> BundleInfo*
```

### BundleParser (HAP 解析器)

**位置**: `services/bundlemgr_lite/include/bundle_parser.h:28`

**类定义**:
```cpp
class BundleParser {
public:
    BundleParser();
    ~BundleParser();
    
    // 解析 HAP 包
    BundleInfo *ParseHapProfile(const char *path);
    uint8_t ParseHapProfile(const std::string &path, Permissions &permissions, 
        BundleRes &bundleRes, BundleInfo **bundleInfo);
    
    // 解析特定文件
    static uint8_t ParseBundleParam(const char *path, char **bundleName, int32_t &versionCode);
    
private:
    // 解析 config.json
    uint8_t ParseJsonInfo(const char *path, BundleInfo *info);
    uint8_t ParseAppInfo(const cJSON *object, BundleInfo *info);
    uint8_t ParseModuleInfo(const cJSON *object, BundleInfo *info);
    uint8_t ParseAbilityInfo(const cJSON *object, BundleInfo *info);
    // ...
};
```

### BundleDaemonClient (Daemon 客户端)

**位置**: `services/bundlemgr_lite/include/bundle_daemon_client.h:28`

**类定义**:
```cpp
class BundleDaemonClient {
public:
    static BundleDaemonClient &GetInstance();
    
    // 初始化
    bool Initialize();
    
    // HAP 操作
    int32_t ExtractHap(const char *hapPath, const char *codePath);
    
    // 目录操作
    int32_t CreateDataDirectory(const char *dataPath, int32_t uid, int32_t gid, bool isChown);
    int32_t RemoveInstallDirectory(const char *codePath, const char *dataPath, bool keepData);
    int32_t RenameFile(const char *oldPath, const char *newPath);
    
    // 文件操作
    int32_t RemoveFile(const char *filePath);
    int32_t MoveFile(const char *srcPath, const char *dstPath);
    int32_t StoreContentToFile(const char *filePath, const char *content, uint32_t len);
    
    // 权限目录
    int32_t CreatePermissionDir();
    
private:
    BundleDaemonClient();
    ~BundleDaemonClient();
    // ...
};
```

**单例实现**:
```cpp
BundleDaemonClient &BundleDaemonClient::GetInstance() {
    static BundleDaemonClient instance;
    return instance;
}
```

### HapSignVerify (签名验证)

**位置**: `services/bundlemgr_lite/include/hap_sign_verify.h:28`

**类定义**:
```cpp
class HapSignVerify {
public:
    static uint8_t VerifySignature(const std::string &hapFilepath, SignatureInfo &signatureInfo);
};

struct SignatureInfo {
    std::string appId;
    std::string provisionBundleName;
    std::vector<std::string> restrictedPermissions;
};
```

**调用链**:
```
HapSignVerify::VerifySignature()
  └── APPVERI_AppVerify()  // 调用 appverify_lite 库
       └── 验证 HAP 签名和证书链
```

## 模块依赖图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           模块依赖关系                                   │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│  BundleManager  │ (对外 API 实现)
└────────┬────────┘
         │ 调用
         ▼
┌─────────────────┐     ┌─────────────────┐
│   BMS Feature   │◄────│  BMS Host       │
│  (对外特性)      │     │  (服务注册)      │
└────────┬────────┘     └─────────────────┘
         │
         ▼
┌─────────────────┐
│ ManagerService  │◄────┐
│  (服务管理单例)  │     │
└────────┬────────┘     │
         │              │
    ┌────┴────┬─────────┘
    ▼         ▼         ▼
┌────────┐ ┌────────┐ ┌──────────────┐
│Bundle  │ │Bundle  │ │BundleDaemon  │
│Installer│ │Parser  │ │Client        │
└───┬────┘ └───┬────┘ └──────┬───────┘
    │          │             │
    ▼          ▼             ▼
┌────────┐ ┌────────┐ ┌──────────────┐
│HapSign │ │ZipFile │ │ BundleDaemon │
│Verify  │ │        │ │  (独立进程)   │
└────────┘ └────────┘ └──────────────┘
```

## 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `bundle_manager.h` (C API) | **稳定** | 对外公开接口，向后兼容 |
| `capability_module.h` (JS API) | **稳定** | 对外公开接口，向后兼容 |
| `bundle_service_interface.h` | **内部** | 内部 IPC 接口，可能变更 |
| `bundle_inner_interface.h` | **内部** | 内部 IPC 接口，可能变更 |
| `bundle_daemon_interface.h` | **内部** | 内部 IPC 接口，可能变更 |
| `ManagerService` | **内部** | 内部类，可能变更 |
| `BundleInstaller` | **内部** | 内部类，可能变更 |

## 可替换点

### 1. 签名验证模块

**位置**: `services/bundlemgr_lite/src/hap_sign_verify.cpp`

可替换为其他签名验证实现，需保持接口兼容：
```cpp
class HapSignVerify {
public:
    static uint8_t VerifySignature(const std::string &hapFilepath, SignatureInfo &signatureInfo);
};
```

### 2. 存储后端

**位置**: `services/bundlemgr_lite/src/bundle_map.cpp`

当前使用内存中的 `std::map`，可替换为数据库或其他持久化存储。

### 3. 解压实现

**位置**: `services/bundlemgr_lite/src/zip_file.cpp`

当前使用 zlib，可替换为其他 ZIP 库。

---

**相关链接**:
- [对外 API](03_Public_API.md)
- [架构说明](02_Architecture.md)
- [GN 构建目标](05_GN_Targets.md)
