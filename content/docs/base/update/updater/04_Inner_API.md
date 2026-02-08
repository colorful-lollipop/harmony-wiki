# OpenHarmony Updater 内部 API

## 目的

本文档描述 Updater 子系统内部模块的接口，供子系统开发者进行二次开发或维护。

## 适用范围

- 子系统开发者
- 维护工程师

## 接口稳定性说明

| 稳定性级别 | 说明 | 使用建议 |
|----------|------|---------|
| **Stable** | 接口稳定，向后兼容 | 可放心使用 |
| **Unstable** | 可能变更 | 谨慎使用 |
| **Internal** | 仅供内部使用 | 不建议外部调用 |
| **Deprecated** | 已废弃 | 请勿使用 |

---

## 1. PkgManager - 包管理器

**稳定性**: Stable  
**头文件**: `services/include/package/pkg_manager.h`

### 类定义

```cpp
class PkgManager {
public:
    using PkgManagerPtr = PkgManager *;
    using VerifyCallback = std::function<void(int32_t result, uint32_t percent)>;
    
    // 创建/获取/释放实例
    static PkgManagerPtr CreatePackageInstance();
    static PkgManagerPtr GetPackageInstance();
    static void ReleasePackageInstance(PkgManagerPtr manager);
    
    // 包创建
    virtual int32_t CreatePackage(const std::string &path, const std::string &keyName, 
                                  PkgInfoPtr header, std::vector<std::pair<std::string, ZipFileInfo>> &files) = 0;
    
    // 包验证
    virtual int32_t VerifyPackage(const std::string &packagePath, const std::string &keyPath,
                                  const std::string &version, const PkgBuffer &digest, VerifyCallback cb) = 0;
    
    // 包加载
    virtual int32_t LoadPackage(const std::string &packagePath, const std::string &keyPath,
                                  std::vector<std::string> &fileIds) = 0;
    
    // 文件提取
    virtual int32_t ExtractFile(const std::string &fileId, StreamPtr output) = 0;
    
    // 流创建
    virtual int32_t CreatePkgStream(StreamPtr &stream, const std::string &fileName, 
                                  size_t size, int32_t type) = 0;
    virtual void ClosePkgStream(StreamPtr &stream) = 0;
    
    // 包信息
    virtual const PkgInfo *GetPackageInfo(const std::string &packagePath) = 0;
    virtual const FileInfo *GetFileInfo(const std::string &fileId) = 0;
};
```

### PkgStream 类型

```cpp
class PkgStream {
public:
    enum StreamType {
        PkgStreamType_Read = 0,      // 普通文件读取
        PkgStreamType_Write,         // 普通文件写入
        PkgStreamType_MemoryMap,     // 内存映射
        PkgStreamType_Process,       // 边解析边处理
        PkgStreamType_Buffer,        // 缓冲区
        PKgStreamType_FileMap,       // 文件映射到内存
        PkgStreamType_FlowData,      // 流数据
        PkgStreamType_ShmData        // 共享内存数据
    };
    
    virtual int32_t Read(PkgBuffer &data, size_t start, size_t needRead, size_t &readLen) = 0;
    virtual int32_t Write(const PkgBuffer &data, size_t size, size_t start) = 0;
    virtual int32_t Flush(size_t size) = 0;
    virtual size_t GetFileLength() = 0;
};
```

### 包类型枚举

```cpp
class PkgFile {
public:
    enum PkgType {
        PKG_TYPE_NONE = PKG_PACK_TYPE_NONE,
        PKG_TYPE_UPGRADE = PKG_PACK_TYPE_UPGRADE,  // 升级包
        PKG_TYPE_ZIP = PKG_PACK_TYPE_ZIP,          // ZIP 包
        PKG_TYPE_LZ4 = PKG_PACK_TYPE_LZ4,          // LZ4 包
        PKG_TYPE_GZIP = PKG_PACK_TYPE_GZIP,        // GZIP 包
    };
};
```

---

## 2. ScriptManager - 脚本管理器

**稳定性**: Stable  
**头文件**: `services/include/script/script_manager.h`

### 类定义

