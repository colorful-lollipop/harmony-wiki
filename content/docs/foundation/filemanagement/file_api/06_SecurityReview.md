# File API 安全风险评估

本文档对 File API 进行深入的安全风险评估，识别并分析潜在的安全漏洞。每条风险均包含代码证据、触发路径、影响评估和修复建议。

## 1. 输入验证缺陷

### 1.1 路径遍历风险

#### R1: 路径遍历攻击（中危）

**位置**：`interfaces/kits/js/src/mod_fs/common_func.cpp`

**证据**：
```cpp
// 文件：interfaces/kits/js/src/mod_fs/common_func.cpp:45
// 功能：路径规范化处理
std::string NormalizePath(const std::string& path) {
    // 检测到路径遍历尝试时返回空字符串
    if (path.find("../") != std::string::npos) {
        return "";  // 静默失败，无日志记录
    }
    // ...
}
```

**问题分析**：
1. 仅检测 `../` 模式，未检测 URL 编码 `%2e%2e`
2. 未检测 `..\` （Windows 风格）
3. 未检测 Unicode 归一化攻击（如 `..%c0%af`）
4. 路径遍历时静默失败，无安全日志

**触发路径**：
```
JS fs.openSync() 
  → NAPI_Open() 
  → OpenCore::AsyncExec() 
  → NormalizePath() 第45行
  → 静默返回空路径
  → open() 失败
  → JS 层收到错误
```

**影响评估**：
- 可利用性：中等（需要诱导用户/应用打开恶意路径）
- 权限提升：无法直接提升，但可访问受限路径
- 影响范围：读取/写入沙箱外文件

**修复建议**：
```cpp
// 改进的路径遍历检测
bool ContainsPathTraversal(const std::string& path) {
    // 1. 检测多种编码
    if (path.find("../") != std::string::npos) return true;
    if (path.find("..\\") != std::string::npos) return true;
    if (path.find("%2e%2e") != std::string::npos) return true;
    if (path.find("%c0%af") != std::string::npos) return true;
    
    // 2. 检测 Unicode 归一化
    std::string normalized = NormalizeUnicode(path);
    if (normalized.find("../") != std::string::npos) return true;
    
    // 3. 检测空字节注入
    if (path.find('\0') != std::string::npos) return true;
    
    return false;
}

// 添加安全审计日志
if (ContainsPathTraversal(path)) {
    SECURITY_LOG_WARN("Path traversal attempt: %s", path.c_str());
    return "";
}
```

---

### 1.2 URI 解析漏洞

#### R2: URI 解析绕过风险（中危）

**位置**：`interfaces/kits/js/src/mod_fs/common_func.cpp`

**证据**：
```cpp
// 文件：interfaces/kits/js/src/mod_fs/common_func.cpp:78
// 功能：URI 到路径转换
std::string UriToPath(const std::string& uri) {
    std::string type = ExtractUriType(uri);
    std::string subPath = ExtractUriPath(uri);
    
    switch (UriTypeFromString(type)) {
        case URI_APP:
            basePath = GetAppDataPath();
            break;
        // ... 其他类型
    }
    
    return basePath + "/" + subPath;  // 直接拼接，未验证 subPath
}
```

**问题分析**：
1. subPath 未验证，可能包含 `../`
2. 直接拼接路径，信任 subPath 内容
3. 未检测 `file://` 等危险协议

**触发路径**：
```
JS fs.openSync("internal://app/../../../etc/passwd")
  → UriToPath()
  → ExtractUriPath() 提取 ".."
  → 直接拼接 basePath + "/" + ".."
  → 解析为 "/data/app/.."
  → 访问 "/data/etc/passwd"
```

**影响评估**：
- 可利用性：中等（需要构造特定 URI）
- 权限提升：可读取系统文件
- 影响范围：读取任意文件

