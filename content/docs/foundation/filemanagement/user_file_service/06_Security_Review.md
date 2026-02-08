# User File Service - 安全风险评审

## 概述

本文档对 user_file_service 进行安全风险评审，识别潜在攻击面、信任边界和安全风险点。

**评审范围**：
- N-API 接口层 (`frameworks/js/napi/`)
- 服务层 (`services/native/`)
- 内部 API (`interfaces/inner_api/`)
- 工具类 (`utils/`)

**评审依据**：基于代码静态分析（测试代码除外）

---

## 攻击面清单

> **提示**：详细的攻击面分析（含完整入口点清单、数据流图、检查点）请参阅 [05_AttackSurface.md](05_AttackSurface.md)

### 1. N-API 接口攻击面

| 攻击面 | 输入来源 | 风险等级 | 关键入口 |
|--------|----------|----------|----------|
| JS API 参数 | 上层应用 | 高 | `NAPI_OpenFile:312`, `NAPI_CreateFile:361` |
| 文件 URI | 上层应用 | 高 | `napi_fileaccess_helper.cpp` |
| 文件路径 | 上层应用 | 高 | 所有文件操作接口 |
| 文件名 | 上层应用 | 中 | `NAPI_CreateFile`, `NAPI_Rename` |
| 权限令牌 | 系统 | 低 | `recent_n_exporter.cpp:41` |

**证据**：`frameworks/js/napi/file_access_module/napi_fileaccess_helper.cpp`

```cpp
// 第312行：NAPI_OpenFile 入口点
static napi_value NAPI_OpenFile(napi_env env, napi_callback_info info)
{
    // JS 参数通过 napi_get_cb_info 获取
    NAPI_GET_STRING_PARAM(env, args[0], uri, value);  // URI 参数
    NAPI_GET_INT32_PARAM(env, args[1], flags, value); // flags 参数
    // ... 后续处理
}
```

### 2. IPC 接口攻击面

| 攻击面 | 说明 | 风险等级 | 入口点 |
|--------|------|----------|--------|
| SA 接口 | FileAccessService (5010) | 高 | `OnRequest()` |
| 扩展 Ability 连接 | 动态加载的文件访问扩展 | 高 | `ConnectFileExtAbility()` |
| 观察者回调 | 跨进程回调 | 中 | `RegisterNotify()` |

**证据**：`services/native/file_access_service/include/file_access_service.h:223`

```cpp
class FileAccessService final : public SystemAbility, public FileAccessServiceBaseStub {
    // 第227行：权限检查方法
    bool CheckCallingPermission(const std::string &permission);
    
    // IPC 接口处理
    virtual int32_t OnRequest(int code, MessageParcel &data, MessageParcel &reply) override;
};
```

### 3. 文件系统攻击面

| 攻击面 | 说明 | 风险等级 | 相关函数 |
|--------|------|----------|----------|
| 公共文件路径 | `/storage/emulated/0/` | 高 | `OpenFile`, `CreateFile` |
| 沙箱路径 | 应用私有目录 | 中 | 路径映射相关 |
| 云同步路径 | 网络存储 | 中 | `Register()`, `Active()` |

### 4. 权限攻击面

| 攻击面 | 说明 | 风险等级 | 检查点 |
|--------|------|----------|--------|
| 权限校验 | `CheckCallingPermission()` | 中 | `file_access_ext_stub_impl.cpp:38` |
| Token 验证 | AccessToken 验证 | 中 | `ufs_access_token_helper.cpp:63` |
| 签名校验 | 应用签名验证 | 低 | `recent_n_exporter.cpp:54` |

**证据**：`interfaces/inner_api/file_access/src/file_access_ext_stub_impl.cpp:38`

```cpp
bool FileAccessExtStubImpl::CheckCallingPermission(const std::string &permission)
{
    // 第41行：获取调用者 Token
    // 第45行：权限不足时记录错误
    return AccessTokenKit::VerifyAccessToken(tokenId, permission) == PERMISSION_GRANTED;
}
```

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   受信任区域:                                                    │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ • FileAccessService (SA 进程)                          │   │
│   │ • 系统库 (/system/lib*)                                │   │
│   │ • 预置应用 (signature 校验通过)                         │   │
│   │ • 媒体库服务 (medialibrary)                             │   │
│   │ • 外置存储服务 (externalFileManager)                   │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   非信任区域:                                                    │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ • 第三方应用 JS 输入                                     │   │
│   │ • 用户提供的文件 URI/路径                                │   │
│   │ • 网络返回的云盘数据                                     │   │
│   │ • 外部存储设备内容                                       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 数据流