```cpp
class ScriptManager {
public:
    virtual ~ScriptManager() = default;
    
    // 执行脚本文件
    virtual int32_t ExecuteScript(const std::string &scriptFile) = 0;
    
    // 执行脚本字符串
    virtual int32_t ExecuteScriptString(const std::string &scriptContent) = 0;
    
    // 注册指令
    virtual void RegisterInstruction(const std::string &name, 
                                     std::function<int32_t(const std::vector<std::string> &)> func) = 0;
    
    // 设置进度回调
    virtual void SetProgressCallback(std::function<void(int32_t)> callback) = 0;
    
    // 设置日志回调
    virtual void SetLogCallback(std::function<void(const std::string &)> callback) = 0;
    
    // 获取实例
    static std::unique_ptr<ScriptManager> Create();
};
```

### 内置指令

| 指令 | 说明 | 实现位置 |
|------|------|---------|
| `mount` | 挂载分区 | `services/script/script_instruction/script_basicinstruction.cpp` |
| `unmount` | 卸载分区 | 同上 |
| `format` | 格式化分区 | 同上 |
| `write_raw_image` | 写入原始镜像 | `services/script/script_instruction/script_updateprocesser.cpp` |
| `package_extract_file` | 提取包文件 | 同上 |
| `apply_patch` | 应用差分补丁 | 同上 |
| `set_progress` | 设置进度 | `services/script/script_instruction/script_basicinstruction.cpp` |
| `ui_print` | 打印 UI 消息 | 同上 |
| `run_program` | 运行程序 | 同上 |
| `delete` | 删除文件 | 同上 |
| `symlink` | 创建符号链接 | 同上 |

---

## 3. Updater - 升级器核心

**稳定性**: Stable  
**头文件**: `services/include/updater/updater.h`

### 状态枚举

```cpp
enum UpdaterStatus {
    UPDATE_ERROR = -1,           // 通用错误
    UPDATE_SUCCESS,              // 成功
    UPDATE_CORRUPT,              // 包损坏或校验失败
    UPDATE_SKIP,                 // 跳过
    UPDATE_RETRY,                // 可重试
    UPDATE_RETRY_FAIL,           // 重试失败
    UPDATE_SPACE_NOTENOUGH,      // 空间不足
    UPDATE_UNKNOWN
};
```

### 升级模式

```cpp
enum PackageUpdateMode {
    HOTA_UPDATE = 0,       // 标准 OTA
    SDCARD_UPDATE,         // SD 卡升级
    SUBPKG_UPDATE,         // 子包升级
    UNKNOWN_UPDATE,
};
```

### UpdaterParams 结构

```cpp
struct UpdaterParams {
    bool forceUpdate = false;           // 强制升级
    bool forceReboot = false;           // 强制重启
    bool isLoadReduction = false;       // 减负模式
    std::string sdExtMode {};           // SD 扩展模式
    std::string factoryResetMode {};    // 恢复出厂模式
    PackageUpdateMode updateMode = HOTA_UPDATE;
    int retryCount = 0;                 // 重试次数
    int panicCount = 0;                 // 崩溃次数
    pid_t binaryPid = -1;               // 二进制进程 ID
    float initialProgress = 0;          // 初始进度
    float currentPercentage = 0;        // 当前百分比
    unsigned int pkgLocation = 0;       // 包位置
    int64_t allMaxStashSize = -1;       // stash 所需空间
    std::string miscCmd {"boot_updater"}; // Misc 命令
    std::vector<std::string> updateBin {};     // 更新二进制文件
    std::vector<std::string> updatePackage {}; // 升级包列表
    std::function<void(float)> callbackProgress {}; // 进度回调
};
```

### 核心函数