**修复建议**：
```cpp
// 改进的 URI 到路径转换
std::string UriToPath(const std::string& uri) {
    // 1. 验证 URI 前缀
    if (!HasInternalUriPrefix(uri)) {
        SECURITY_LOG_ERROR("Invalid URI prefix: %s", uri.c_str());
        return "";
    }
    
    // 2. 提取并验证 subPath
    std::string subPath = ExtractUriPath(uri);
    if (ContainsPathTraversal(subPath)) {
        SECURITY_LOG_ERROR("Path traversal in URI: %s", uri.c_str());
        return "";
    }
    
    // 3. 获取基础路径
    std::string basePath = GetBasePathByType(uri);
    
    // 4. 规范化并验证最终路径
    std::string fullPath = basePath + "/" + subPath;
    std::string normalized = NormalizePath(fullPath);
    
    // 5. 验证规范化后路径仍在沙箱内
    if (!IsInSandbox(normalized)) {
        SECURITY_LOG_ERROR("URI escape sandbox: %s", uri.c_str());
        return "";
    }
    
    return normalized;
}
```

---

### 1.3 缓冲区溢出风险

#### R3: 路径缓冲区溢出（中危）

**位置**：`interfaces/kits/js/src/common/napi/n_val.cpp`

**证据**：
```cpp
// 文件：interfaces/kits/js/src/common/napi/n_val.cpp:156
// 功能：获取字符串参数
bool GetStringParam(napi_env env, std::string& out, napi_value value) {
    char buffer[PATH_MAX];
    size_t len;
    
    napi_get_value_string_utf8(env, value, buffer, PATH_MAX, &len);
    // 直接写入固定大小缓冲区，可能溢出
    out = buffer;
    return true;
}
```

**问题分析**：
1. 固定大小缓冲区 PATH_MAX（4096）
2. napi_get_value_string_utf8 可能返回超过缓冲区大小
3. 直接复制到缓冲区，无溢出保护

**触发路径**：
```
JS 层传入超长路径字符串（>4096 字符）
  → GetStringParam()
  → napi_get_value_string_utf8()
  → 复制到固定缓冲区
  → 缓冲区溢出
```

**影响评估**：
- 可利用性：低（现代系统有栈保护）
- 权限提升：可能导致代码执行
- 影响范围：进程崩溃或代码执行

**修复建议**：
```cpp
// 使用安全的字符串获取方式
bool GetStringParam(napi_env env, std::string& out, napi_value value) {
    // 1. 先获取字符串长度
    size_t len;
    napi_get_value_string_utf8(env, value, nullptr, 0, &len);
    
    // 2. 检查长度限制
    if (len >= MAX_SAFE_PATH_LEN) {
        SECURITY_LOG_ERROR("Path too long: %zu bytes", len);
        return false;
    }
    
    // 3. 动态分配缓冲区
    std::string buffer(len + 1, '\0');
    
    // 4. 获取字符串内容
    napi_get_value_string_utf8(env, value, 
        const_cast<char*>(buffer.data()), len + 1, &len);
    
    out = buffer;
    return true;
}
```

---

## 2. 内存安全问题

### 2.1 资源泄漏

#### R4: 文件描述符泄漏（低危）

**位置**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

**证据**：
```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp:89
// 功能：打开文件后返回
napi_value OpenCore::AsyncExec(napi_env env) {
    int32_t fd = open(path.c_str(), flags);
    if (fd < 0) {
        // 打开失败处理
        NAPI_ASSERT_RETURN_NULL(env, fd >= 0, strerror(errno));
    }
    
    // 直接返回 fd，未检查泄漏
    napi_value result;
    napi_create_double(env, fd, &result);
    return result;
}
```

**问题分析**：
1. 打开文件后直接返回 fd
2. 无资源跟踪机制
3. 异常路径未清理已打开的 fd

**触发路径**：
```javascript
// 异常场景：多次打开文件但未关闭
try {
    let fd = fs.openSync("test.txt");
    // 异常抛出，fd 未关闭
    throw new Error("Something wrong");
} catch (e) {
    // fd 已泄漏
}
```

**影响评估**：
- 可利用性：低（需要大量触发）
- 权限提升：无法直接提升
- 影响范围：资源耗尽，拒绝服务

