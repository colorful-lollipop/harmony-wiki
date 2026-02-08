# 05_安全风险评审

> 基于代码证据的安全风险分析与修复建议。

## 1. 评审范围与方法

### 1.1 评审范围

| 类别 | 范围 |
|------|------|
| **代码路径** | `base/update/updateservice/` |
| **排除范围** | `test/` 测试目录 |
| **评审深度** | 静态代码分析 + 架构审查 |

### 1.2 威胁模型

```
┌─────────────────────────────────────────────────────────────────┐
│                     外部输入层                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │  JS API  │  │  IPC 调用│  │  网络    │  │  文件    │        │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
└───────┼─────────────┼─────────────┼─────────────┼──────────────┘
        │             │             │             │
        ▼             ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     信任边界                                     │
│  │  N-API 层 → SA 层 → Firmware 层 → Core 层 → 文件系统        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 攻击面清单

### 2.1 已识别攻击面

| 攻击面 | 类型 | 风险等级 |
|--------|------|----------|
| **N-API 接口** | JS API 调用 | 高 |
| **IPC 接口** | 21 个 SA 方法 | 高 |
| **文件操作** | 升级包读写 | 高 |
| **网络通信** | OTA 下载 | 中 |
| **系统权限** | Factory Reset | 高 |

### 2.2 信任边界

| 边界 | 说明 |
|------|------|
| **JS 层** | 应用进程，信任度低 |
| **SA 层** | 系统服务进程，信任度中 |
| **Core 层** | 系统能力，信任度高 |

---

## 3. 安全机制分析

### 3.1 权限校验机制

**位置**: `update_service.cpp:594-631`

#### 3.1.1 调用者类型校验

```cpp
bool UpdateService::IsCallerValid()
{
    auto callerTokenType = AccessTokenKit::GetTokenType(callerToken);
    
    switch (callerTokenType) {
        case TOKEN_HAP:
            // HAP 只允许系统应用
            return TokenIdKit::IsSystemAppByFullTokenID(callerFullTokenID);
            
        case TOKEN_NATIVE:
            // Native 只允许 root(uid=0) 和 edm(uid=3057)
            pid_t callerUid = IPCSkeleton::GetCallingUid();
            return callerUid == ROOT_UID || callerUid == EDM_UID;
            
        default:
            return false;
    }
}
```

**证据**: `update_service.cpp:594-613`

**评估**: ✅ 良好的权限隔离

---

#### 3.1.2 权限校验

```cpp
bool UpdateService::IsPermissionGranted(uint32_t code)
{
    std::string permission = "ohos.permission.UPDATE_SYSTEM";
    
    // Factory Reset 需要特殊权限
    if (code == FACTORY_RESET) {
        permission = "ohos.permission.FACTORY_RESET";
    } else if (code == FORCE_FACTORY_RESET) {
        permission = "ohos.permission.FORCE_FACTORY_RESET";
    }
    
    return AccessTokenKit::VerifyAccessToken(callerToken, permission)
           == PERMISSION_GRANTED;
}
```

**证据**: `update_service.cpp:615-631`

**评估**: ✅ 敏感操作有独立权限控制

---

### 3.2 N-API 权限检查

**位置**: `napi_common_utils.cpp:259-273`

```cpp
bool NapiCommonUtils::IsCallerValid()
{
    auto callerTokenType = AccessTokenKit::GetTokenType(callerToken);
    
    switch (callerTokenType) {
        case TOKEN_HAP:
            // HAP 必须是系统应用
            return TokenIdKit::IsSystemAppByFullTokenID(callerFullTokenID);
        default:
            return false;
    }
}
```

**证据**: `napi_common_utils.cpp:259-273`

**评估**: ✅ N-API 层有独立的权限检查

---

### 3.3 SELinux 隔离

**配置文件**: `updater_sa.cfg`

```json
{
    "services": [{
        "name": "updater_sa",
        "secon": "u:r:updater_sa:s0"
    }]
}
```

**RC 脚本**: `updater_sa.rc`

```bash
service updater_sa /system/bin/sa_main /system/profile/updater_sa.json
    seclabel u:r:updater_sa:s0
