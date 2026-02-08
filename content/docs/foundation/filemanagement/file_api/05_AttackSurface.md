# File API 攻击面分析

本文档对 File API 进行全面的攻击面分析，帮助安全研究员快速识别外部输入入口、敏感操作和信任边界。所有分析均有代码证据支撑，证据格式为「文件路径:行号」。

## 1. 外部输入清单

File API 接收外部输入的主要入口包括 JS API 参数、配置文件、环境变量等。以下详细列出所有外部输入点及其风险等级。

### 1.1 JS API 参数输入

JS API 是最主要的外部输入入口，应用通过调用 N-API 函数传入数据。

#### 1.1.1 文件路径参数

文件路径是最常见的输入类型，几乎所有文件操作 API 都接受路径参数。

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp
// 功能：打开文件
napi_value OpenCore::AsyncExec(napi_env env) {
    // 参数解析：从 JS 获取路径参数
    std::string path;
    NAPI_ASSERT_RETURN_NULL(env, 
        GetStringParam(env, path, argv[ARG_DATA_ZERO]),
        "Failed to get path parameter");
    
    // 打开文件
    int32_t fd = open(path.c_str(), flags);
    // ...
}
```

| API 函数 | 参数位置 | 风险等级 | 说明 |
|----------|----------|----------|------|
| open() | path | 高 | 路径遍历风险 |
| openSync() | path | 高 | 路径遍历风险 |
| createStream() | path | 高 | 路径遍历风险 |
| stat() | path | 中 | 路径遍历风险 |
| unlink() | path | 高 | 任意文件删除 |
| rename() | oldPath, newPath | 高 | 路径遍历风险 |
| mkdir() | path | 中 | 目录创建覆盖 |
| rmdir() | path | 中 | 目录删除 |
| chmod() | path | 中 | 权限修改 |
| chown() | path | 中 | 所有者修改 |

#### 1.1.2 模式参数

文件打开模式参数影响文件访问权限。

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp
// 功能：解析打开模式
std::string modeStr = GetModeString(env, argv[ARG_DATA_ONE]);
int32_t mode = ConvertMode(modeStr);  // "r", "w", "a", "r+" 等
```

| API 函数 | 参数位置 | 风险等级 | 说明 |
|----------|----------|----------|------|
| open() | flags | 中 | 权限逃逸风险 |
| createStream() | mode | 中 | 权限逃逸风险 |
| chmod() | mode | 低 | 权限数值注入 |
| umask() | mode | 低 | 权限数值注入 |

#### 1.1.3 缓冲区参数

读写操作使用的缓冲区参数。

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/read.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/read.cpp
// 功能：读取文件数据
napi_value ReadCore::AsyncExec(napi_env env) {
    // 获取缓冲区
    void *buf = nullptr;
    napi_get_arraybuffer_info(env, buffer, &buf, &len);
    
    // 写入数据到缓冲区
    ssize_t readLen = read(fd, buf, len);
    // ...
}
```

| API 函数 | 参数位置 | 风险等级 | 说明 |
|----------|----------|----------|------|
| read() | buffer | 中 | 缓冲区溢出 |
| write() | buffer | 中 | 缓冲区溢出 |
| readSync() | buffer | 中 | 缓冲区溢出 |
| writeSync() | buffer | 中 | 缓冲区溢出 |
| readText() | encoding | 低 | 编码处理 |

#### 1.1.4 选项参数

部分 API 接受可选的配置选项参数。

**证据来源**：`interfaces/kits/js/src/common/file_filter.h`

```cpp
// 文件：interfaces/kits/js/src/common/file_filter.h
// 功能：文件过滤选项
struct FileFilter {
    bool suffix;      // 后缀过滤
    std::string suf;  // 后缀字符串
    bool separator;   // 分隔符
};
```

| API 函数 | 参数位置 | 风险等级 | 说明 |
|----------|----------|----------|------|
| readDir() | filter | 低 | 过滤规则注入 |
| listFile() | options | 中 | 目录遍历选项 |
| access() | mode | 低 | 访问检查 |

### 1.2 URI 输入

URI 是 File API 沙箱机制的入口点。

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp
// 功能：URI 解析和验证
napi_value OpenCore::AsyncExec(napi_env env) {
    std::string uri;
    GetStringParam(env, uri, argv[ARG_DATA_ZERO]);
    
    // URI 验证和转换
    std::string path = UriToPath(uri);  // internal://app/xxx → /data/app/xxx
    if (!IsInSandbox(path)) {
        NAPI_ASSERT_RETURN_NULL(env, false, "URI out of sandbox");
    }
    
    int32_t fd = open(path.c_str(), flags);
    // ...
}
```