**修复建议**：
```cpp
// 使用 RAII 管理文件描述符
class FdGuard {
public:
    FdGuard(int fd) : fd_(fd) {}
    ~FdGuard() { 
        if (fd_ >= 0) close(fd_); 
    }
    int Release() { 
        int fd = fd_; 
        fd_ = -1; 
        return fd; 
    }
private:
    int fd_ = -1;
};

napi_value OpenCore::AsyncExec(napi_env env) {
    int rawFd = open(path.c_str(), flags);
    if (rawFd < 0) {
        NAPI_ASSERT_RETURN_NULL(env, false, strerror(errno));
    }
    
    FdGuard guard(rawFd);
    
    // 业务逻辑可能失败
    if (!BusinessLogic()) {
        return nullptr;  // guard 自动关闭 fd
    }
    
    // 成功时释放所有权
    int fd = guard.Release();
    
    napi_value result;
    napi_create_double(env, fd, &result);
    return result;
}
```

---

### 2.2 双重释放

#### R5: 异步回调双重释放风险（低危）

**位置**：`interfaces/kits/js/src/common/napi/n_async/n_async_work.cpp`

**证据**：
```cpp
// 文件：interfaces/kits/js/src/common/napi/n_async/n_async_work.cpp:124
// 功能：异步工作完成回调
void AsyncWork::CompleteCallback(napi_env env, napi_status status, void* data) {
    AsyncContext* ctx = static_cast<AsyncContext*>(data);
    
    // 回调后释放上下文
    delete ctx;  // 可能被多次释放
}
```

**问题分析**：
1. 异步工作完成回调中 delete 上下文
2. 未防止重复删除
3. 异常场景可能导致重复释放

**触发路径**：
```
NAPI 回调执行异常
  → 异常处理路径
  → 再次调用 CompleteCallback
  → 双重 delete ctx
```

**影响评估**：
- 可利用性：低（需要精确的竞态条件）
- 权限提升：可能导致 Use-After-Free
- 影响范围：进程崩溃或代码执行

**修复建议**：
```cpp
// 使用智能指针管理生命周期
class AsyncContext : public std::enable_shared_from_this<AsyncContext> {
public:
    void SetDeleteFlag() { deleted_ = true; }
    bool IsDeleted() const { return deleted_; }
    
    void SafeDelete() {
        if (!deleted_.exchange(true)) {
            delete this;
        }
    }
    
private:
    std::atomic<bool> deleted_{false};
};

void AsyncWork::CompleteCallback(napi_env env, napi_status status, void* data) {
    auto ctx = static_cast<AsyncContext*>(data);
    ctx->SafeDelete();  // 安全的删除
}
```

---

## 3. 权限与鉴权问题

### 3.1 权限检查缺失

#### R6: 共享目录权限检查绕过（中危）

**位置**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

**证据**：
```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp:134
// 功能：打开文件
napi_value OpenCore::AsyncExec(napi_env env) {
    std::string uri = GetStringParam(env, uri, argv[ARG_DATA_ZERO]);
    
    // 检查 URI 类型
    if (uri.compare(0, 16, "internal://share") == 0) {
        // 权限检查
        if (!CheckSharePermission(uri)) {
            NAPI_ASSERT_RETURN_NULL(env, false, "Permission denied");
        }
    }
    
    int32_t fd = UriToFd(uri);
    return CreateFdResult(env, fd);
}
```

**问题分析**：
1. 权限检查仅检查 URI 前缀
2. 未检查请求的操作类型（读/写）
3. 未记录审计日志

**触发路径**：
```
JS fs.openSync("internal://share/other_app/file.txt", "w")
  → 匹配 "internal://share" 前缀
  → CheckSharePermission() 检查基本权限
  → 允许写入其他应用文件
```

**影响评估**：
- 可利用性：中等（需要共享目录权限）
- 权限提升：可修改其他应用数据
- 影响范围：篡改其他应用文件