```

**证据**: `updater_sa.cfg:35`, `updater_sa.rc:5`

**评估**: ✅ 独立的 SELinux 上下文

---

## 4. 可被利用点与修复建议

### 4.1 高风险问题

#### 问题 1: 路径遍历风险

**证据 (路径)**: `local_updater.cpp` (TODO: 需确认具体文件和行号)

**问题描述**:
`verifyUpgradePackage` 和 `applyNewVersion` 方法接收文件路径参数，如果未正确校验路径，可能存在路径遍历漏洞。

**触发条件**:
```javascript
// 恶意应用可能尝试传入特殊路径
localUpdater.verifyUpgradePackage({
    packagePath: '../../../etc/passwd'
})
```

**影响**:
- 任意文件读取
- 敏感配置文件泄露

**修复建议**:
```cpp
// 在服务层添加路径校验
std::string ValidatePath(const std::string &path) {
    // 1. 必须是绝对路径
    if (path.empty() || path[0] != '/') {
        return "";
    }
    
    // 2. 规范化路径
    char resolved[PATH_MAX];
    if (realpath(path.c_str(), resolved) == nullptr) {
        return "";
    }
    
    // 3. 白名单目录校验
    std::string allowedDirs[] = {
        "/data/update/ota_package/",
        "/data/service/el1/public/update/"
    };
    
    for (const auto &dir : allowedDirs) {
        if (strncmp(resolved, dir.c_str(), dir.length()) == 0) {
            return std::string(resolved);
        }
    }
    
    return "";
}
```

**优先级**: P0 (高危)

---

#### 问题 2: Factory Reset 权限绕过风险

**证据 (路径)**: `update_service_restorer.cpp:36-61`

**问题描述**:
`GetCallingAppId()` 方法用于获取调用者身份，但未验证调用者是否为系统应用。

**触发条件**:
```cpp
// 任何有 FACTORY_RESET 权限的应用都可触发
restorer.factoryReset()
```

**影响**:
- 设备恢复出厂设置
- 用户数据丢失

**当前代码**:
```cpp
static std::string GetCallingAppId()
{
    auto callerTokenType = AccessTokenKit::GetTokenType(callerToken);
    
    if (callerTokenType == TOKEN_HAP) {
        HapTokenInfo hapTokenInfo;
        if (AccessTokenKit::GetHapTokenInfo(callerToken, hapTokenInfo) != 0) {
            return "";
        }
        return hapTokenInfo.bundleName;  // 仅记录，未校验
    }
    
    if (callerTokenType == TOKEN_NATIVE) {
        return std::to_string(IPCSkeleton::GetCallingUid());
    }
    
    return "";
}
```

**修复建议**:
```cpp
// 在 FactoryReset 前增加系统应用校验
if (!TokenIdKit::IsSystemAppByFullTokenID(callerFullTokenID)) {
    ENGINE_LOGE("FactoryReset denied: non-system app");
    return INT_NOT_SYSTEM_APP;
}
```

**优先级**: P0 (高危)

---

#### 问题 3: 网络下载安全

**证据 (路径)**: `dupdate_net_manager.cpp` (TODO: 需确认)

**问题描述**:
`download` 方法从网络下载升级包，未验证服务器证书和包完整性。

**触发条件**:
```javascript
// 下载不受信的升级包
updater.download({
    allowNetwork: 'CELLULAR'  // 使用移动网络
})
```

**影响**:
- 中间人攻击
- 恶意固件安装

**修复建议**:
```cpp
// 1. 强制 HTTPS
// 2. 证书固定 (Certificate Pinning)
// 3. 下载后验证签名
int32_t DownloadManager::Download(const std::string &url) {
    // 检查 URL 协议
    if (url.substr(0, 8) != "https://") {
        return ERR_INVALID_URL;
    }
    
    // 证书验证
    if (!VerifyServerCertificate(url)) {
        return ERR_CERTIFICATE_INVALID;
    }
    
    // 下载后签名验证
    if (!VerifyPackageSignature(downloadedPath)) {
        return ERR_SIGNATURE_INVALID;
    }
    
    return SUCCESS;
}
```

**优先级**: P0 (高危)

---

### 4.2 中风险问题

#### 问题 4: 缓冲区溢出风险

**证据 (路径)**: `stream_progress_thread.cpp` (TODO: 需确认)

**问题描述**:
下载进度处理可能存在缓冲区操作风险。

**触发条件**:
```javascript
// 服务器返回超长进度数据
```

**影响**:
- 远程代码执行
- 服务崩溃

**证据代码**:
```cpp
// 需要检查具体的缓冲区操作
```

**修复建议**:
```cpp
// 1. 使用安全的字符串函数 (strncpy_s, snprintf)
// 2. 添加边界检查
// 3. 启用 Address Sanitizer 测试
```

**优先级**: P1 (中危)

---

#### 问题 5: 竞态条件

**证据 (路径)**: `update_service.cpp:126-127` (clientProxyMap_)

**问题描述**:
回调注册/注销使用互斥锁，但可能存在 TOCTOU (Time-of-check to time-of-use) 竞态。

**触发条件**:
```cpp
// 并发调用 Register/Unregister
```

**当前代码**:
```cpp
std::lock_guard<std::mutex> lock(clientProxyMapLock_);
auto iter = clientProxyMap_.find(info);
// ...
```

**评估**: ✅ 使用了 `lock_guard`，竞态风险较低

**修复建议**:
```cpp
// 可考虑使用读写锁优化性能
std::shared_mutex clientProxyMapMutex_;
```

**优先级**: P2 (低危)

---

#### 问题 6: 日志信息泄露

**证据 (路径)**: `update_service.cpp:386-404` (Dump 函数)

**问题描述**:
`Dump` 函数可能输出敏感信息。

**当前代码**:
```cpp
void BuildVersionInfoDump(const int fd, const CheckResult &checkResult)
{
    dprintf(fd, "---------------------version info--------------------\n");
    dprintf(fd, "isExistNewVersion: %d\n", checkResult.isExistNewVersion);
    dprintf(fd, "PackageSize: %zu\n", checkResult.newVersionInfo.versionComponents[0].size);
    // ...
}
```

**影响**:
- 调试信息泄露
- 版本信息暴露

**评估**: ✅ `Dump` 仅用于调试，受 dump-level 控制

**修复建议**:
```cpp
// 在发布版本中限制敏感日志
void BuildVersionInfoDump(const int fd, const CheckResult &checkResult)
{
    #ifdef RELEASE_BUILD
    // 发布版不输出详细信息
    return;
    #endif
}
```

**优先级**: P3 (低危)

---

## 5. 安全加固建议

### 5.1 输入校验

| API | 输入参数 | 校验项 |
|-----|----------|--------|
| `verifyUpgradePackage` | packagePath | 路径白名单、扩展名 |
| `applyNewVersion` | miscFile, packageNames | 路径校验、权限校验 |
| `download` | url | URL 白名单、HTTPS 强制 |
| `setUpgradePolicy` | policy | 范围校验 |

### 5.2 输出编码

| 输出 | 场景 | 编码方式 |
|------|------|----------|
| BusinessError.message | JS 错误 | UTF-8 编码 |
| Dump 输出 | 调试信息 | 敏感字段脱敏 |

### 5.3 内存安全

| 措施 | 实施位置 | 状态 |
|------|----------|------|
| `-fstack-protector-strong` | 所有库 | ✅ 已启用 |
| `boundary_sanitize` | SA/N-API | ✅ 已启用 |
| `cfi` | SA/N-API | ✅ 已启用 |
| Bounds Checking | 字符串操作 | ⚠️ 需确认 |

### 5.4 密钥管理

| 密钥 | 用途 | 存储位置 |
|------|------|----------|
| 升级包签名密钥 | 包验证 | TODO: 需确认 |
| 签名密钥路径 | 参数传递 | 配置文件 |

---

## 6. 权限清单

### 6.1 系统权限

| 权限名 | 描述 | 受保护操作 |
|--------|------|------------|
| `ohos.permission.UPDATE_SYSTEM` | 系统升级 | 核心升级操作 |
| `ohos.permission.FACTORY_RESET` | 恢复出厂 | factoryReset |
| `ohos.permission.FORCE_FACTORY_RESET` | 强制恢复 | forceFactoryReset |

### 6.2 文件权限

| 路径 | 权限 | 所有者 | 说明 |
|------|------|--------|------|
| `/data/update/ota_package/` | 0770 | update:update | 升级包存储 |
| `/data/service/el1/public/update/` | 0751 | update:update | 数据根目录 |

---

## 7. 事件监控

### 7.1 安全事件

**文件**: `update_system_event.h`

```cpp
// 安全相关事件
EVENT_PERMISSION_VERIFY_FAILED    // 权限验证失败
EVENT_PKG_VERIFY_FAILED          // 包验证失败
```

### 7.2 HiSysEvent 配置

**文件**: `hisysevent.yaml`

**证据**: `hisysevent.yaml`

---

## 8. 安全检查清单

| 检查项 | 状态 | 说明 |
|--------|------|------|
| ✅ HAP 调用者校验 | 已实现 | `IsSystemAppByFullTokenID` |
| ✅ Native 调用者校验 | 已实现 | UID 0/3057 |
| ✅ 敏感操作权限校验 | 已实现 | FACTORY_RESET 独立校验 |
| ✅ SELinux 隔离 | 已实现 | u:r:updater_sa:s0 |
| ✅ 安全编译选项 | 已启用 | CFI, Stack Protector |
| ⚠️ 路径遍历防护 | 需确认 | 建议增加白名单 |
| ⚠️ 下载完整性校验 | 需确认 | 建议增加签名验证 |
| ⚠️ 参数范围校验 | 需确认 | 建议增加边界检查 |

---

## 9. 总结

### 9.1 总体评估

| 维度 | 评分 | 说明 |
|------|------|------|
| 权限模型 | 8/10 | 良好的权限隔离，但部分 API 需增强 |
| 输入校验 | 6/10 | 需增强路径和参数校验 |
| 内存安全 | 9/10 | 已启用多种安全编译选项 |
| 日志审计 | 7/10 | 有安全事件记录 |

### 9.2 建议优先级

| 优先级 | 问题 | 预期工作量 |
|--------|------|------------|
| P0 | 路径遍历防护 | 中 |
| P0 | Factory Reset 权限 | 低 |
| P0 | 下载完整性校验 | 高 |
| P1 | 参数范围校验 | 中 |

---

## 10. 下一步

- **架构设计**: [01_Architecture.md](./01_Architecture.md)
- **N-API 参考**: [02_N-API.md](./02_N-API.md)
- **构建配置**: [04_GN_Build.md](./04_GN_Build.md)
- **故障排查**: [06_Troubleshooting.md](./06_Troubleshooting.md)