```
第三方应用 (非信任)
       │
       │ JS 参数 (uri, path, name)
       ▼
┌───────────────────────────────────────────────────────────────┐
│  N-API 层 (frameworks/js/napi/)                               │
│  • 参数解析                                                   │
│  • 类型转换                                                   │
│  ⚠️ 风险: 未校验输入                                           │
└───────────────────────────────────────────────────────────────┘
       │
       │ IPC 调用
       ▼
┌───────────────────────────────────────────────────────────────┐
│  FileAccessService (SA 5010)                                 │
│  • 权限校验 (CheckCallingPermission)                          │
│  • URI 规范化                                                 │
│  • 业务逻辑                                                   │
│  ✅ 风险缓解                                                  │
└───────────────────────────────────────────────────────────────┘
       │
       │ 连接扩展
       ▼
┌───────────────────────────────────────────────────────────────┐
│  底层服务 (MediaLibrary, ExternalFileManager)                 │
│  • 实际文件操作                                               │
│  • 权限验证 (MediaLibrary)                                    │
│  ✅ 风险缓解                                                  │
└───────────────────────────────────────────────────────────────┘
```

---

## 安全风险点

### 高风险

#### R1: 路径遍历漏洞（中危）

**位置**：`utils/file_uri_check.h:29`

**证据**：
```cpp
// 第29-45行：路径遍历检查实现
static bool IsFilePathValid(const std::string &filePath)
{
    size_t pos = filePath.find(PATH_INVALID_FLAG1);  // "../"
    while (pos != std::string::npos) {
        if (pos == 0 || filePath[pos - 1] == FILE_SEPARATOR_CHAR) {
            HILOG_ERROR("Relative path is not allowed, path contain ../");
            return false;
        }
        pos = filePath.find(PATH_INVALID_FLAG1, pos + PATH_INVALID_FLAG_LEN);
    }
    // ... 检查 "/.." 后缀
}
```

**触发路径**：
```
JS createFile(uri, "../../../etc/passwd") 
→ NAPI_CreateFile() → uri 解析 
→ IsFilePathValid() → 拦截或放行
→ IPC → 文件操作
```

**潜在绕过点**：
1. **URL 编码绕过**：`%2e%2e%2f` 形式未经解码检查
2. **双重编码绕过**：`%252e%252e%252f` 经多次解码后生效
3. **空字节绕过**：`valid.txt%00/../../../etc/passwd`
4. **规范化绕过**：`....//....//etc/passwd` 等变形

**影响评估**：
| 场景 | 可利用性 | 影响 |
|------|---------|------|
| 标准路径遍历 | 低（已防护） | 文件访问越界 |
| 编码绕过 | 待验证 | 系统文件访问 |

**缓解措施状态**：⚠️ 部分防护，需验证编码绕过

**修复建议**：
```cpp
// 1. 路径规范化后再检查
std::string NormalizePath(const std::string &path) {
    // 1. URL 解码
    // 2. 解析 . 和 ..
    // 3. 转换为绝对路径
}

// 2. 白名单验证
bool IsPathAllowed(const std::string &path) {
    std::string normalized = NormalizePath(path);
    return normalized.rfind(ALLOWED_BASE_PATH, 0) == 0;  // 前缀匹配
}

// 3. 使用 chroot 或命名空间隔离
```

---

#### R2: URI 注入（中危）

**位置**：`interfaces/inner_api/file_access/src/file_access_helper.cpp:107`

**证据**：
```cpp
// URI 格式验证逻辑
int32_t FileAccessHelper::CheckUri(const Uri &uri)
{
    // 第107行起：检查 URI scheme 和格式
    std::string scheme = uri.GetScheme();
    if (scheme != "file" && scheme != "datashare") {
        return E_INVALID_URI;
    }
    // ... 更多检查
}
```

**触发路径**：
```
JS listFile("datashare://../../../system/etc/")
→ NAPI_ListFile() → CheckUri() 
→ IsFilePathValid() → 拦截或放行
→ 文件操作
```