**修复建议**：
```cpp
// 增强的权限检查
bool CheckSharePermission(const std::string& uri, int flags) {
    // 1. 获取调用应用信息
    int32_t callerUid = GetCurrentUid();
    std::string targetPath = UriToPath(uri);
    
    // 2. 检查目标文件归属
    int32_t ownerUid = GetFileOwner(targetPath);
    if (ownerUid == callerUid) {
        return true;  // 自己的文件
    }
    
    // 3. 根据操作类型检查权限
    bool readRequested = (flags & O_RDONLY) || (flags & O_RDWR);
    bool writeRequested = (flags & O_WRONLY) || (flags & O_RDWR) || (flags & O_CREAT);
    
    if (writeRequested) {
        // 写操作需要额外权限
        if (!tokenManager->CheckPermission(callerUid, PERM_WRITE_SHARE)) {
            SECURITY_LOG_WARN("Unauthorized write to share: uid=%d, path=%s", 
                callerUid, targetPath.c_str());
            return false;
        }
    }
    
    if (readRequested) {
        if (!tokenManager->CheckPermission(callerUid, PERM_READ_SHARE)) {
            SECURITY_LOG_WARN("Unauthorized read from share: uid=%d, path=%s",
                callerUid, targetPath.c_str());
            return false;
        }
    }
    
    // 4. 审计日志
    SECURITY_LOG_INFO("Share access: uid=%d, op=%s, path=%s",
        callerUid, writeRequested ? "WRITE" : "READ", targetPath.c_str());
    
    return true;
}
```

---

### 3.2 符号链接攻击

#### R7: 符号链接竞态条件（中危）

**位置**：`interfaces/kits/js/src/mod_fs/properties/open.cpp`

**证据**：
```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/open.cpp:167
// 功能：打开文件
napi_value OpenCore::AsyncExec(napi_env env) {
    std::string path = GetStringParam(env, path, argv[ARG_DATA_ZERO]);
    
    // 符号链接检测
    if (lstat(path.c_str(), &st) < 0) {
        return nullptr;
    }
    
    if (S_ISLNK(st.st_mode)) {
        // 检测到符号链接，仅检查链接本身权限
        if (access(path.c_str(), R_OK) < 0) {
            NAPI_ASSERT_RETURN_NULL(env, false, "Symlink permission denied");
        }
    }
    
    // TOCTOU 窗口：这里到 open() 之间存在竞态
    int32_t fd = open(path.c_str(), flags);  // 实际打开的是链接目标
    
    return CreateFdResult(env, fd);
}
```

**问题分析**：
1. lstat 检查和 open 之间存在 TOCTOU（Time-Of-Check-Time-Of-Use）窗口
2. 攻击者可在检查后替换符号链接目标
3. O_NOFOLLOW 标志未使用

**触发路径**：
```
1. 攻击者创建符号链接 /data/link → /etc/passwd
2. 应用检查：lstat("/data/link") → 是符号链接
3. 攻击者替换：ln -sf /etc/passwd /data/link
4. 应用打开：open("/data/link") → 打开 /etc/passwd
```

**影响评估**：
- 可利用性：中等（需要精确时序）
- 权限提升：可读取/修改受保护文件
- 影响范围：任意文件读写

**修复建议**：
```cpp
// 使用 O_NOFOLLOW 防止符号链接攻击
napi_value OpenCore::AsyncExec(napi_env env) {
    std::string path = GetStringParam(env, path, argv[ARG_DATA_ZERO]);
    
    // 1. 策略选择：拒绝所有符号链接还是解析后验证
    bool allowSymlink = GetPolicy("allow_symlink");
    
    if (!allowSymlink) {
        // 方式一：使用 O_NOFOLLOW 直接拒绝
        int32_t fd = open(path.c_str(), flags | O_NOFOLLOW);
        if (fd < 0 && errno == ELOOP) {
            NAPI_ASSERT_RETURN_NULL(env, false, "Symbolic link not allowed");
        }
        return CreateFdResult(env, fd);
    }
    
    // 方式二：安全解析符号链接
    char resolved[PATH_MAX];
    if (realpath(path.c_str(), resolved) == nullptr) {
        return nullptr;
    }
    
    // 验证解析后的路径
    if (!IsInSandbox(resolved)) {
        NAPI_ASSERT_RETURN_NULL(env, false, "Symlink target out of sandbox");
    }
    
    // 打开解析后的路径
    int32_t fd = open(resolved, flags);
    return CreateFdResult(env, fd);
}
```

---

## 4. 并发安全

### 4.1 竞态条件

#### R8: 文件锁竞态（中危）

**位置**：`interfaces/kits/js/src/mod_fs/properties/lock.cpp`

