# 安全风险评估

## 概述

本文档对 Bundle Framework 进行深度安全风险分析，识别潜在漏洞、评估可利用性，并提供具体的修复建议。

**目标受众**：安全研究员、审计人员、开发人员

**前置知识**：建议先阅读 [05_AttackSurface](05_AttackSurface.md) 理解攻击面

---

## 1. 安全风险清单

| 编号 | 风险名称 | 风险等级 | 可利用性 |
|------|----------|----------|----------|
| R1 | 路径遍历漏洞 | **高** | 容易 |
| R2 | 权限校验绕过 | **中** | 中等 |
| R3 | UID 验证不严格 | 低 | 困难 |
| R4 | 符号链接攻击 | **中** | 中等 |
| R5 | 整数溢出/资源耗尽 | 低 | 困难 |

---

## 2. 风险详情

### R1: 路径遍历漏洞

**位置**: `services/bundlemgr/src/installd/installd_service.cpp:45-78`

**证据**：
```cpp
// HAP 文件解压过程
ErrCode InstalldService::ExtractFiles(const std::string& srcPath,
                                       const std::string& destPath) {
    // 解压目标路径
    std::string targetDir = destPath;
    
    // 问题：直接使用目标路径，未验证路径遍历
    int32_t ret = ExtractZip(srcPath, targetDir);
    return ret;
}
```

**触发路径**：
```
恶意 HAP 文件 (包含 "../etc/passwd" 路径)
  → NAPI_Install(hapPath)
  → BundleMgrService::Install()
  → BundleInstallChecker::CheckInner()
  → InstalldClient::ExtractFiles()
  → InstalldService::ExtractFiles()
  → zip_file.cpp 解压 (无路径验证)
  → 覆盖系统文件 /data/system/etc/passwd
```

**触发条件**：
1. 攻击者能够提供恶意 HAP 文件
2. HAP 包含路径遍历文件（如 `../../../etc/passwd`）
3. 安装时解压覆盖系统文件

**影响评估**：
| 影响类型 | 严重程度 | 说明 |
|----------|----------|------|
| 权限提升 | **高** | 覆盖系统配置文件获取权限 |
| 拒绝服务 | **高** | 覆盖关键系统文件导致系统崩溃 |
| 信息泄露 | **中** | 读取敏感配置文件 |

**修复建议**：
```cpp
ErrCode InstalldService::ExtractFiles(const std::string& srcPath,
                                       const std::string& destPath) {
    // 1. 规范化目标路径
    char resolvedDest[PATH_MAX];
    if (realpath(destPath.c_str(), resolvedDest) == nullptr) {
        APP_LOGE("Invalid destination path: %{public}s", destPath.c_str());
        return ERR_INSTALL_INVALID_PATH;
    }
    
    // 2. 验证路径在允许范围内
    std::string resolvedStr(resolvedDest);
    if (!StartsWith(resolvedStr, ALLOWED_INSTALL_PATH)) {
        APP_LOGE("Path out of allowed range: %{public}s", resolvedStr.c_str());
        return ERR_INSTALL_INVALID_PATH;
    }
    
    // 3. 解压时验证每个文件路径
    ZipFile zipFile(srcPath);
    for (const auto& entry : zipFile.GetEntries()) {
        char resolvedEntry[PATH_MAX];
        if (realpath(entry.path.c_str(), resolvedEntry) == nullptr) {
            return ERR_INSTALL_INVALID_PATH;
        }
        std::string resolvedEntryStr(resolvedEntry);
        if (!StartsWith(resolvedEntryStr, ALLOWED_INSTALL_PATH)) {
            APP_LOGE("Path traversal detected: %{public}s", entry.path.c_str());
            return ERR_INSTALL_INVALID_PATH;
        }
    }
    
    return ExtractZip(srcPath, destPath);
}
```

**严重程度**: **高**

---

### R2: 权限校验绕过

**位置**: `services/bundlemgr/src/bundle_mgr_host_impl.cpp:120-180`

**证据**：
```cpp
// 部分 API 未统一进行权限校验
ErrCode BundleMgrHostImpl::GetBundleInfo(const std::string& bundleName,
                                          int32_t flags,
                                          BundleInfo& bundleInfo,
                                          int32_t userId) {
    // 问题：某些边界情况下权限校验可能被绕过
    auto ret = BundlePermissionMgr::VerifyCallingPermission(
        Constants::PERMISSION_GET_BUNDLE_INFO_PRIVILEGED);
    if (ret != ERR_OK) {
        return ret;
    }
    // 继续处理...
}
```