```cpp
// 执行升级
UpdaterStatus DoInstallUpdaterPackage(Hpackage::PkgManager::PkgManagerPtr pkgManager,
                                      UpdaterParams &upParams, PackageUpdateMode updateMode);

// 启动升级进程
UpdaterStatus StartUpdaterProc(Hpackage::PkgManager::PkgManagerPtr pkgManager, UpdaterParams &upParams);

// 检查空间
UpdaterStatus IsSpaceCapacitySufficient(UpdaterParams &upParams);

// 提取升级二进制
int32_t ExtractUpdaterBinary(Hpackage::PkgManager::PkgManagerPtr manager, 
                             std::string &packagePath, const std::string &updaterBinary);

// 设置 AB 槽位参数
UpdaterStatus SetUpdateSlotParam(UpdaterParams &upParams, bool isUpdateCurrSlot);
UpdaterStatus ClearUpdateSlotParam();

// 后处理
void PostUpdater(bool clearMisc);
```

---

## 4. ApplyPatch - 补丁应用

**稳定性**: Stable  
**头文件**: `services/include/applypatch/apply_patch.h`

### 主要函数

```cpp
// 应用补丁到块设备
int ApplyPatch(const std::string &source, const std::string &target, 
               const std::string &patch);

// 应用补丁到文件
int ApplyPatchToFile(const std::string &source, const std::string &target, 
                     const std::string &patch);

// 验证补丁
int VerifyPatch(const std::string &source, const std::string &patch);
```

### TransferManager

```cpp
// services/include/applypatch/transfer_manager.h
class TransferManager {
public:
    int ParseTransfers(const std::vector<std::string> &transferList);
    int ExecuteTransfers(const std::string &source, const std::string &target);
    int VerifyTransfers(const std::string &target);
};
```

---

## 5. FSManager - 文件系统管理

**稳定性**: Stable  
**头文件**: `services/include/fs_manager/mount.h`, `services/include/fs_manager/partitions.h`

### 挂载管理

```cpp
// services/include/fs_manager/mount.h

// 挂载分区
int MountPartition(const std::string &partition, const std::string &mountPoint, 
                   const std::string &fsType, uint64_t mountFlags);

// 卸载分区
int UmountPartition(const std::string &mountPoint);

// 检查挂载状态
bool IsPartitionMounted(const std::string &mountPoint);
```

### 分区操作

```cpp
// services/include/fs_manager/partitions.h

// 格式化分区
int FormatPartition(const std::string &partition, const std::string &fsType);

// 擦除分区
int WipePartition(const std::string &partition);

// 获取分区大小
uint64_t GetPartitionSize(const std::string &partition);

// 检查分区存在
bool IsPartitionExist(const std::string &partition);
```

---

## 6. Log - 日志系统

**稳定性**: Stable  
**头文件**: `services/include/log/log.h`, `services/include/log/updater_hilog.h`

### 日志级别

```cpp
enum LogLevel {
    LOG_LEVEL_DEBUG = 0,
    LOG_LEVEL_INFO,
    LOG_LEVEL_WARN,
    LOG_LEVEL_ERROR,
    LOG_LEVEL_FATAL,
};
```

### 日志宏

```cpp
// services/include/log/log.h
#define LOGD(fmt, ...) LogMessage(LOG_LEVEL_DEBUG, __FILE__, __LINE__, fmt, ##__VA_ARGS__)
#define LOGI(fmt, ...) LogMessage(LOG_LEVEL_INFO, __FILE__, __LINE__, fmt, ##__VA_ARGS__)
#define LOGW(fmt, ...) LogMessage(LOG_LEVEL_WARN, __FILE__, __LINE__, fmt, ##__VA_ARGS__)
#define LOGE(fmt, ...) LogMessage(LOG_LEVEL_ERROR, __FILE__, __LINE__, fmt, ##__VA_ARGS__)
#define LOGF(fmt, ...) LogMessage(LOG_LEVEL_FATAL, __FILE__, __LINE__, fmt, ##__VA_ARGS__)

// 带 LastWord 的日志（崩溃时写入）
#define UPDATER_LAST_WORD(errCode, ...) \
    UpdaterLastWord(__FILE__, __LINE__, errCode, ##__VA_ARGS__)
```

### HiLog 集成

```cpp
// services/include/log/updater_hilog.h
#define UPDATER_LOGI(...) HILOG_INFO(LOG_CORE, __VA_ARGS__)
#define UPDATER_LOGE(...) HILOG_ERROR(LOG_CORE, __VA_ARGS__)
```