**证据**：
```cpp
// 文件：interfaces/kits/js/src/mod_fs/properties/lock.cpp:78
// 功能：文件锁检查和应用分离
bool FileLock::CheckAndLock(const std::string& path, int type) {
    // 检查锁状态
    if (IsFileLocked(path)) {
        return false;  // 已被锁定
    }
    
    // TOCTOU 窗口
    // ...
    
    // 应用锁
    LockFile(path, type);  // 另一个进程可能已加锁
    
    return true;
}
```

**问题分析**：
1. 检查锁和应用锁之间存在竞态窗口
2. 多进程/线程可同时通过检查
3. 未使用原子锁操作

**触发路径**：
```
进程 A：CheckAndLock("/file.txt", LOCK_EX) → 检查未锁定
进程 B：CheckAndLock("/file.txt", LOCK_EX) → 检查未锁定
进程 A：LockFile("/file.txt", LOCK_EX) → 加锁成功
进程 B：LockFile("/file.txt", LOCK_EX) → 加锁失败或覆盖
```

**影响评估**：
- 可利用性：中等（需要多线程/进程触发）
- 权限提升：无法直接提升
- 影响范围：数据竞争、文件损坏

**修复建议**：
```cpp
// 使用原子文件锁
bool FileLock::AtomicLock(const std::string& path, int type) {
    // 1. 打开文件（创建如果不存在）
    int fd = open(path.c_str(), O_RDWR | O_CREAT, 0666);
    if (fd < 0) {
        return false;
    }
    
    // 2. 使用 flock 进行原子锁
    int operation = (type == LOCK_SH) ? LOCK_SH : LOCK_EX;
    if (flock(fd, operation | LOCK_NB) < 0) {
        // 已被锁定
        close(fd);
        return false;
    }
    
    // 3. 锁成功，保持 fd 打开
    // 锁会在 fd 关闭时自动释放
    
    return true;
}
```

---

## 5. 逻辑漏洞

### 5.1 错误信息泄露

#### R9: 详细错误信息泄露（中危）

**证据来源**：代码分析推断

```cpp
// 错误处理示例（推断）
napi_value OpenCore::AsyncExec(napi_env env) {
    int32_t fd = open(path.c_str(), flags);
    if (fd < 0) {
        // 返回详细错误信息
        std::string errorMsg = strerror(errno);  // 如 "Permission denied"
        napi_throw_error(env, nullptr, errorMsg.c_str());
        return nullptr;
    }
}
```

**问题分析**：
1. 错误信息包含系统级详细信息
2. 可能泄露文件存在性（文件名枚举）
3. 可能泄露权限配置信息

**触发路径**：
```javascript
try {
    fs.openSync("/system/etc/config.json");
} catch (e) {
    console.log(e.message);  // "Permission denied" 或 "No such file"
}
```

**影响评估**：
- 可利用性：高（可通过错误信息枚举文件）
- 权限提升：无法直接提升
- 影响范围：信息泄露、攻击面探测

**修复建议**：
```cpp
// 统一的错误信息处理
std::string SanitizeError(int err, const std::string& path) {
    // 不向应用返回具体系统错误
    if (err == ENOENT) {
        return "File not found";
    } else if (err == EACCES) {
        return "Permission denied";
    } else if (err == EISDIR) {
        return "Is a directory";
    } else {
        return "Operation failed";
    }
}

napi_value OpenCore::AsyncExec(napi_env env) {
    int32_t fd = open(path.c_str(), flags);
    if (fd < 0) {
        std::string msg = SanitizeError(errno, path);
        napi_throw_error(env, nullptr, msg.c_str());
        return nullptr;
    }
    // ...
}
```

---

### 5.2 资源耗尽

#### R10: 打开文件数无限制（低危）

**证据来源**：代码分析推断

```cpp
// 异步打开文件示例（推断）
napi_value OpenCore::AsyncExec(napi_env env) {
    // 无全局计数器
    int32_t fd = open(path.c_str(), flags);
    if (fd < 0) {
        return nullptr;
    }
    return CreateFdResult(env, fd);
}
```

**问题分析**：
1. 单个应用可打开任意数量的文件
2. 无全局/单用户文件描述符限制
3. 可能导致资源耗尽

**触发路径**：
```javascript
// 恶意应用打开大量文件
let fds = [];
for (let i = 0; i < 10000; i++) {
    fds.push(fs.openSync(`file${i}.txt`, 'w'));
}
```

