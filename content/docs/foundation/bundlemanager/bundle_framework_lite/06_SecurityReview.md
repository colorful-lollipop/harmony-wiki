# 安全风险评估 - Bundle Framework Lite

## 目录

- [输入验证缺陷](#输入验证缺陷)
- [内存安全问题](#内存安全问题)
- [权限与鉴权](#权限与鉴权)
- [并发安全](#并发安全)
- [逻辑漏洞](#逻辑漏洞)

---

## 输入验证缺陷

### R1: 路径遍历检查不完整（中危）

**位置**: `services/bundlemgr_lite/src/gt_bundle_extractor.cpp:301`

**证据**:
```cpp
// 只检查 "../" - 可被绕过
if (strstr(*bundleName, "../") != nullptr) {
    return ERR_APPEXECFWK_INSTALL_FAILED_PARSE_INVALID_BUNDLENAME;
}
```

**触发路径**:
```
恶意 HAP → BundleParser::Extract()
        → gt_bundle_extractor.cpp:301
        → 仅检查 "../"
        → 攻击者使用 "..\" 或 "..//"
        → 成功遍历到任意目录
```

**影响评估**:
- **可利用性**: 中（需要构造特定 HAP 包）
- **权限提升**: 可能（如果结合其他漏洞）
- **影响范围**: 文件系统任意操作

**修复建议**:
```cpp
// 使用 realpath() 进行路径规范化
char realPath[PATH_MAX] = {0};
if (realpath(*bundleName, realPath) == nullptr) {
    return ERR_APPEXECFWK_INSTALL_FAILED_PARSE_INVALID_BUNDLENAME;
}

// 或者使用更全面的检查
if (strstr(*bundleName, "..") != nullptr ||
    strstr(*bundleName, "%2e%2e") != nullptr ||
    strstr(*bundleName, "..\\") != nullptr) {
    return ERR_APPEXECFWK_INSTALL_FAILED_PARSE_INVALID_BUNDLENAME;
}
```

**参考**: `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon_handler.cpp:41-97` 有正确的实现

---

### R2: CheckRealPath 未使用 realpath()（低危）

**位置**: `services/bundlemgr_lite/src/bundle_util.cpp:76-104`

**证据**:
```cpp
bool BundleUtil::CheckRealPath(const char *path)
{
    if (strlen(path) > PATH_LENGTH) {
        return false;
    }
    // 拒绝包含 ".", "..", "./" 的路径
    for (; *next != '\0'; next++) {
        if (*next != '.') continue;
        next++;
        if (*next == '\0' || *next == '.' || *next == '/') {
            return false;  // 检测到路径遍历尝试
        }
    }
}
```

**触发路径**:
```
用户输入路径 → Install(hapPath, ...)
            → CheckRealPath(hapPath)
            → 仅检查字面字符串
            → 攻击者使用编码绕过或符号链接
            → 路径遍历成功
```

**影响评估**:
- **可利用性**: 低（需要绕过其他检查）
- **权限提升**: 低
- **影响范围**: 有限

**修复建议**:
```cpp
bool BundleUtil::CheckRealPath(const char *path)
{
    if (strlen(path) > PATH_LENGTH) {
        return false;
    }
    
    // 使用 realpath() 规范化路径并解析符号链接
    char realPath[PATH_MAX] = {0};
    if (realpath(path, realPath) == nullptr) {
        return false;
    }
    
    // 验证规范化后的路径
    if (strstr(realPath, "..") != nullptr) {
        return false;
    }
    
    return true;
}
```

---

### R3: BundleName Regex 异常未记录（信息）

**位置**: `services/bundlemgr_lite/src/bundle_parser.cpp:79-101`

**证据**:
```cpp
bool BundleParser::CheckBundleNameIsValid(const char *bundleName)
{
    std::string pattern { "([a-zA-Z0-9_]+\\.)+[a-zA-Z0-9_]+" };
    std::regex re(pattern);
    if (!std::regex_match(bundleName, re)) {
        return false;  // 异常被静默吞下
    }
}
```

**触发路径**:
```
恶意 bundleName → BundleParser::CheckBundleNameIsValid()
                 → regex 编译失败（异常）
                 → 静默返回 false
                 → 用户收到通用错误，无法调试
```

**影响评估**:
- **可利用性**: 信息（降低可调试性）
- **权限提升**: 否
- **影响范围**: 调试困难

**修复建议**:
```cpp
bool BundleParser::CheckBundleNameIsValid(const char *bundleName)
{
    try {
        std::string pattern { "([a-zA-Z0-9_]+\\.)+[a-zA-Z0-9_]+" };
        std::regex re(pattern);
        return std::regex_match(bundleName, re);
    } catch (const std::regex_error& e) {
        HILOG_ERROR(HILOG_MODULE_APP, "Regex error: %s", e.what());
        return false;
    }
}
```

---

## 内存安全问题

### R4: 潜在的缓冲区溢出（中危）

**位置**: 多个文件使用 `strcpy()`, `sprintf()`

**证据**:
```cpp
// BundleUtil 中多处使用不安全的字符串函数
char path[PATH_MAX];
strcpy(path, sourcePath);  // 如果 sourcePath > PATH_MAX，溢出
```

**触发路径**:
```
超长路径输入 → strcpy(path, longInput)
             → 栈溢出
             → 覆盖返回地址
             → 任意代码执行
```

**影响评估**:
- **可利用性**: 中（需要超长输入）
- **权限提升**: 高（可能 RCE）
- **影响范围**: 任意代码执行

**修复建议**:
```cpp
// 使用安全的字符串函数
strncpy(path, sourcePath, PATH_MAX - 1);
path[PATH_MAX - 1] = '\0';

// 或者使用 std::string
std::string path = sourcePath;
if (path.length() >= PATH_MAX) {
    return ERR_APPEXECFWK_INSTALL_FAILED_INVALID_PATH;
}
```

**缓解措施**: 已链接 `bounds_checking_function` 库（`bundle.json:39`）

---

### R5: 潜在的 Use-After-Free（低危）

**位置**: `bundle_map.cpp` - Bundle 信息管理

**证据**:
```cpp
// 假设场景：异步回调释放后继续使用指针
BundleInfo *info = QueryBundleInfo(bundleName);
RegisterCallback(info);  // 异步操作
// ... 回调可能删除 info
UpdateBundleInfo(info);  // 如果 info 已被释放，UAF
```

**触发路径**:
```
并发卸载和查询 → QueryBundleInfo()
                    → 卸载操作删除 info
                    → QueryBundleInfo() 返回悬空指针
                    → 使用悬空指针
                    → 崩溃或任意读写
```

**影响评估**:
- **可利用性**: 低（需要时序竞争）
- **权限提升**: 中（可能信息泄露或崩溃）
- **影响范围**: 崩溃或信息泄露

**修复建议**:
```cpp
// 使用智能指针或引用计数
std::shared_ptr<BundleInfo> info = QueryBundleInfo(bundleName);
RegisterCallback(info);
// ... 自动管理生命周期

// 或者使用拷贝而非指针
BundleInfo infoCopy = *QueryBundleInfo(bundleName);
UpdateBundleInfo(&infoCopy);
```

---

## 权限与鉴权

### R6: 调试模式签名绕过（低危）

**位置**: `services/bundlemgr_lite/src/hap_sign_verify.cpp`

**证据**:
```cpp
#ifdef OHOS_DEBUG
    if (ManagerService::GetInstance().IsSignMode()) {
        errorCode = HapSignVerify::VerifySignature(path, signatureInfo);
    }
#else
    errorCode = HapSignVerify::VerifySignature(path, signatureInfo);  // Release 始终强制
#endif
```

**触发路径**:
```
调试构建 + IsSignMode()=false → 跳过签名验证
                              → 安装任意 HAP
                              → 恶意应用获得系统权限
```

**影响评估**:
- **可利用性**: 低（需要调试构建访问）
- **权限提升**: 高（完全绕过签名）
- **影响范围**: 恶意应用安装

**修复建议**:
```cpp
// 始终强制签名验证，仅在生产环境关闭调试日志
uint8_t HapSignVerify::VerifySignature(const std::string &hapFilepath, 
                                     SignatureInfo &signatureInfo)
{
    VerifyResult verifyResult;
    int32_t ret = APPVERI_AppVerify(hapFilepath.c_str(), &verifyResult);
    uint8_t errorCode = SwitchErrorCode(ret);
    
    // 调试模式下仅记录，不跳过验证
#ifdef OHOS_DEBUG
    if (errorCode != ERR_OK) {
        HILOG_DEBUG(HILOG_MODULE_APP, "Signature failed: %d", errorCode);
    }
#endif
    
    // 始终返回验证结果
    return errorCode;
}
```

---

### R7: UID 检查绕过风险（中危）

**位置**: `services/bundlemgr_lite/bundle_daemon/src/bundle_daemon.cpp:108-111`

**证据**:
```cpp
// Bundle Daemon 权限检查
if (!CheckPermission()) {
    PRINTE("BundleDaemon", "permission denied");
    return;
}
```

**触发路径**:
```
攻击者伪造 UID → SAMGR_GetInstance()->GetFeatureApi()
                 → 伪造 IPC 消息，伪造 UID=7
                 → BundleDaemon::CheckPermission() 通过
                 → 执行高权限操作
```

**影响评估**:
- **可利用性**: 中（需要能伪造 UID）
- **权限提升**: 高（获得 root 等价）
- **影响范围**: 任意文件操作

**修复建议**:
```cpp
// 使用更健壮的权限验证
bool BundleDaemon::CheckPermission()
{
    // 1. 验证调用者 UID
    uid_t callerUid = GetCallingUid();
    if (callerUid != BMS_UID) {
        HILOG_ERROR(HILOG_MODULE_APP, "Invalid caller UID: %d", callerUid);
        return false;
    }
    
    // 2. 验证 PID 是否匹配
    pid_t callerPid = GetCallingPid();
    if (!IsBMSProcess(callerPid)) {
        HILOG_ERROR(HILOG_MODULE_APP, "Invalid caller PID: %d", callerPid);
        return false;
    }
    
    return true;
}
```

---

## 并发安全

### R8: 竞态条件（TOCTOU）（低危）

**位置**: 文件系统操作

**证据**:
```cpp
// Time-of-Check to Time-of-Use 模式
if (CheckFileExists(path)) {  // Check
    // ... 攻击者在这期间删除/替换文件
    ReadFile(path);  // Use（已非原文件）
}
```

**触发路径**:
```
并发操作 → CheckFileExists() 返回 true
           → 攻击者替换文件为符号链接
           → ReadFile() 读取符号链接目标
           → 信息泄露或任意文件读取
```

**影响评估**:
- **可利用性**: 低（需要精确时序）
- **权限提升**: 中（可能绕过检查）
- **影响范围**: 文件系统操作异常

**修复建议**:
```cpp
// 原子操作：检查和使用在同一个系统调用
int fd = open(path, O_RDONLY);
if (fd < 0) {
    return ERR_FILE_NOT_FOUND;
}
// ... 使用 fd 进行所有操作，不依赖路径
read(fd, buffer, size);
close(fd);
```

---

## 逻辑漏洞

### R9: 系统应用保护绕过（中危）

**位置**: `services/bundlemgr_lite/src/bundle_installer.cpp:425-438`

**证据**:
```cpp
uint8_t BundleInstaller::Uninstall(const char *bundleName, const InstallParam &installParam)
{
    BundleInfo *bundleInfo = ManagerService::GetInstance().QueryBundleInfo(bundleName);
    if (bundleInfo->isSystemApp) {
        return ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_UNINSTALLABLE;
    }
}
```

**潜在问题**: 如果 `bundleInfo` 为 null，`bundleInfo->isSystemApp` 会崩溃

**触发路径**:
```
攻击者卸载不存在应用 → QueryBundleInfo() 返回 null
                       → bundleInfo->isSystemApp 访问 null 指针
                       → 崩溃（DoS）
```

**影响评估**:
- **可利用性**: 低（仅导致崩溃）
- **权限提升**: 否
- **影响范围**: 服务拒绝

**修复建议**:
```cpp
uint8_t BundleInstaller::Uninstall(const char *bundleName, const InstallParam &installParam)
{
    BundleInfo *bundleInfo = ManagerService::GetInstance().QueryBundleInfo(bundleName);
    if (bundleInfo == nullptr) {
        return ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_FOUND;
    }
    if (bundleInfo->isSystemApp) {
        return ERR_APPEXECFWK_UNINSTALL_FAILED_BUNDLE_NOT_UNINSTALLABLE;
    }
}
```

---

### R10: 资源耗尽（低危）

**位置**: HAP 安装过程

**证据**:
```cpp
// 无限制的文件创建
while (files.hasNext()) {
    CreateFile(file.path);  // 无数量限制
}
```

**触发路径**:
```
恶意 HAP → 包含大量文件
           → 逐个创建文件
           → 耗尽磁盘空间
           → 系统不可用（DoS）
```

**影响评估**:
- **可利用性**: 低（需要大量磁盘空间）
- **权限提升**: 否
- **影响范围**: 服务拒绝

**修复建议**:
```cpp
// 添加资源限制
uint32_t MAX_FILES_PER_BUNDLE = 1000;
uint32_t MAX_TOTAL_SIZE_PER_BUNDLE = 100 * 1024 * 1024; // 100MB

uint32_t fileCount = 0;
uint64_t totalSize = 0;
while (files.hasNext()) {
    if (fileCount++ >= MAX_FILES_PER_BUNDLE) {
        return ERR_APPEXECFWK_INSTALL_FAILED_TOO_MANY_FILES;
    }
    if (totalSize + file.size() >= MAX_TOTAL_SIZE_PER_BUNDLE) {
        return ERR_APPEXECFWK_INSTALL_FAILED_BUNDLE_TOO_LARGE;
    }
    totalSize += file.size();
    CreateFile(file.path);
}
```

---

## 风险汇总表

| ID | 风险 | 严重性 | 可利用性 | 影响范围 | 修复难度 |
|----|------|----------|----------|----------|----------|
| R1 | 路径遍历检查不完整 | 中 | 中 | 文件系统 | 低 |
| R2 | CheckRealPath 未使用 realpath() | 低 | 低 | 有限 | 低 |
| R3 | Regex 异常未记录 | 信息 | 否 | 调试困难 | 低 |
| R4 | 潜在缓冲区溢出 | 中 | 中 | 任意代码执行 | 中 |
| R5 | 潜在 Use-After-Free | 低 | 低 | 崩溃/泄露 | 中 |
| R6 | 调试模式签名绕过 | 低 | 低 | 恶意应用安装 | 低 |
| R7 | UID 检查绕过风险 | 中 | 中 | 高权限文件操作 | 中 |
| R8 | TOCTOU 竞态条件 | 低 | 低 | 文件操作异常 | 中 |
| R9 | 系统应用保护绕过 | 中 | 低 | 服务拒绝 | 低 |
| R10 | 资源耗尽 | 低 | 低 | 服务拒绝 | 中 |

---

## 相关文档

- [攻击面分析](05_AttackSurface.md) - 外部输入和敏感操作
- [架构与数据流](03_Architecture.md) - 理解信任边界
- [内部实现细节](08_Internals.md) - 深入理解代码逻辑

---

**最后更新**: 2026-02-07
