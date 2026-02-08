# 安全风险评审

> 基于架构分析的安全威胁模型与风险清单

## 评审范围

| 组件 | 范围 | 说明 |
|------|------|------|
| @OHOS.distributedfile.fileio | ✅ | 文件 I/O API |
| @system.file | ✅ | 沙箱文件 API |
| LibN | ⚠️ | 抽象库（需代码确认） |
| IPC/SA | ❌ | 分布式部分（其他仓库） |

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                         不可信边界                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         应用进程 (Untrusted App Process)              │   │
│  │         - JS 代码执行                                │   │
│  │         - 用户输入处理                               │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│                          ▼ Trust Boundary                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         N-API Layer (LibN 抽象)                      │   │
│  │         - 参数校验                                   │   │
│  │         - 类型转换                                   │   │
│  │         - 权限检查（URI 沙箱）                        │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         Native Core                                 │   │
│  │         - 文件系统调用                               │   │
│  │         - 权限 enforcement                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│                          ▼                                  │
│                    GLIBC / Kernel                           │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

| 输入 | 处理 | 输出 | 风险等级 |
|------|------|------|----------|
| JS 路径字符串 | N-API 解析 | 系统调用 | 高 |
| JS 文件描述符 | 参数转换 | fd 操作 | 高 |
| URI | 沙箱映射 | 内部路径 | 中 |

## 攻击面清单

| 攻击面 | 类型 | 入口 API | 风险等级 |
|--------|------|----------|----------|
| **路径遍历** | 文件系统 | `fileio.access(path)`, `fileio.read(dir)` | 高 |
| **权限提升** | 权限 | `fileio.chown(path, uid, gid)` | 高 |
| **符号链接攻击** | 文件系统 | `fileio.open(path)` | 中 |
| **资源耗尽** | 资源 | `fileio.createStream()`, `Dir.openDirSync()` | 中 |
| **信息泄露** | 信息 | `Stat.statSync(path)` | 低 |
| **竞态条件** | 竞态 | 多文件操作 | 中 |
| **编码注入** | 输入 | `read/write` with encoding | 低 |

## 风险详情

### 🔴 高风险

#### R1: 路径遍历（Path Traversal）

**证据**: 所有接受路径参数的 API
- `fileio.access(path)`
- `fileio.chown(path, uid, gid)`
- `fileio.chmod(path, mode)`
- `fileio.createStreamSync(path, mode)`

**触发方式**:
```javascript
// 恶意构造路径
fileio.accessSync("/data/../../../etc/passwd");
fileio.readFileSync("/data/./../../etc/shadow");
```

**影响**:
- 越权访问敏感文件（`/etc/passwd`, `/etc/shadow`）
- 绕过沙箱隔离

**修复建议**:
```cpp
// N-API 层实现路径规范化
std::string NormalizePath(const std::string& path) {
    // 1. 解析 .. 和 .
    // 2. 检查是否在允许的沙箱目录内
    // 3. 拒绝符号链接（可选）
    // 4. 使用 realpath() 获取规范路径
}

// 白名单目录检查
bool IsPathAllowed(const std::string& path) {
    std::string real = realpath(path.c_str(), nullptr);
    for (auto& allowed : kAllowedDirs) {
        if (real.rfind(allowed, 0) == 0) {  // prefix match
            return true;
        }
    }
    return false;
}
```

**参考**: CWE-22: Improper Limitation of a Pathname to a Restricted Directory

---

#### R2: 权限滥用（chown/chmod）

**证据**: `fileio.chownSync(path, uid, gid)`, `fileio.chmodSync(path, mode)`

**触发方式**:
```javascript
// 尝试修改系统文件权限
fileio.chownSync("/system/bin/ls", 0, 0);  // uid=0, gid=0
fileio.chmodSync("/data/app/ sensitive", 0o777);
```

**影响**:
- 提权攻击
- 破坏权限隔离
- 篡改系统文件