**风险点**：
| 检查项 | 状态 | 说明 |
|--------|------|------|
| Scheme 白名单 | ✅ 已检查 | 仅允许 `file`, `datashare` |
| 路径遍历 | ⚠️ 部分检查 | 依赖 `IsFilePathValid()` |
| 特殊字符 | ❓ 待验证 | 需确认是否过滤所有危险字符 |

**潜在载荷**：
```javascript
// 尝试注入非法 scheme
faHelper.listFile("http://evil.com/malicious");

// 尝试路径遍历
faHelper.listFile("datashare://storage/../../../../etc/passwd");

// 尝试空字节注入
faHelper.listFile("datashare://valid\x00/../../../etc/passwd");
```

**修复建议**：
```cpp
// 1. 严格的 URI 白名单
const std::set<std::string> ALLOWED_SCHEMES = {"file", "datashare"};

// 2. 多层次验证
bool ValidateUri(const std::string &uri) {
    // 1. Scheme 检查
    // 2. 路径规范化
    // 3. 路径遍历检查
    // 4. 特殊字符过滤
    // 5. 白名单前缀匹配
}

// 3. 拒绝服务防护
if (uri.length() > MAX_URI_LENGTH) {
    return E_URI_TOO_LONG;
}
```

---

### 中风险

#### R3: 符号链接攻击（中危）

**位置**：`interfaces/kits/native/recent/recent_n_exporter.cpp:128`

**证据**：
```cpp
// 创建符号链接
symlink(targetPath, linkPath);
```

**风险分析**：

| 风险项 | 描述 | 严重性 |
|--------|------|--------|
| 目标路径未验证 | `targetPath` 可能指向任意位置 | 高 |
| 竞态条件 | 检查与使用之间窗口期 | 中 |
| 符号链接追逐 | 可能追逐到受保护目录外 | 高 |

**触发场景**：
```cpp
// 攻击者控制 targetPath
std::string targetPath = "/storage/emulated/0/../../etc/passwd";
symlink(targetPath, "/storage/recent/link");
// 后续操作可能访问到 /etc/passwd
```

**修复建议**：
```cpp
// 1. 目标路径验证
bool ValidateSymlinkTarget(const std::string& target) {
    // 规范化路径
    std::string normalized = NormalizePath(target);
    // 确保在允许目录内
    return normalized.rfind(ALLOWED_BASE_PATH, 0) == 0;
}

// 2. 使用安全的符号链接处理
int SafeOpen(const std::string& path, int flags) {
    // 使用 O_NOFOLLOW 避免追逐符号链接
    return open(path.c_str(), flags | O_NOFOLLOW);
}

// 3. 验证后使用 (TOCTOU 防护)
int32_t CreateRecentFileWithValidation(const std::string& target, const std::string& link) {
    // 1. 验证目标路径
    if (!ValidateSymlinkTarget(target)) {
        return E_INVALID_PATH;
    }
    
    // 2. 创建符号链接
    if (symlink(target.c_str(), link.c_str()) != 0) {
        return E_SYSCALL_ERROR;
    }
    
    // 3. 验证创建的链接 (防止竞态)
    char resolved[PATH_MAX];
    if (realpath(link.c_str(), resolved) == nullptr) {
        unlink(link.c_str());
        return E_INVALID_LINK;
    }
    
    return ERR_OK;
}
```

---

#### R4: 资源耗尽（中危）

**位置**：`services/native/file_access_service/include/file_access_service.h:336`

**证据**：
```cpp
// 第336行：连接映射，无数量限制
std::unordered_map<std::string, sptr<IFileAccessExtBase>> cMap_;

// 第333行：观察者关系映射
std::unordered_map<std::string, std::shared_ptr<ObserverNode>> relationshipMap_;

// 第350行：App 代理映射
std::unordered_map<size_t, sptr<AgentFileAccessExtConnection>> appProxyMap_;
```

**风险点分析**：

| 资源类型 | 风险描述 | 当前限制 | 建议限制 |
|----------|----------|----------|----------|
| IPC 连接 | `cMap_` 无上限 | ❌ 无限制 | 100 |
| 观察者 | `relationshipMap_` 无上限 | ❌ 无限制 | 1000 |
| App 代理 | `appProxyMap_` 无上限 | ❌ 无限制 | 50 |
| 共享内存 | 大文件传输 | ⚠️ 部分检查 | 100MB |