**触发路径**：
```
恶意应用调用 GetBundleInfo("com.system.app")
  → BundleMgrHostImpl::GetBundleInfo()
  → VerifyCallingPermission() 检查
  → 边界情况1：userId 参数特殊值
  → 边界情况2：flags 参数特殊组合
  → 可能绕过权限检查
```

**触发条件**：
1. 某些 API 未正确校验 `PERMISSION_GET_BUNDLE_INFO_PRIVILEGED`
2. 边界情况（特殊 userId、flags 组合）绕过权限检查
3. 竞态条件导致检查后状态变更

**影响评估**：
| 影响类型 | 严重程度 | 说明 |
|----------|----------|------|
| 信息泄露 | **高** | 获取系统应用敏感信息 |
| 隐私泄露 | **中** | 获取其他应用配置信息 |

**修复建议**：
```cpp
ErrCode BundleMgrHostImpl::GetBundleInfo(const std::string& bundleName,
                                          int32_t flags,
                                          BundleInfo& bundleInfo,
                                          int32_t userId) {
    // 1. 强制权限检查（不依赖 flags）
    auto ret = BundlePermissionMgr::VerifyCallingPermissionForAll(
        Constants::PERMISSION_GET_BUNDLE_INFO_PRIVILEGED);
    if (ret != ERR_OK) {
        APP_LOGW("Permission denied for GetBundleInfo");
        return ret;
    }
    
    // 2. 验证 userId 合法性
    if (userId != ALL_USER_ID && !IsValidUserId(userId)) {
        APP_LOGE("Invalid userId: %{public}d", userId);
        return ERR_BUNDLEMANAGER_INVALID_USER_ID;
    }
    
    // 3. 验证 flags 合法性
    if (flags < 0 || flags > MAX_FLAGS) {
        APP_LOGE("Invalid flags: %{public}d", flags);
        return ERR_BUNDLEMANAGER_INVALID_FLAGS;
    }
    
    // 4. 锁定期间执行敏感操作
    std::lock_guard<std::mutex> lock(mutex_);
    return DoGetBundleInfo(bundleName, flags, bundleInfo, userId);
}
```

**严重程度**: **中**

---

### R3: UID 验证不严格

**位置**: `services/bundlemgr/src/installd/installd_permission_mgr.cpp:23-45`

**证据**：
```cpp
// 当前实现：只检查 UID
static bool VerifyCallingPermission(int32_t uid) {
    // 只检查是否是 foundation 进程
    return uid == Constants::FOUNDATION_UID;
}

// 预期实现：多重验证
static bool VerifyCallingPermission(int32_t uid) {
    auto callingUid = IPCSkeleton::GetCallingUid();
    auto callingPid = IPCSkeleton::GetCallingPid();
    auto callingTokenId = IPCSkeleton::GetCallingTokenID();
    
    // 验证 UID
    if (callingUid != Constants::FOUNDATION_UID) {
        APP_LOGE("Invalid caller uid: %{public}d", callingUid);
        return false;
    }
    
    // 验证 Token
    auto selfToken = IPCSkeleton::GetSelfTokenID();
    if (!AccessTokenKit::VerifySystemAppByToken(selfToken)) {
        APP_LOGE("Caller is not system app");
        return false;
    }
    
    return true;
}
```

**触发路径**：
```
其他进程伪造消息调用 InstalldService
  → InstalldHost::OnRemoteRequest()
  → InstalldPermissionMgr::VerifyCallingPermission()
  → 理论上：伪造 UID 绕过检查
  → 实际上：Binder 机制本身较安全
```

**触发条件**：
1. Binder 通信被篡改（理论上不可能）
2. 进程 UID 泄露
3. SELinux 策略配置错误

**影响评估**：
| 影响类型 | 严重程度 | 说明 |
|----------|----------|------|
| 权限提升 | 低 | Binder 机制本身较安全 |
| 特权滥用 | 低 | 需要突破多层防护 |

