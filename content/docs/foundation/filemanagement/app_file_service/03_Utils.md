# 工具库（Utils）

## 4.1 工具库概述

工具库是应用文件服务的基础支撑模块，提供通用的功能组件供其他层次复用。工具库采用模块化设计，每个模块有明确的职责边界和依赖关系。工具库全部以静态库或头文件的方式提供，不直接产生可执行产物。

### 4.1.1 模块列表

工具库包含以下 17 个功能模块：

| 模块 | 路径 | 职责 |
|------|------|------|
| b_anony | `utils/include/b_anony/` | 字符串/路径匿名化 |
| b_encryption | `utils/include/b_encryption/` | 加密校验和计算 |
| b_error | `utils/include/b_error/` | 错误码和异常处理 |
| b_filesystem | `utils/include/b_filesystem/` | 文件系统操作 |
| b_hiaudit | `utils/include/b_hiaudit/` | 审计日志 |
| b_hilog | `utils/include/b_hilog/` | 日志宏定义 |
| b_json | `utils/include/b_json/` | JSON 实体类 |
| b_jsonutil | `utils/include/b_jsonutil/` | JSON 工具函数 |
| b_ohos | `utils/include/b_ohos/` | OHOS 系统参数 |
| b_process | `utils/include/b_process/` | 进程管理 |
| b_radar | `utils/include/b_radar/` | 性能统计 |
| b_resources | `utils/include/b_resources/` | 常量定义 |
| b_sa | `utils/include/b_sa/` | 系统能力工具 |
| b_tarball | `utils/include/b_tarball/` | tar 归档操作 |
| b_utils | `utils/include/b_utils/` | 通用工具 |
| b_ipc | `utils/src/b_ipc/` | IPC 工具（内部） |

### 4.1.2 依赖关系

```
b_resources (基础常量)
    │
    ├── b_hilog (日志)
    │       │
    │       └── 所有模块都依赖
    │
    ├── b_error (错误处理)
    │       │
    │       ├── b_excep_utils (异常工具)
    │       │
    │       ├── b_process (进程管理)
    │       │       │
    │       │       ├── b_guard_signal (信号防护)
    │       │       ├── b_guard_cwd (CWD 防护)
    │       │       └── b_multiuser (多用户)
    │       │
    │       ├── b_json (JSON 实体)
    │       │       │
    │       │       └── b_json_entity_* (各类实体)
    │       │
    │       ├── b_tarball (归档操作)
    │       │       │
    │       │       └── b_filesystem (文件系统)
    │       │
    │       ├── b_radar (性能统计)
    │       │       │
    │       │       └── b_utils (通用工具)
    │       │
    │       └── b_filesystem (文件系统)
    │               │
    │               ├── b_anony (匿名化)
    │               ├── b_jsonutil (JSON 工具)
    │               └── b_utils (通用工具)
    │
    ├── b_sa (SA 工具)
    │       └── 无外部依赖
    │
    ├── b_anony (匿名化)
    │       └── 无外部依赖
    │
    ├── b_encryption (加密)
    │       └── 无外部依赖
    │
    ├── b_hiaudit (审计日志)
    │       └── b_resources (常量)
    │
    ├── b_ohos (系统参数)
    │       └── b_error, b_resources
    │
    └── b_utils (通用工具)
            └── b_resources
```

## 4.2 错误处理模块（b_error）

### 4.2.1 模块概述

b_error 是工具库的核心模块，提供统一的错误码定义和异常处理机制。该模块定义了应用文件服务使用的所有错误码，并提供 BError 异常类和异常捕获工具函数。

### 4.2.2 头文件

| 文件 | 说明 |
|------|------|
| `utils/include/b_error/b_error.h` | BError 异常类定义 |
| `utils/include/b_error/b_excep_utils.h` | 异常捕获工具 |

### 4.2.3 错误码分类

```cpp
// utils/include/b_error/b_error.h

class BError : public std::runtime_error {
public:
    enum Codes {
        // UTILS 错误 (0x1000-0x1999)
        UTILS_ERROR = 0x1000,
        INVALID_ARGUMENT = 0x1001,
        IO_ERROR = 0x1002,
        PERMISSION_DENIED = 0x1003,
        NOT_FOUND = 0x1004,
        ALREADY_EXISTS = 0x1005,
        // ... 更多错误码
        
        // TOOL 错误 (0x2000-0x2999)
        TOOL_ERROR = 0x2000,
        CMD_EXEC_FAILED = 0x2001,
        
        // SA 错误 (0x3000-0x3999)
        SA_ERROR = 0x3000,
        SA_LOAD_FAILED = 0x3001,
        SA_NOT_READY = 0x3002,
        
        // SDK 错误 (0x4000-0x4999)
        SDK_ERROR = 0x4000,
        SESSION_NOT_INIT = 0x4001,
        
        // EXT 错误 (0x5000-0x5999)
        EXT_ERROR = 0x5000,
        EXT_CONNECT_FAILED = 0x5001,
        EXT_TIMEOUT = 0x5002,
    };
};
```