**触发场景**：
```javascript
// 场景1：注册大量观察者
for (let i = 0; i < 100000; i++) {
    faHelper.registerObserver("uri" + i, callback);
}

// 场景2：大量并发连接
// 多应用同时发起文件操作请求

// 场景3：大文件列表
// 请求包含数百万文件的目录列表
```

**影响评估**：
- **DoS 风险**：内存耗尽导致服务崩溃
- **性能下降**：大量连接导致响应延迟
- **可用性影响**：正常用户请求被拒绝

**修复建议**：
```cpp
// 1. 连接池限制
class FileAccessService {
    static constexpr size_t MAX_CONNECTIONS = 100;
    static constexpr size_t MAX_OBSERVERS = 1000;
    static constexpr size_t MAX_APP_PROXIES = 50;
    
    int32_t AddConnection(const std::string& key, sptr<IFileAccessExtBase> proxy) {
        std::lock_guard<std::mutex> lock(mapMutex_);
        if (cMap_.size() >= MAX_CONNECTIONS) {
            HILOG_ERROR("Connection pool full");
            return E_RESOURCE_EXHAUSTED;
        }
        cMap_[key] = proxy;
        return ERR_OK;
    }
};

// 2. 速率限制
class RateLimiter {
    bool AllowRequest(uint32_t tokenId) {
        // 基于 token 的速率限制
        // 防止单个应用耗尽资源
    }
};

// 3. 请求超时和取消机制
// 长时间运行的操作应有超时保护
```

---

#### R5: 权限校验绕过（中危）

**位置**：`interfaces/kits/native/recent/recent_n_exporter.cpp:41-65`

**证据**：
```cpp
// 第41行：权限检查函数
static bool CheckPermission(const std::string &permission)
{
    // 第47行：获取调用者 Token
    uint32_t tokenCaller = IPCSkeleton::GetCallingTokenID();
    // 第52行：验证权限
    int res = AccessTokenKit::VerifyAccessToken(tokenCaller, permission);
    return res == PERMISSION_GRANTED;
}

// 第54行：系统应用检查
static bool CheckSystemAppAndPermission()
{
    uint32_t tokenCaller = IPCSkeleton::GetCallingTokenID();
    // 第60行：检查是否为系统应用
    if (!TokenIdKit::IsSystemAppByFullTokenID(tokenCaller)) {
        HILOG_ERROR("CheckSystemAppAndPermission: caller is not system app");
        return false;
    }
    // 第64行：检查权限
    return CheckPermission(FILE_ACCESS_PERMISSION);
}
```

**权限检查覆盖分析**：

| 接口 | 检查点 | 权限要求 | 状态 |
|------|--------|----------|------|
| `OpenFile` | `file_access_ext_stub_impl.cpp:64` | FILE_ACCESS_MANAGER | ✅ 已检查 |
| `CreateFile` | `file_access_ext_stub_impl.cpp:91` | FILE_ACCESS_MANAGER | ✅ 已检查 |
| `Delete` | `file_access_ext_stub_impl.cpp:115` | FILE_ACCESS_MANAGER | ✅ 已检查 |
| `Move` | `file_access_ext_stub_impl.cpp:139` | FILE_ACCESS_MANAGER | ✅ 已检查 |
| `Copy` | `file_access_ext_stub_impl.cpp:165` | FILE_ACCESS_MANAGER | ✅ 已检查 |
| `AddRecentFile` | `recent_n_exporter.cpp:61` | FILE_ACCESS_MANAGER + 系统应用 | ✅ 已检查 |
| `RemoveRecentFile` | `recent_n_exporter.cpp:61` | FILE_ACCESS_MANAGER + 系统应用 | ✅ 已检查 |

**潜在绕过场景**：

1. **Token 伪造**：
   ```cpp
   // 如果攻击者能获取到具有权限的 token
   uint32_t privilegedToken = getPrivilegedToken();
   // 使用此 token 进行 IPC 调用
   ```

2. **权限检查顺序**：
   ```cpp
   // 危险模式：先执行操作后检查权限
   int fd = open(path, O_RDWR);  // ❌ 先执行
   if (!CheckPermission()) {      // ❌ 后检查
       close(fd);
   }
   ```

3. **TOCTOU 竞态**：
   ```cpp
   // 检查和应用之间的时间窗口
   if (CheckPermission(permission)) {
       // 窗口期：权限可能被撤销
       PerformPrivilegedOperation();
   }
   ```