**修复建议**:
```cpp
// 权限检查
bool CanModifyOwnership(const std::string& path, uid_t targetUid) {
    // 1. 检查调用进程是否为 root 或文件所有者
    // 2. 检查目标路径是否在应用沙箱内
    // 3. 禁止修改系统文件权限
    struct stat st;
    if (stat(path.c_str(), &st) != 0) return false;
    
    // 禁止修改 system 文件
    if (S_ISREG(st.st_mode) && 
        (strncmp(path.c_str(), "/system", 7) == 0 ||
         strncmp(path.c_str(), "/vendor", 7) == 0)) {
        return false;
    }
    return true;
}
```

**参考**: CWE-732: Incorrect Permission Assignment for Critical Resource

---

### 🟠 中风险

#### R3: 符号链接攻击（Symlink Attack）

**证据**: `fileio.open(path)`, `fileio.createStreamSync(path)`

**触发方式**:
```bash
# 恶意应用创建符号链接
ln -sf /data/secure/data /data/malicious/link
```

```javascript
// 利用符号链接读取敏感文件
fileio.readFileSync("/data/malicious/link");
```

**修复建议**:
```cpp
// 打开文件时使用 O_NOFOLLOW
int fd = open(path.c_str(), O_RDWR | O_NOFOLLOW);
if (fd < 0 && errno == ELOOP) {
    // 拒绝符号链接
    napi_throw_error(env, "ELOOP", "Symbolic link not allowed");
    return nullptr;
}
```

**参考**: CWE-59: Improper Link Resolution Before File Access

---

#### R4: 资源耗尽（Resource Exhaustion）

**证据**: `fileio.createStream()`, `fileio.openDirSync()`

**触发方式**:
```javascript
// 打开大量文件描述符
for (let i = 0; i < 10000; i++) {
    fileio.createStreamSync("/data/file" + i, "r");
}
```

**影响**:
- 拒绝服务（DoS）
- 内存耗尽

**修复建议**:
```cpp
// 资源限制检查
bool CheckResourceLimit(napi_env env, ResourceType type) {
    // 1. 检查当前进程打开文件数
    // 2. 检查内存使用
    // 3. 实施配额限制
    
    struct rlimit rl;
    if (getrlimit(RLIMIT_NOFILE, &rl) == 0) {
        if (GetOpenFdCount() >= rl.rlim_cur - 10) {
            napi_throw_error(env, "EMFILE", "Too many open files");
            return false;
        }
    }
    return true;
}
```

**参考**: CWE-400: Uncontrolled Resource Consumption

---

#### R5: Time-of-Check to Time-of-Use (TOCTOU)

**证据**: 多步骤文件操作

**触发方式**:
```javascript
// 检查文件存在 -> 使用文件的窗口期
if (fileio.accessSync("/data/target")) {
    // 窗口期：另一进程/线程可修改文件
    fileio.openSync("/data/target", "r");
}
```

**修复建议**:
```cpp
// 使用原子操作或锁定
int fd = open(path.c_str(), O_RDWR);
if (fd < 0) {
    if (errno == ENOENT) {
        // 文件不存在
    }
    return fd;
}
// fd 已打开，后续操作在 fd 上进行
```

**参考**: CWE-367: Time-of-check Time-of-use (TOCTOU) Race Condition

---

### 🟡 低风险

#### R6: 信息泄露 via Stat

**证据**: `Stat.statSync(path)`

**触发方式**:
```javascript
// 探测系统文件存在性
try {
    fileio.statSync("/data/secret");
    console.log("File exists");
} catch (e) {
    console.log("File not found");
}
```

**影响**:
- 文件存在性探测
- 敏感路径枚举

**修复建议**:
```cpp
// 对敏感路径返回模糊错误
if (IsSensitivePath(path)) {
    napi_throw_error(env, "ENOENT", "File not found");
    return;
}
```

**参考**: C-WE-200: Exposure of Sensitive Information to an Unauthorized Actor