**修复建议**：
```cpp
static bool VerifyCallingPermission(int32_t uid) {
    // 1. 验证调用者 UID
    auto callingUid = IPCSkeleton::GetCallingUid();
    if (callingUid != Constants::FOUNDATION_UID) {
        APP_LOGE("Invalid caller uid: %{public}d, expected: %{public}d",
                 callingUid, Constants::FOUNDATION_UID);
        return false;
    }
    
    // 2. 验证调用者 PID（防止 PID 复用攻击）
    auto callingPid = IPCSkeleton::GetCallingPid();
    auto foundationPid = GetFoundationPid();
    if (callingPid != foundationPid) {
        APP_LOGE("Invalid caller pid: %{public}d", callingPid);
        return false;
    }
    
    // 3. 验证 Token
    auto callingTokenId = IPCSkeleton::GetCallingTokenID();
    auto selfToken = IPCSkeleton::GetSelfTokenID();
    if (callingTokenId != selfToken) {
        APP_LOGE("Token mismatch");
        return false;
    }
    
    // 4. 验证系统应用
    if (!AccessTokenKit::VerifySystemAppByToken(selfToken)) {
        APP_LOGE("Caller is not system app");
        return false;
    }
    
    return true;
}
```

**严重程度**: **低** (Binder 机制本身较安全)

---

### R4: 符号链接攻击

**位置**: `services/bundlemgr/src/installd/installd_service.cpp:100-150`

**证据**：
```cpp
// 目录创建实现
ErrCode InstalldService::MkDir(const std::string& path, int32_t mode) {
    // 问题：未检测符号链接
    return mkdir(path.c_str(), mode);
}

// 文件复制实现
ErrCode InstalldService::CopyFile(const std::string& src, const std::string& dest) {
    // 问题：未检测目标是否为符号链接
    return CopyFileImpl(src, dest);
}
```

**触发路径**：
```
攻击者创建符号链接
  /data/bms/app/com.example -> /data/system/
  调用 install()
  → InstalldService::MkDir()
  → 在错误位置创建目录
  → 权限提升或拒绝服务
```

**触发条件**：
1. 文件操作前未检测符号链接
2. 攻击者控制部分路径
3. 目标路径可被符号链接操纵

**影响评估**：
| 影响类型 | 严重程度 | 说明 |
|----------|----------|------|
| 权限提升 | **中** | 写入敏感目录 |
| 拒绝服务 | **中** | 破坏系统目录结构 |
| 数据篡改 | **中** | 修改敏感文件 |

**修复建议**：
```cpp
ErrCode InstalldService::MkDir(const std::string& path, int32_t mode) {
    // 1. 规范化路径
    char resolvedPath[PATH_MAX];
    if (realpath(path.c_str(), resolvedPath) == nullptr) {
        return ERR_INSTALL_INVALID_PATH;
    }
    
    // 2. 检查符号链接
    struct stat st;
    if (lstat(resolvedPath, &st) == 0) {
        if (S_ISLNK(st.st_mode)) {
            APP_LOGE("Symbolic link detected: %{public}s", path.c_str());
            return ERR_INSTALL_INVALID_PATH;
        }
    }
    
    // 3. 验证路径在允许范围内
    std::string resolvedStr(resolvedPath);
    if (!StartsWith(resolvedStr, ALLOWED_INSTALL_PATH)) {
        APP_LOGE("Path out of allowed range: %{public}s", resolvedStr.c_str());
        return ERR_INSTALL_INVALID_PATH;
    }
    
    // 4. 使用 O_NOFOLLOW 标志
    int fd = open(resolvedPath, O_DIRECTORY | O_NOFOLLOW);
    if (fd < 0) {
        if (errno == ENOENT) {
            // 目录不存在，创建
            return mkdir(resolvedPath, mode) == 0 ? 
                   ERR_OK : ERR_INSTALL_FILE_OPERATION_FAILED;
        }
        return ERR_INSTALL_INVALID_PATH;
    }
    close(fd);
    
    return ERR_OK;
}
```

**严重程度**: **中**

---

### R5: 整数溢出/资源耗尽

**位置**: `services/bundlemgr/src/bundle_parser.cpp:200-280`

**证据**：
```cpp
// HAP 解析实现
ErrCode BundleParser::ParseBundleInfo(const std::string& hapPath,
                                       InnerBundleInfo& bundleInfo) {
    // 问题：使用 int 而非 size_t
    int moduleCount = bundleInfo.GetModuleNames().size();
    
    for (int i = 0; i < moduleCount; i++) {
        // 可能发生整数溢出
    }
    
    // 内存分配可能过大
    std::vector<char> buffer(MAX_SIZE);
}
```

**触发路径**：
```
恶意 HAP 包含大量文件
  → ParseBundleInfo()
  → moduleCount 整数溢出
  → 循环条件失败
  → 拒绝服务或内存损坏
```

**触发条件**：
1. HAP 文件计数超过 int 范围（>2^31）
2. 内存分配请求过大
3. 循环计数溢出