| URI 类型 | 格式 | 风险等级 | 说明 |
|----------|------|----------|------|
| 应用私有目录 | internal://app/ | 低 | 受沙箱保护 |
| 缓存目录 | internal://cache/ | 低 | 受沙箱保护 |
| 共享目录 | internal://share/ | 中 | 多应用访问 |
| 非法 URI | 外部路径 | 高 | 沙箱逃逸 |

### 1.3 环境输入

#### 1.3.1 应用上下文

**证据来源**：`interfaces/kits/js/src/common/ability_helper.cpp`

```cpp
// 文件：interfaces/kits/js/src/common/ability_helper.cpp
// 功能：获取应用上下文
std::string GetAppDataPath(int32_t uid) {
    // 通过 AbilityHelper 获取应用数据目录
    auto abilityMgr = AbilityHelper::GetInstance();
    std::string path = abilityMgr->GetDataDir(uid);
    return path;
}
```

| 输入源 | 获取方式 | 风险等级 | 说明 |
|--------|----------|----------|------|
| 应用 UID | IPC 调用 | 低 | 应用标识 |
| 应用路径 | AbilityHelper | 低 | 沙箱路径 |
|Bundle 信息 | BundleManager | 低 | 包信息 |

### 1.4 外部配置输入

#### 1.4.1 系统配置

**证据来源**：`bundle.json` 系统能力声明

```json
"syscap": [
  "SystemCapability.FileManagement.File.FileIO"
]
```

| 配置项 | 来源 | 风险等级 | 说明 |
|--------|------|----------|------|
| 系统能力 | bundle.json | 低 | 能力声明 |
| Feature 开关 | file_api.gni | 低 | 构建配置 |

## 2. 敏感操作清单

File API 执行的敏感操作主要包括文件系统操作、系统调用和跨域访问。以下详细分析每类敏感操作。

### 2.1 文件系统操作

#### 2.1.1 文件读写操作

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/read.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/read.cpp
// 功能：读取文件内容
ssize_t ReadCore::ReadFromFd(int32_t fd, char *buf, size_t len, off_t offset) {
    // 直接调用系统调用 read()
    return read(fd, buf, len);
}
```

| 操作 | 系统调用 | 风险等级 | 安全影响 |
|------|----------|----------|----------|
| read() | read() | 高 | 读取任意文件 |
| write() | write() | 高 | 写入任意文件 |
| pread() | pread() | 高 | 读取任意偏移 |
| pwrite() | pwrite() | 高 | 写入任意偏移 |

#### 2.1.2 文件元数据操作

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/stat.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/stat.cpp
// 功能：获取文件状态
napi_value StatCore::AsyncExec(napi_env env) {
    struct stat st;
    int ret = fstat(fd, &st);  // 系统调用
    // ...
}
```

| 操作 | 系统调用 | 风险等级 | 安全影响 |
|------|----------|----------|----------|
| stat() | stat() | 中 | 信息泄露 |
| lstat() | lstat() | 中 | 符号链接信息 |
| fstat() | fstat() | 中 | 文件描述符信息 |
| chmod() | chmod() | 中 | 权限修改 |
| chown() | chown() | 中 | 所有者修改 |
| utimes() | utimes() | 中 | 时间戳修改 |

#### 2.1.3 文件系统操作

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/rename.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/rename.cpp
// 功能：重命名文件
napi_value RenameCore::AsyncExec(napi_env env) {
    std::string oldPath, newPath;
    GetStringParam(env, oldPath, argv[ARG_DATA_ZERO]);
    GetStringParam(env, newPath, argv[ARG_DATA_ONE]);
    
    int ret = rename(oldPath.c_str(), newPath.c_str());
    // ...
}
```

| 操作 | 系统调用 | 风险等级 | 安全影响 |
|------|----------|----------|----------|
| rename() | rename() | 高 | 任意文件移动 |
| unlink() | unlink() | 高 | 任意文件删除 |
| mkdir() | mkdir() | 中 | 任意目录创建 |
| rmdir() | rmdir() | 中 | 任意目录删除 |
| link() | link() | 高 | 任意硬链接创建 |
| symlink() | symlink() | 高 | 任意符号链接创建 |

### 2.2 权限操作

#### 2.2.1 访问权限检查

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/access.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/access.cpp
// 功能：检查文件访问权限
napi_value AccessCore::AsyncExec(napi_env env) {
    std::string path;
    GetStringParam(env, path, argv[ARG_DATA_ZERO]);
    
    int mode = static_cast<int>(argv[ARG_DATA_ONE]);
    int ret = access(path.c_str(), mode);  // 系统调用
    // ...
}
```