---

## 安全编码建议

### 1. 输入验证

```cpp
// ✅ 正确：完整的路径验证
napi_value ValidateAndOpenFile(napi_env env, napi_value pathValue) {
    std::string path;
    napi_get_value_string_utf8(env, pathValue, ...);
    
    // 检查空路径
    if (path.empty()) {
        napi_throw_type_error(env, "EINVAL", "Path cannot be empty");
        return nullptr;
    }
    
    // 检查长度
    if (path.length() > MAX_PATH_LEN) {
        napi_throw_range_error(env, "ENAMETOOLONG", "Path too long");
        return nullptr;
    }
    
    // 检查特殊字符
    if (path.find("..") != std::string::npos) {
        napi_throw_error(env, "EINVAL", "Path traversal not allowed");
        return nullptr;
    }
    
    // 检查白名单
    if (!IsPathAllowed(path)) {
        napi_throw_error(env, "EACCES", "Path not in allowed directory");
        return nullptr;
    }
    
    return OpenFile(path);
}

// ❌ 错误：缺少验证
napi_value BadOpenFile(napi_env env, napi_value pathValue) {
    std::string path;
    napi_get_value_string_utf8(env, pathValue, ...);
    return open(path.c_str(), O_RDWR);  // 直接打开，无验证
}
```

### 2. 编码安全

```cpp
// ✅ 正确：显式指定 UTF-8 编码
napi_value ReadTextFile(napi_env env, napi_value pathValue) {
    std::string path;
    napi_get_value_value_string_utf8(env, pathValue, ...);
    
    int fd = open(path.c_str(), O_RDONLY);
    if (fd < 0) { /* error */ }
    
    // 读取并验证 UTF-8
    std::string content = ReadAll(fd);
    if (!IsValidUTF8(content)) {
        close(fd);
        napi_throw_error(env, "EILSEQ", "Invalid UTF-8 encoding");
        return nullptr;
    }
    
    close(fd);
    return napi_create_string_utf8(env, content.c_str(), ...);
}
```

### 3. 资源管理

```cpp
// ✅ 正确：RAII 资源管理
class FileHandle {
public:
    FileHandle(const std::string& path) : fd_(open(path.c_str(), O_RDONLY)) {}
    ~FileHandle() { if (fd_ >= 0) close(fd_); }
    
    bool IsValid() const { return fd_ >= 0; }
    int GetFd() const { return fd_; }
    
private:
    int fd_;
    DISALLOW_COPY_AND_ASSIGN(FileHandle);
};

napi_value SafeRead(napi_env env, napi_value pathValue) {
    std::string path;
    napi_get_value_string_utf8(env, pathValue, ...);
    
    FileHandle file(path);
    if (!file.IsValid()) {
        napi_throw_errno_error(env, errno, "Cannot open file");
        return nullptr;
    }
    
    // 使用 file.GetFd() 进行操作
    // RAII 保证异常时文件句柄正确关闭
}
```

---

## 已实现的安全机制

| 机制 | 实现位置 | 说明 |
|------|----------|------|
| 沙箱 URI | N-API 层 | `internal://cache/`, `internal://app/`, `internal://share/` |
| UTF-8/16 编码限制 | N-API 层 | 仅支持 UTF-8/16 |
| 外部存储禁止 | N-API 层 | URI 不能包含外部存储目录 |

## 待确认项

| 项 | 状态 | 说明 |
|----|------|------|
| 路径遍历防护 | ⚠️ 需确认 | README 未明确 |
| 符号链接处理 | ⚠️ 需确认 | README 未明确 |
| 权限检查 | ⚠️ 需确认 | README 未明确 |
| TOCTOU 防护 | ⚠️ 需确认 | README 未明确 |

## 参考

- [CWE Top 25](https://cwe.mitre.org/top25/)
- [OWASP File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [OpenHarmony 安全指南](https://docs.openharmony.cn)