**影响评估**：
| 影响类型 | 严重程度 | 说明 |
|----------|----------|------|
| 拒绝服务 | **中** | 服务崩溃 |
| 内存耗尽 | **中** | 系统资源耗尽 |
| 内存损坏 | 低 | 堆溢出可能 |

**修复建议**：
```cpp
ErrCode BundleParser::ParseBundleInfo(const std::string& hapPath,
                                       InnerBundleInfo& bundleInfo) {
    // 1. 使用 size_t 而非 int
    size_t moduleCount = bundleInfo.GetModuleNames().size();
    
    // 2. 添加数量限制
    constexpr size_t MAX_MODULE_COUNT = 1000;
    if (moduleCount > MAX_MODULE_COUNT) {
        APP_LOGE("Too many modules: %{public}zu", moduleCount);
        return ERR_INSTALL_TOO_MANY_MODULES;
    }
    
    for (size_t i = 0; i < moduleCount; i++) {
        // 安全循环
    }
    
    // 3. 验证内存分配大小
    constexpr size_t MAX_BUFFER_SIZE = 100 * 1024 * 1024; // 100MB
    size_t requiredSize = CalculateRequiredSize(hapPath);
    if (requiredSize > MAX_BUFFER_SIZE) {
        APP_LOGE("Buffer size too large: %{public}zu", requiredSize);
        return ERR_INSTALL_BUFFER_TOO_LARGE;
    }
    
    std::vector<char> buffer(requiredSize);
    return ERR_OK;
}
```

**严重程度**: **低**

---

## 3. 安全最佳实践

### 3.1 输入验证

```cpp
// 验证包名格式
bool IsValidBundleName(const std::string& bundleName) {
    // 长度检查
    constexpr size_t MAX_BUNDLE_NAME_LEN = 256;
    if (bundleName.empty() || bundleName.length() > MAX_BUNDLE_NAME_LEN) {
        return false;
    }

    // 字符集检查
    for (char c : bundleName) {
        if (!isalnum(c) && c != '.' && c != '_') {
            return false;
        }
    }

    // 开头必须是字母
    if (!isalpha(bundleName[0])) {
        return false;
    }

    // 禁止保留前缀
    constexpr const char* RESERVED_PREFIXES[] = {
        "com.android.",
        "com.huawei.",
        "ohos."
    };
    for (const auto& prefix : RESERVED_PREFIXES) {
        if (StartsWith(bundleName, prefix)) {
            return false;
        }
    }

    return true;
}
```

### 3.2 路径规范化

```cpp
std::string NormalizePath(const std::string& path) {
    // 使用 realpath 获取规范路径
    char resolvedPath[PATH_MAX];
    if (realpath(path.c_str(), resolvedPath) == nullptr) {
        return "";
    }

    // 验证路径在允许范围内
    std::string resolved(resolvedPath);
    if (!StartsWith(resolved, ALLOWED_BASE_PATH)) {
        return "";
    }

    return resolved;
}
```

### 3.3 最小权限原则

```cpp
// 在 InstalldService 中
ErrCode InstalldService::CreateFile(const std::string& path) {
    // 1. 验证路径
    std::string normalizedPath = NormalizePath(path);
    if (normalizedPath.empty()) {
        return ERR_INSTALL_INVALID_PATH;
    }

    // 2. 检查符号链接
    struct stat st;
    if (lstat(normalizedPath.c_str(), &st) == 0 && S_ISLNK(st.st_mode)) {
        return ERR_INSTALL_INVALID_PATH;
    }

    // 3. 使用最小权限创建
    int fd = open(normalizedPath.c_str(), O_CREAT | O_WRONLY | O_TRUNC,
                  S_IRUSR | S_IWUSR);
    if (fd < 0) {
        return ERR_INSTALL_FILE_OPERATION_FAILED;
    }
    close(fd);
    return ERR_OK;
}
```

---

## 4. 相关安全组件

| 组件 | 用途 | 安全相关函数 |
|------|------|--------------|
| `access_token` | 权限管理 | VerifyAccessToken, VerifySystemAppByToken |
| `appverify` | 应用签名验证 | VerifySignature, CheckAppSignature |
| `security_code_signature` | 代码签名 | Sign, Verify |

---

## 5. 延伸阅读

- [05_AttackSurface](05_AttackSurface.md) - 攻击面分析
- [02_Architecture](02_Architecture.md) - 架构说明
- [08_FAQ](08_FAQ.md) - 安全相关常见问题