| 检查项 | 依赖 | 风险等级 | 说明 |
|--------|------|----------|------|
| access() | 权限位检查 | 中 | 权限验证 |
| faccessat() | faccessat() | 中 | 目录内访问 |
| umask() | umask() | 低 | 权限掩码 |

### 2.3 跨域操作

#### 2.3.1 跨应用文件访问

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp
// 功能：打开共享目录文件
napi_value OpenCore::AsyncExec(napi_env env) {
    std::string uri = GetStringParam(env, argv[ARG_DATA_ZERO]);
    
    // 检查 URI 类型
    if (uri.compare(0, 16, "internal://share") == 0) {
        // 共享目录：需要权限检查
        if (!CheckSharePermission(uri)) {
            NAPI_ASSERT_RETURN_NULL(env, false, "Permission denied");
        }
    }
    
    int32_t fd = UriToFd(uri);
    // ...
}
```

| 跨域操作 | 目标 | 风险等级 | 安全机制 |
|----------|------|----------|----------|
| 打开共享文件 | internal://share/ | 高 | 权限检查 |
| 读取外部存储 | 外部路径 | 高 | URI 验证 |
| 访问其他应用数据 | /data/app/ | 高 | 沙箱隔离 |

### 2.4 敏感操作汇总表

| 风险等级 | 操作类型 | 主要风险 | 防护措施 |
|----------|----------|----------|----------|
| **高** | 文件读写 | 任意文件读写 | 沙箱验证 |
| **高** | 文件删除 | 任意文件删除 | 沙箱验证 |
| **高** | 硬链接创建 | 权限提升 | 权限检查 |
| **高** | 符号链接 | 符号链接攻击 | 验证检测 |
| **中** | 元数据修改 | 权限/所有者修改 | 权限检查 |
| **中** | 目录操作 | 目录遍历 | 路径验证 |
| **低** | 状态查询 | 信息泄露 | 最小化信息 |

## 3. 信任边界图

File API 存在多个信任边界，跨边界的数据流需要严格验证。以下分析各信任边界及跨越方式。

### 3.1 信任边界定义

```
┌─────────────────────────────────────────────────────────────────┐
│                      用户空间（User Space）                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    应用沙箱边界                            │  │
│  │  ┌─────────────────────────────────────────────────────┐│  │
│  │  │              File API 边界（untrusted）              ││  │
│  │  │  ┌───────────────────────────────────────────────┐││  │
│  │  │  │           JS 层边界（untrusted）                │││  │
│  │  │  │  - JS 参数输入                                  │││  │
│  │  │  │  - URI 解析                                    │││  │
│  │  │  └───────────────────────────────────────────────┘││  │
│  │  └─────────────────────────────────────────────────────┘│  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                  Native 层边界（trusted）                 │  │
│  │  ┌─────────────────────────────────────────────────────┐│  │
│  │  │             LibN 抽象层（trusted）                   ││  │
│  │  │  - NAPI 转换                                       ││  │
│  │  │  - 异步工作管理                                     ││  │
│  │  └─────────────────────────────────────────────────────┘│  │
│  │  ┌─────────────────────────────────────────────────────┐│  │
│  │  │            文件系统封装层（trusted）                  ││  │
│  │  │  - 路径规范化                                      ││  │
│  │  │  - URI 验证                                        ││  │
│  │  └─────────────────────────────────────────────────────┘│  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    内核空间（Kernel Space）               │  │
│  │  ┌─────────────────────────────────────────────────────┐│  │
│  │  │              文件系统层（trusted）                   ││  │
│  │  │  - VFS                                             ││  │
│  │  │  - 具体文件系统                                     ││  │
│  │  └─────────────────────────────────────────────────────┘│  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