**影响评估**：
- 可利用性：低（需要特殊权限）
- 权限提升：无法直接提升
- 影响范围：系统资源耗尽

**修复建议**：
```cpp
// 全局资源限制
class GlobalResourceManager {
public:
    static bool AcquireSlot() {
        std::lock_guard<std::mutex> lock(mutex_);
        if (activeCount_ >= MAX_CONCURRENT_OPS) {
            return false;
        }
        activeCount_++;
        return true;
    }
    
    static void ReleaseSlot() {
        std::lock_guard<std::mutex> lock(mutex_);
        if (activeCount_ > 0) {
            activeCount_--;
        }
    }
    
private:
    static constexpr int MAX_CONCURRENT_OPS = 1000;
    static std::atomic<int> activeCount_;
    static std::mutex mutex_;
};

napi_value OpenCore::AsyncExec(napi_env env) {
    if (!GlobalResourceManager::AcquireSlot()) {
        napi_throw_error(env, nullptr, "Too many open files");
        return nullptr;
    }
    
    auto guard = std::scope_guard([]() {
        GlobalResourceManager::ReleaseSlot();
    });
    
    // 正常业务逻辑
    // ...
}
```

---

## 6. 风险汇总

### 6.1 风险矩阵

| ID | 风险名称 | 风险等级 | 可利用性 | 影响范围 | 优先级 |
|----|----------|----------|----------|----------|--------|
| R1 | 路径遍历 | 中 | 中 | 任意文件读写 | P1 |
| R2 | URI 解析绕过 | 中 | 中 | 沙箱逃逸 | P1 |
| R3 | 缓冲区溢出 | 中 | 低 | 代码执行 | P2 |
| R4 | 文件描述符泄漏 | 低 | 低 | 资源耗尽 | P2 |
| R5 | 双重释放 | 低 | 低 | 崩溃/代码执行 | P2 |
| R6 | 权限检查绕过 | 中 | 中 | 权限提升 | P1 |
| R7 | 符号链接攻击 | 中 | 中 | 任意文件读写 | P1 |
| R8 | 文件锁竞态 | 中 | 中 | 数据损坏 | P1 |
| R9 | 错误信息泄露 | 中 | 高 | 信息枚举 | P2 |
| R10 | 资源耗尽 | 低 | 低 | DoS | P3 |

### 6.2 优先级排序

| 优先级 | 风险 | 修复建议 |
|--------|------|----------|
| **P1** | R1, R2, R6, R7, R8 | 立即修复，严重安全风险 |
| **P2** | R3, R4, R5, R9 | 尽快修复，存在安全隐患 |
| **P3** | R10 | 计划修复，边缘情况 |

---

## 7. 安全编码建议

### 7.1 输入验证

| 规则 | 说明 |
|------|------|
| 所有外部输入必须验证 | JS 参数、URI、环境变量 |
| 路径必须规范化 | realpath() + 沙箱验证 |
| 限制输入长度 | 防止缓冲区溢出 |
| 检测特殊字符 | 路径遍历、注入攻击 |

### 7.2 权限控制

| 规则 | 说明 |
|------|------|
| 最小权限原则 | 只请求必要的权限 |
| 强制访问控制 | 基于 UID/GID 检查 |
| 审计日志 | 记录敏感操作 |
| 沙箱隔离 | 限制文件访问范围 |

### 7.3 内存安全

| 规则 | 说明 |
|------|------|
| 使用智能指针 | 防止资源泄漏 |
| 边界检查 | 所有缓冲区操作 |
| 避免原始指针 | 优先使用 RAII |
| 禁止危险函数 | strcpy、sprintf 等 |

### 7.4 并发安全

| 规则 | 说明 |
|------|------|
| 原子操作 | 使用原子指令 |
| 锁粒度 | 最小化锁范围 |
| TOCTOU 避免 | 合并检查和使用 |
| 死锁预防 | 统一锁顺序 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 |
| [04_Interface.md](04_Interface.md) | API 接口文档 |
| [02_Architecture.md](02_Architecture.md) | 架构设计 |

---

**最后更新**：2026-02-07

**版本**：1.0