**修复建议**：
```cpp
// 1. 统一的权限检查入口
class SecurityManager {
public:
    static int32_t EnforcePermission(const std::string& permission) {
        uint32_t token = IPCSkeleton::GetCallingTokenID();
        int result = AccessTokenKit::VerifyAccessToken(token, permission);
        if (result != PERMISSION_GRANTED) {
            HILOG_ERROR("Permission denied: %{public}s", permission.c_str());
            return E_PERMISSION_DENIED;
        }
        return ERR_OK;
    }
    
    // 2. 双重检查模式（敏感操作）
    static int32_t EnforcePermissionStrict(const std::string& permission) {
        // 检查1：调用时
        int32_t ret = EnforcePermission(permission);
        if (ret != ERR_OK) return ret;
        
        // 执行操作
        // ...
        
        // 检查2：操作后验证（可选）
        ret = EnforcePermission(permission);
        if (ret != ERR_OK) {
            // 回滚操作
        }
        return ret;
    }
};

// 3. 所有敏感操作必须检查
#define ENFORCE_PERMISSION(perm) \
    do { \
        int32_t __ret = SecurityManager::EnforcePermission(perm); \
        if (__ret != ERR_OK) return __ret; \
    } while(0)
```

---

### 低风险

#### R6: 信息泄露（低危）

**位置**：多处使用 `%{public}s` 输出敏感信息

**证据**：
```cpp
// 日志中可能泄露敏感路径
HILOG_INFO("Open file: %{public}s", uri.c_str());
HILOG_ERROR("Failed to access: %{public}s", filePath.c_str());
```

**风险分析**：

| 信息类型 | 泄露方式 | 风险等级 |
|----------|----------|----------|
| 文件路径 | 日志输出 | 低 |
| URI 内容 | 错误信息 | 低 |
| 内部错误码 | 返回值 | 低 |
| 内存地址 | 调试日志 | 极低 |

**敏感信息示例**：
```cpp
// 泄露用户目录结构
HILOG_INFO("User document path: %{public}s", "/storage/emulated/0/Documents/private/");

// 泄露文件操作
HILOG_INFO("Accessing secret file: %{public}s", "/storage/emulated/0/.hidden_config");
```

**修复建议**：
```cpp
// 1. 敏感信息使用 %s（脱敏）或省略
HILOG_INFO("Open file");  // 不输出路径
HILOG_ERROR("Failed to access file, errno=%{public}d", errno);  // 仅输出错误码

// 2. 分级日志策略
#ifdef DEBUG
    HILOG_DEBUG("Debug: path=%{public}s", path.c_str());  // 仅调试版
#else
    HILOG_INFO("Open file");  // 发布版
#endif

// 3. 路径脱敏处理
std::string SanitizePath(const std::string& path) {
    // 将 /storage/emulated/0/xxx 脱敏为 /storage/<uid>/xxx
    // 隐藏实际用户ID和敏感目录名
}
```

---

#### R7: TOCTOU 竞态条件（低危）

**位置**：`interfaces/kits/native/recent/recent_n_exporter.cpp:128` 附近

**证据**：
```cpp
// 典型 TOCTOU 模式（如果存在）
bool exists = CheckFileExists(path);  // T1: 检查
if (!exists) {
    // T1-T2 之间：文件可能被创建或修改
    CreateSymlink(targetPath, linkPath);  // T2: 使用
}

// 或
if (ValidatePath(targetPath)) {  // T1: 验证
    // 窗口期：targetPath 可能被修改
    symlink(targetPath.c_str(), linkPath.c_str());  // T2: 使用
}
```

**风险场景**：

| 场景 | 攻击方式 | 影响 |
|------|----------|------|
| 符号链接追逐 | 在验证和创建之间切换目标 | 访问未授权文件 |
| 路径替换 | 替换目标路径指向 | 数据泄露/篡改 |
| 竞争创建 | 同时创建同名文件 | 未定义行为 |

**安全模式示例**：
```cpp
// ❌ 不安全模式
if (access(path, F_OK) == 0) {  // 检查
    unlink(path);                // 使用
}

// ✅ 安全模式：使用原子操作
int fd = open(path, O_CREAT | O_EXCL | O_WRONLY, mode);
if (fd < 0) {
    if (errno == EEXIST) {
        // 文件已存在，处理冲突
    }
}

// ✅ 安全模式：使用文件描述符验证
int fd = open(dirPath, O_RDONLY | O_DIRECTORY);
if (fd < 0) return E_INVALID_PATH;

// 使用 *at 函数族，基于文件描述符操作
int fileFd = openat(fd, filename, O_CREAT | O_EXCL | O_WRONLY);
```