### 4.2.4 主要 API

**BError 构造**：
```cpp
BError(int32_t code, const std::string& message = "");
BError(int32_t code, const std::exception& e);
```

**错误码检查**：
```cpp
// 检查是否为指定错误码
bool IsErrorCode(int32_t code) const;

// 获取错误码
int32_t GetCode() const;

// 获取错误消息
const std::string& GetMessage() const;
```

**异常捕获工具**：
```cpp
// 捕获异常并记录日志
void ExceptionCatcherLocked(std::function<void()> func,
                            const std::string& funcName);
```

## 4.3 文件系统模块（b_filesystem）

### 4.3.1 模块概述

b_filesystem 提供文件系统操作功能，包括文件读写、目录遍历、路径处理、文件哈希等。该模块封装了 POSIX 文件操作接口，提供更安全易用的 C++ 接口。

### 4.3.2 头文件

| 文件 | 说明 |
|------|------|
| `utils/include/b_filesystem/b_file.h` | BFile 类 |
| `utils/include/b_filesystem/b_dir.h` | BDir/DirScanner 类 |
| `utils/include/b_filesystem/b_file_hash.h` | BackupFileHash 类 |

### 4.3.3 主要类

**BFile 文件操作类**：
```cpp
class BFile {
public:
    // 打开文件
    static bool Open(const std::string& path, int flags, int& fd);
    static bool OpenRead(const std::string& path, int& fd);
    static bool OpenWrite(const std::string& path, int& fd);
    
    // 读取文件
    static bool Read(int fd, void* buf, size_t size, ssize_t& bytesRead);
    
    // 写入文件
    static bool Write(int fd, const void* buf, size_t size, ssize_t& bytesWritten);
    
    // 发送文件（零拷贝）
    static bool SendFile(int outFd, int inFd, off_t* offset, size_t count);
    
    // 关闭文件
    static bool Close(int& fd);
    
    // 获取文件大小
    static bool GetFileSize(int fd, size_t& size);
    static bool GetFileSize(const std::string& path, size_t& size);
};
```

**BDir 目录操作类**：
```cpp
class BDir {
public:
    // 打开目录
    static bool Open(const std::string& path, DIR*& dir);
    
    // 关闭目录
    static bool Close(DIR*& dir);
    
    // 遍历目录项
    static bool GetNextFile(DIR* dir, const std::string& path,
                           std::vector<std::string>& files);
    
    // 扫描所有子目录
    static bool ScanAllDirs(const std::string& path,
                           std::vector<std::string>& dirs);
};
```

**BackupFileHash 哈希类**：
```cpp
class BackupFileHash {
public:
    // 计算文件 SHA256 哈希
    static std::string HashWithSHA256(const std::string& filePath);
    
    // 计算数据哈希
    static std::string HashWithSHA256(const uint8_t* data, size_t len);
};
```

## 4.4 JSON 模块（b_json/b_jsonutil）

### 4.4.1 模块概述

b_json 提供 JSON 实体类封装，用于解析和构造备份配置相关的 JSON 数据。b_jsonutil 提供 JSON 工具函数，包括 Bundle 信息解析、JSON 构建等功能。

### 4.4.2 JSON 实体类

**基础实体类**：
```cpp
class BJsonEntity {
public:
    virtual ~BJsonEntity() = default;
    
    // 从 JSON 字符串解析
    virtual bool Unmarshall(const std::string& json);
    
    // 序列化为 JSON 字符串
    virtual std::string Marshall() const;
    
    // 从文件加载
    virtual bool LoadFromFile(const std::string& path);
    
    // 保存到文件
    virtual bool SaveToFile(const std::string& path) const;
};
```

**备份能力实体**（BJsonEntityCaps）：
- 解析 `caps.json` 配置文件
- 包含支持的 Bundle 列表和能力信息

**扩展配置实体**（BJsonEntityExtensionConfig）：
- 解析扩展备份配置
- 包含过滤规则和备份排除列表

**Manifest 实体**（BReportEntity）：
- 解析备份清单文件
- 跟踪备份进度和文件状态

### 4.4.3 JSON 工具函数

```cpp
class BJsonUtil {
public:
    // 构建 Bundle 信息
    static std::vector<BundleDetailInfo> BuildBundleInfos(
        const std::string& jsonData);
    
    // 解析 Bundle 数据大小
    static std::vector<BundleDataSize> ParseDataSize(
        const std::string& jsonData);
    
    // 构建设置信息
    static BundleSettingInfo BuildSettingInfo(
        const std::string& jsonData);
};
```

## 4.5 日志模块（b_hilog/b_hiaudit）

### 4.5.1 b_hilog 日志宏