---

## 7. UI - 用户界面

**稳定性**: Internal  
**头文件**: `services/ui/`

### UI 状态

```cpp
// services/ui/include/updater_ui_const.h
enum class UpdaterUiStatus {
    NONE = 0,
    INITIALIZING,
    READY,
    PROCESSING,
    SUCCESS,
    FAILED,
    CANCELLED,
};
```

### UI 回调

```cpp
// 设置进度回调
void SetProgressCallback(std::function<void(int)> callback);

// 设置日志回调
void SetLogCallback(std::function<void(const std::string &)> callback);

// 设置结果回调
void SetResultCallback(std::function<void(int)> callback);
```

---

## 8. Utils - 工具函数

**稳定性**: Stable  
**头文件**: `utils/include/utils.h`

### 文件操作

```cpp
// utils/include/utils.h

// 读取文件到字符串
bool ReadFileToString(const std::string &file, std::string &content);

// 写入字符串到文件
bool WriteStringToFile(const std::string &file, const std::string &content);

// 复制文件
bool CopyFile(const std::string &src, const std::string &dst);

// 复制目录
bool CopyDir(const std::string &src, const std::string &dst);

// 删除文件
bool DeleteFile(const std::string &file);

// 获取文件大小
uint64_t GetFileSize(const std::string &file);
```

### 系统操作

```cpp
// 重启到指定模式
int UpdaterDoReboot(const std::string &cmd);

// 关闭系统
int DoShutdown();

// 卸载用户数据分区
int UmountUserdata();

// 检查是否在 Updater 模式
bool IsUpdaterMode();
```

### AB 分区操作

```cpp
// 设置更新槽
UpdaterStatus SetUpdateSlot();

// 获取更新槽
UpdaterStatus GetUpdateSlot();

// 设置更新后缀
UpdaterStatus SetUpdateSuffix();

// 获取更新后缀
UpdaterStatus GetUpdateSuffix();

// 检查是否为 VAB 设备
bool IsVabDevice();
```

### Misc 操作

```cpp
// 设置消息到 Misc
bool SetMessageToMisc(const std::string &path, const std::string &command, 
                      const std::string &update);

// 设置命令到 Misc
bool SetCmdToMisc(const std::string &path, const std::string &command);

// 设置故障信息
bool SetFaultInfoToMisc(const std::string &info);

// 检查故障信息
bool CheckFaultInfo();
```

---

## 9. Flashd - 工厂刷机

**稳定性**: Stable  
**头文件**: `services/include/flashd/flashd.h`

### 刷机命令

```cpp
// 格式化分区
int FormatPartition(const std::string &partition);

// 擦除分区
int ErasePartition(const std::string &partition);

// 刷写镜像
int FlashImage(const std::string &partition, const std::string &imagePath);

// 刷写 ZIP 包
int FlashZip(const std::string &zipPath);
```

---

## 接口依赖图

```mermaid
graph TD
    A[ScriptManager] -->|调用| B[PkgManager]
    A -->|调用| C[FSManager]
    A -->|调用| D[ApplyPatch]
    
    E[Updater] -->|使用| A
    E -->|使用| B
    E -->|使用| F[Utils]
    
    G[UI] -->|回调| E
    H[Log] -->|所有模块使用
```

## 关键结论

1. **PkgManager 是核心**: 包管理器提供完整的包生命周期管理，是升级流程的基础。

2. **ScriptManager 驱动流程**: 升级逻辑通过脚本引擎执行，支持灵活的升级策略。

3. **UpdaterParams 传递状态**: 升级参数结构体贯穿整个升级流程，包含所有状态信息。

4. **Utils 提供基础设施**: 工具函数提供文件、系统、AB 分区等基础操作。

5. **Log 系统集成**: 支持本地日志和 HiLog，支持 LastWord 崩溃信息记录。

## 相关跳转

- [目录结构](./02_Directory_Structure.md)
- [对外 API](./03_Public_API.md)
- [架构说明](./01_Architecture.md)
- [调用链附录](./appendix/Callgraphs.md)