**修复建议**：
```cpp
// 1. 使用原子文件操作
int AtomicCreateFile(const std::string& path) {
    return open(path.c_str(), O_CREAT | O_EXCL | O_WRONLY, 0644);
}

// 2. 使用文件描述符验证（符号链接安全）
int SafeCreateSymlink(const std::string& target, const std::string& link) {
    // 1. 打开父目录
    std::string dir = GetParentDir(link);
    int dirFd = open(dir.c_str(), O_RDONLY | O_DIRECTORY);
    if (dirFd < 0) return E_INVALID_PATH;
    
    // 2. 验证目标路径（在创建前）
    char resolved[PATH_MAX];
    if (realpath(target.c_str(), resolved) == nullptr) {
        close(dirFd);
        return E_INVALID_PATH;
    }
    
    // 3. 创建符号链接
    int ret = symlinkat(target.c_str(), dirFd, GetFileName(link).c_str());
    
    // 4. 验证创建的链接
    char linkTarget[PATH_MAX];
    ssize_t len = readlinkat(dirFd, GetFileName(link).c_str(), linkTarget, sizeof(linkTarget));
    
    close(dirFd);
    return (ret == 0) ? ERR_OK : E_SYSCALL_ERROR;
}

// 3. 使用互斥锁保护关键区域
std::mutex fileOperationMutex;
int32_t SafeFileOperation(const std::string& path) {
    std::lock_guard<std::mutex> lock(fileOperationMutex);
    // 检查和操作原子化
    if (!ValidatePath(path)) return E_INVALID_PATH;
    return PerformOperation(path);
}
```

---

## 缓解措施

### 已有的安全机制

| 机制 | 实现位置 | 有效性 |
|------|----------|--------|
| AccessToken 校验 | `UfsAccessTokenHelper` | ✅ 有效 |
| SELinux 策略 | `selinux_adapter` | ✅ 有效 |
| CFI 保护 | `BUILD.gn:cfi=true` | ✅ 有效 |
| PAC 保护 | `branch_protector_ret` | ✅ 有效 |
| 整数溢出检测 | `ubsan=true` | ✅ 有效 |

**证据**：`services/BUILD.gn:176-183`

```gn
sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
}
branch_protector_ret = "pac_ret"
```

### 建议的额外措施

| 措施 | 优先级 | 实施难度 |
|------|--------|----------|
| 路径规范化 | 高 | 中 |
| URI 白名单 | 高 | 低 |
| API 限流 | 中 | 低 |
| 沙箱隔离 | 中 | 高 |
| 审计日志 | 低 | 低 |

---

## 权限清单

### 系统权限

| 权限名 | 授权方式 | 用途 |
|--------|----------|------|
| `ohos.permission.FILE_ACCESS_MANAGER` | system_grant | 文件访问管理 |
| `ohos.permission.FILE_ACCESS_AS_USER` | user_grant | 用户级文件访问 |

### 权限校验点

| 校验位置 | 校验方法 |
|----------|----------|
| N-API 入口 | `FileAccessHelperInit()` |
| SA 服务端 | `CheckCallingPermission()` |
| 扩展连接 | `ConnectFileExtAbility()` |

**证据**：`recent_n_exporter.h:48`, `file_access_service.h:227`

---

## 安全配置建议

### 1. SELinux 策略

```selinux
# 允许 FileAccessService 访问必要路径
allow filemanagement_service storage_file:dir { read write search };
allow filemanagement_service media_file:file { read write open };
```

### 2. 沙箱配置

```json
{
    "sandbox": {
        "enabled": true,
        "paths": [
            "/data/service/el2/"
        ]
    }
}
```

---

## 结论

user_file_service 整体安全状况良好，主要风险集中在输入校验方面。建议：

1. **高优先级**：实施路径遍历防护
2. **高优先级**：完善 URI 校验机制
3. **中优先级**：添加 API 限流
4. **低优先级**：加强日志脱敏

**评审人员**：[待填写]

**评审日期**：2026-02-06

**下次评审**：代码重大变更时