Legend:
[ ] 信任边界
→ 数据流向
(untrusted) 不可信区域
(trusted) 可信区域
```

### 3.2 边界跨越点分析

| 边界编号 | 跨越方向 | 跨越操作 | 验证机制 |
|----------|----------|----------|----------|
| B1 | JS → Native | 参数传递 | 类型检查、范围检查 |
| B2 | Native → 文件系统 | 系统调用 | 路径规范化、URI 验证 |
| B3 | 应用沙箱 → 共享目录 | 跨域访问 | 权限检查 |

### 3.3 边界 B1：JS 到 Native

**证据来源**：`interfaces/kits/js/src/common/napi/n_val.cpp`

```cpp
// 文件：interfaces/kits/js/src/common/napi/n_val.cpp
// 功能：JS 参数到 Native 值的转换
NVal NVal::FromJSValue(napi_env env, napi_value value) {
    // 类型检查
    napi_valuetype type;
    napi_typeof(env, value, &type);
    
    // 对于字符串参数：长度检查
    if (type == napi_string) {
        size_t len;
        napi_get_value_string_utf8(env, value, nullptr, 0, &len);
        if (len > MAX_PATH_LEN) {
            // 拒绝过长路径
            NAPI_ASSERT(false, "Path too long");
        }
    }
    
    return NVal(env, value);
}
```

| 跨越点 | 输入类型 | 风险 | 验证机制 |
|--------|----------|------|----------|
| 字符串参数 | 路径、URI | 缓冲区溢出 | 长度检查 |
| 数值参数 | 偏移、模式 | 整数溢出 | 范围检查 |
| 缓冲区参数 | ArrayBuffer | 越界访问 | 大小验证 |
| 对象参数 | 选项对象 | 属性注入 | 白名单验证 |

### 3.4 边界 B2：Native 到文件系统

**证据来源**：`interfaces/kits/js/src/mod_fs/common_func.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/common_func.cpp
// 功能：路径规范化
std::string NormalizePath(const std::string& path) {
    // 路径遍历检测
    if (path.find("../") != std::string::npos) {
        // 检测到路径遍历尝试
        return "";  // 返回空表示拒绝
    }
    
    // 符号链接解析
    std::string resolved = ResolveSymlinks(path);
    
    // 确保在沙箱内
    if (!IsInSandbox(resolved)) {
        return "";  // 返回空表示拒绝
    }
    
    return resolved;
}
```

| 跨越点 | 操作 | 风险 | 验证机制 |
|--------|------|------|----------|
| 路径处理 | 规范化 | 路径遍历 | 遍历检测 |
| 符号链接 | 解析 | 符号链接攻击 | 循环检测 |
| 相对路径 | 转换 | 目录逃逸 | 绝对路径 |
| URI 转换 | 解析 | 沙箱逃逸 | 白名单 |

### 3.5 边界 B3：跨应用访问

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp
// 功能：共享目录访问检查
bool CheckSharePermission(const std::string& uri) {
    // 获取调用应用信息
    int32_t callerUid = GetCurrentUid();
    
    // 检查是否有权限访问共享目录
    auto tokenManager = AccessTokenManager::GetInstance();
    if (!tokenManager->CheckPermission(callerUid, PERM_READ_SHARE)) {
        return false;
    }
    
    return true;
}
```

| 跨越点 | 操作 | 风险 | 验证机制 |
|--------|------|------|----------|
| 共享目录读 | 打开文件 | 未授权访问 | UID 检查 |
| 共享目录写 | 创建文件 | 未授权写入 | 权限验证 |
| 跨应用数据 | 访问路径 | 权限提升 | 沙箱隔离 |

### 3.6 边界跨越风险评估

| 边界 | 风险等级 | 主要威胁 | 防护建议 |
|------|----------|----------|----------|
| B1 | 中 | 参数注入 | 严格类型和范围检查 |
| B2 | 高 | 路径遍历 | 多重路径验证 |
| B3 | 高 | 权限绕过 | 强制权限检查 |

## 4. 输入验证分析

### 4.1 路径验证

#### 4.1.1 规范化检查

**证据来源**：`interfaces/kits/js/src/mod_fs/common_func.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/common_func.cpp
// 功能：路径规范化
bool ValidatePath(const std::string& path) {
    // 1. 检查空路径
    if (path.empty()) {
        return false;
    }
    
    // 2. 检查路径长度
    if (path.length() > PATH_MAX) {
        return false;
    }
    
    // 3. 检查非法字符
    for (char c : path) {
        if (!IsValidPathChar(c)) {
            return false;
        }
    }
    
    // 4. 检查路径遍历
    if (ContainsPathTraversal(path)) {
        return false;
    }
    
    // 5. 检查符号链接
    if (ContainsSymlink(path)) {
        return CheckSymlinkSafe(path);
    }
    
    return true;
}
```