```cpp
// 日志宏定义（header-only）
HILOGF(fmt, ...)  // Fatal 级别
HILOGE(fmt, ...)  // Error 级别
HILOGW(fmt, ...)  // Warning 级别
HILOGI(fmt, ...)  // Info 级别
HILOGD(fmt, ...)  // Debug 级别

// 示例
HILOGI("Backup started for bundle: %{public}s", bundleName.c_str());
HILOGE("Failed to open file: %{public}s, error: %{public}d",
       path.c_str(), errno);
```

### 4.5.2 b_hiaudit 审计日志

```cpp
class HiAudit {
public:
    // 获取单例
    static HiAudit& GetInstance();
    
    // 写入审计日志
    bool Write(const AuditLog& log);
    
    // 轮转日志文件
    void Rotate();
    
    // 压缩历史日志
    void Compress();
};
```

## 4.6 归档模块（b_tarball）

### 4.6.1 模块概述

b_tarball 提供 tar 归档文件的创建和提取功能。该模块封装了 tar 命令行工具，提供安全的归档操作接口。

### 4.6.2 主要类

```cpp
class BTarballFactory {
public:
    struct Impl {
        std::function<int(const std::vector<std::string>&,
                         const std::string&)> tar;
        std::function<int(const std::string&,
                         const std::string&)> untar;
    };
    
    // 创建归档
    int Tar(const std::vector<std::string>& files,
            const std::string& outputPath);
    
    // 解压归档
    int Untar(const std::string& archivePath,
              const std::string& outputDir);
};
```

## 4.7 性能统计模块（b_radar）

### 4.7.1 模块概述

b_radar 提供备份恢复操作的性能统计功能，包括操作耗时、成功失败统计、资源占用等信息。

### 4.7.2 主要类

```cpp
class AppRadar {
public:
    // 获取单例
    static AppRadar& GetInstance();
    
    // 记录操作结果
    void RecordBackupFuncRes(const std::string& funcName,
                            int32_t result);
    
    // 记录阶段变更
    void RecordBizStage(BizStageBackup stage);
};

class RadarAppStatistic {
public:
    // 更新应用统计
    void UpdateBundleStats(const std::string& bundleName,
                         uint64_t dataSize,
                         uint64_t timeCost);
};
```

## 4.8 系统能力模块（b_sa）

### 4.8.1 主要功能

```cpp
class SAUtils {
public:
    // 检查是否为 SA Bundle 名称
    static bool IsSABundleName(const std::string& bundleName);
    
    // 检查备份权限
    static int CheckBackupPermission();
    
    // 检查通用权限
    static int CheckPermission(const std::string& permission);
    
    // 检查是否为系统应用
    static bool IsSystemApp(int uid);
};
```

## 4.9 常量模块（b_resources）

### 4.9.1 主要常量

```cpp
// 路径常量
const std::string BACKUP_ROOT_PATH = "/data/service/el2/100/backup/";
const std::string BACKUP_TEMP_PATH = "/data/el2/ tempfile/";
const std::string BACKUP_DATA_PATH = "/data/backup/";

// 大小常量
const size_t BIG_FILE_BOUNDARY = 2 * 1024 * 1024;  // 2MB

// 超时常量
const uint32_t DEFAULT_TIMEOUT = 15 * 60 * 1000;     // 15分钟
const uint32_t EXT_CONNECT_MAX_TIME = 25000;          // 25秒
```

## 4.10 公共构建配置

**文件**：`utils/BUILD.gn`

```gn
ohos_shared_library("backup_utils") {
    sources = [
        "src/b_anony/",
        "src/b_encryption/",
        "src/b_error/",
        "src/b_filesystem/",
        "src/b_hiaudit/",
        "src/b_json/",
        "src/b_jsonutil/",
        "src/b_ohos/",
        "src/b_process/",
        "src/b_radar/",
        "src/b_sa/",
        "src/b_tarball/",
        "src/b_utils/",
        "src/b_ipc/",
    ]
    
    public_configs = [ ":utils_public_config" ]
    private_configs = [ ":utils_private_config" ]
    
    deps = [
        ":backup_cxx_cppdeps",
        "//foundation/filemanagement/app_file_service/interfaces/innerkits/native:sandbox_helper_native",
    ]
    
    external_deps = [
        "libaccesstoken_sdk",
        "libtokenid_sdk",
        "cjson",
        "c_utils",
        "libdfx_dumpcatcher",
        "libhilog",
        "libhisysevent:hisysevent_inner",
        "hitrace:hitrace_meter",
        "init:libbegetutil",
        "ipc:ipc_core",
        "jsoncpp:jsoncpp",
        "openssl:libcrypto_shared",
        "zlib:shared_libz",
    ]
}
```

## 4.11 相关文档

| 文档 | 说明 |
|------|------|
| [系统架构](01_Architecture.md) | 工具库在架构中的位置 |
| [备份服务 SA](02_Service_SA.md) | 服务层如何使用工具库 |
| [GN 构建配置](04_GN_Build.md) | 构建配置详情 |
| [项目概览](00_Overview.md) | 工具库目录结构 |
| [安全评审](05_Security_Review.md) | 工具库安全风险 |