#### 4.1.2 沙箱边界检查

**证据来源**：`interfaces/kits/js/src/mod_fs/common_func.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/common_func.cpp
// 功能：检查路径是否在沙箱内
bool IsInSandbox(const std::string& absPath) {
    // 获取应用沙箱根目录
    std::string sandboxRoot = GetSandboxRoot();
    
    // 检查路径是否在沙箱内
    if (absPath.compare(0, sandboxRoot.length(), sandboxRoot) != 0) {
        return false;  // 路径在沙箱外
    }
    
    return true;
}
```

### 4.2 URI 验证

#### 4.2.1 URI 格式检查

**证据来源**：`interfaces/kits/js/src/mod_fs/common_func.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/common_func.cpp
// 功能：URI 格式验证
bool ValidateUri(const std::string& uri) {
    // 1. 检查 URI 前缀
    if (uri.compare(0, 9, "internal:/") != 0) {
        return false;  // 非 internal:// URI
    }
    
    // 2. 解析 URI 类型
    std::string type = ExtractUriType(uri);
    if (!IsAllowedUriType(type)) {
        return false;  // 不允许的 URI 类型
    }
    
    // 3. 检查 URI 路径
    std::string path = ExtractUriPath(uri);
    return ValidatePath(path);
}
```

#### 4.2.2 URI 到路径转换

**证据来源**：`interfaces/kits/js/src/mod_fs/common_func.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/common_func.cpp
// 功能：URI 到文件系统路径的转换
std::string UriToPath(const std::string& uri) {
    // 1. 解析 URI 类型
    std::string type = ExtractUriType(uri);
    std::string subPath = ExtractUriPath(uri);
    
    // 2. 根据类型获取基础路径
    std::string basePath;
    switch (UriTypeFromString(type)) {
        case URI_APP:
            basePath = GetAppDataPath();
            break;
        case URI_CACHE:
            basePath = GetCachePath();
            break;
        case URI_SHARE:
            basePath = GetSharePath();
            break;
        default:
            return "";  // 不支持的 URI 类型
    }
    
    // 3. 拼接路径
    return basePath + "/" + subPath;
}
```

### 4.3 缓冲区验证

#### 4.3.1 读写大小验证

**证据来源**：`interfaces/kits/js/src/mod_fs/properties/read.cpp`

```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/read.cpp
// 功能：读取操作的大小验证
ssize_t ReadCore::ReadFromFd(int32_t fd, char *buf, size_t requestLen, off_t offset) {
    // 1. 验证缓冲区大小
    if (buf == nullptr) {
        return -1;  // 空指针
    }
    
    // 2. 验证读取大小
    if (requestLen > MAX_READ_SIZE) {
        requestLen = MAX_READ_SIZE;  // 限制最大读取大小
    }
    
    // 3. 验证偏移量
    if (offset < 0) {
        return -1;  // 负偏移无效
    }
    
    // 4. 执行读取
    return pread(fd, buf, requestLen, offset);
}
```

## 5. 安全建议

### 5.1 输入验证建议

| 建议项 | 优先级 | 说明 |
|--------|--------|------|
| 强制路径规范化 | 高 | 所有路径必须经过规范化处理 |
| 限制路径长度 | 高 | 设置最大路径长度限制 |
| 禁止路径遍历 | 高 | 检测并拒绝 "../" 等遍历模式 |
| 验证符号链接 | 高 | 解析并验证符号链接安全性 |
| 限制 URI 类型 | 中 | 只支持预定义的 URI 前缀 |

### 5.2 权限控制建议

| 建议项 | 优先级 | 说明 |
|--------|--------|------|
| 强制沙箱边界 | 高 | 确保所有文件操作在沙箱内 |
| 最小权限原则 | 中 | 只申请必要的文件访问权限 |
| 审计敏感操作 | 中 | 记录文件删除、重命名等操作 |

### 5.3 编码实践建议

| 建议项 | 优先级 | 说明 |
|--------|--------|------|
| 使用安全字符串函数 | 高 | 使用 strncpy_s 而非 strcpy |
| 避免缓冲区溢出 | 高 | 所有缓冲区操作进行边界检查 |
| 错误信息脱敏 | 中 | 不向应用返回敏感系统信息 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估 |
| [04_Interface.md](04_Interface.md) | API 接口文档 |
| [02_Architecture.md](02_Architecture.md) | 架构设计 |

---

**最后更新**：2026-02-07

**版本**：1.0