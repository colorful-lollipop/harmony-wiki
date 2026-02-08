# 安全风险分析

## 目的

本文档对 `sys_installer` 进行安全风险评审，识别攻击面、信任边界和潜在可利用点。

## 适用范围

安全审计人员、系统架构师、安全开发人员。

## 威胁模型

### 资产识别

| 资产 | 价值 | 风险等级 |
|------|------|----------|
| 系统完整性 | 高 | 严重 |
| 用户数据 | 高 | 严重 |
| 更新包机密性 | 中 | 高 |
| 服务可用性 | 中 | 中 |

### 攻击者模型

| 攻击者类型 | 能力 | 威胁等级 |
|------------|------|----------|
| 本地恶意应用 | 有限的系统权限 | 中 |
| 特权提升攻击者 | root/系统权限 | 高 |
| 远程攻击者 | 网络访问能力 | 中 |
| 供应链攻击者 | 控制更新包分发 | 严重 |

## 攻击面清单

### 1. IPC 接口攻击面

**位置**: `frameworks/ipc_server/src/`

**接口**: SA 4101 和 SA 4103 的 IPC 接口

**风险**:
- 未授权调用
- 参数注入
- 拒绝服务

**证据**:
```cpp
// frameworks/ipc_server/src/sys_installer_server.cpp:329-350
bool CheckCallingPerm() {
    int32_t callingUid = OHOS::IPCSkeleton::GetCallingUid();
    if (callingUid == 0) {
        return true;  // Root 绕过
    }
    return callingUid == USER_UPDATE_AUTHORITY && IsPermissionGranted();
}
```

### 2. 文件系统攻击面

**位置**: `services/module_update/util/src/module_file.cpp`

**操作**: 文件创建、删除、挂载

**风险**:
- 路径遍历
- 符号链接攻击
- 竞争条件

**关键路径**:
- `/data/module_update_package/`
- `/data/module_update/active/`
- `/data/module_update/backup/`

### 3. 包解析攻击面

**位置**: `services/module_update/util/src/module_zip_helper.cpp`

**操作**: ZIP 包解压

**风险**:
- ZIP 炸弹
- 路径遍历 (ZipSlip)
- 恶意文件属性

### 4. 签名验证攻击面

**位置**: `frameworks/actions/verify_action/src/pkg_verify.cpp`

**操作**: 包签名验证

**风险**:
- 证书伪造
- 验证绕过
- 算法降级攻击

### 5. 设备映射攻击面

**位置**: `services/module_update/src/module_dm.cpp`

**操作**: Device Mapper 设备创建

**风险**:
- 设备名冲突
- 权限提升
- 资源耗尽

### 6. 网络攻击面

**位置**: `services/stream_update/src/stream_update.cpp`

**操作**: 网络流式更新

**风险**:
- 中间人攻击
- 下载劫持
- 断点续传劫持

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界图                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│  │  不可信区域  │     │  半可信区域  │     │   可信区域   │       │
│  │             │     │             │     │             │       │
│  │ 第三方应用   │────>│ SystemAbility│────>│ sys_installer│       │
│  │ 网络请求    │     │   框架       │     │   服务       │       │
│  └─────────────┘     └─────────────┘     └──────┬──────┘       │
│                                                  │              │
│                                         ┌────────┴────────┐     │
│                                         ▼                 ▼     │
│                                  ┌──────────┐      ┌──────────┐ │
│                                  │ 文件系统  │      │  内核    │ │
│                                  └──────────┘      └──────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 边界说明

| 边界 | 位置 | 控制措施 |
|------|------|----------|
| 应用边界 | IPC 调用前 | AccessToken 验证 |
| 服务边界 | SA 入口 | UID/权限检查 |
| 文件边界 | 文件操作 | 路径验证、权限检查 |
| 内核边界 | 设备映射 | 设备名验证 |

## 可利用点分析

### 1. 路径遍历漏洞

**风险等级**: 🔴 高

**位置**: `services/module_update/util/src/module_utils.cpp`

**证据**:
```cpp
// 路径处理代码
int32_t GetRealPath(const std::string& path, std::string& realPath) {
    char resolvedPath[PATH_MAX] = {0};
    if (realpath(path.c_str(), resolvedPath) == nullptr) {
        return ERR_INVALID_VALUE;
    }
    realPath = resolvedPath;
    return ERR_OK;
}
```

**触发条件**:
- 传入包含 `../` 的恶意路径
- 符号链接指向系统目录

**影响**:
- 文件写入系统目录
- 敏感文件覆盖

**修复建议**:
```cpp
// 建议增加前缀验证
bool IsPathWithinBase(const std::string& path, const std::string& base) {
    std::string realPath;
    if (GetRealPath(path, realPath) != ERR_OK) {
        return false;
    }
    return realPath.find(base) == 0;
}
```

### 2. 权限检查绕过

**风险等级**: 🟡 中

**位置**: `frameworks/ipc_server/src/sys_installer_server.cpp:329-350`

**证据**:
```cpp
bool CheckCallingPerm() {
    int32_t callingUid = OHOS::IPCSkeleton::GetCallingUid();
    if (callingUid == 0) {
        return true;  // Root 直接绕过
    }
    // ...
}
```

**触发条件**:
- Root 权限被获取
- UID 6666 被伪造

**影响**:
- 未授权的系统更新
- 恶意包安装

**修复建议**:
- 即使 root 也应记录审计日志
- 增加额外的签名验证

### 3. ZIP 炸弹/ZipSlip

**风险等级**: 🟡 中

**位置**: `services/module_update/util/src/module_zip_helper.cpp`

**证据**:
```cpp
// ZIP 解压代码
int32_t ExtractZipEntry(const std::string& zipPath, 
                        const std::string& entry,
                        const std::string& destPath) {
    // 解压逻辑
    // TODO: 需要验证 entry 是否包含路径遍历
}
```

**触发条件**:
- 包含 `../../../etc/passwd` 的 ZIP 条目
- 极高压缩比的 ZIP 炸弹

**影响**:
- 系统文件覆盖
- 磁盘空间耗尽

**修复建议**:
```cpp
// 建议增加验证
bool ValidateZipEntry(const std::string& entry, const std::string& basePath) {
    std::string fullPath = basePath + "/" + entry;
    std::string realPath;
    if (GetRealPath(fullPath, realPath) != ERR_OK) {
        return false;
    }
    return realPath.find(basePath) == 0;
}
```

### 4. 竞争条件

**风险等级**: 🟡 中

**位置**: `services/module_update/src/module_file_repository.cpp`

**证据**:
```cpp
// 文件操作可能存在 TOCTOU
int32_t WriteModuleFile(const std::string& path, const void* data, size_t size) {
    // 检查文件是否存在
    if (access(path.c_str(), F_OK) == 0) {
        // 时间窗口：文件可能被替换
        unlink(path.c_str());
    }
    // 写入新文件
    int fd = open(path.c_str(), O_WRONLY | O_CREAT, 0644);
    // ...
}
```

**触发条件**:
- 高并发文件操作
- 符号链接竞态

**影响**:
- 写入错误位置
- 权限提升

**修复建议**:
```cpp
// 使用原子操作
int32_t WriteModuleFileAtomic(const std::string& path, 
                              const void* data, size_t size) {
    std::string tmpPath = path + ".tmp";
    int fd = open(tmpPath.c_str(), O_WRONLY | O_CREAT | O_EXCL, 0644);
    // 写入临时文件
    // ...
    rename(tmpPath.c_str(), path.c_str());
}
```

### 5. 拒绝服务

**风险等级**: 🟡 中

**位置**: `frameworks/ipc_server/src/sys_installer_server.cpp`

**证据**:
```cpp
// 没有请求频率限制
int32_t SysInstallerServer::StartUpdatePackageZip(...) {
    // 直接处理请求，没有限流
    return installerManager_->StartUpdate(taskId, pkgPath);
}
```

**触发条件**:
- 大量并发 IPC 调用
- 超大更新包

**影响**:
- 服务不可用
- 资源耗尽

**修复建议**:
- 增加请求队列和限流
- 包大小限制
- 并发数限制

### 6. 签名验证绕过

**风险等级**: 🟢 低

**位置**: `frameworks/actions/verify_action/src/pkg_verify.cpp`

**证据**:
```cpp
// 签名验证逻辑
int32_t PkgVerify::VerifyPackage(const std::string& pkgPath) {
    // 验证包签名
    // 依赖系统证书存储
}
```

**触发条件**:
- 系统证书被篡改
- 验证逻辑被绕过

**影响**:
- 恶意包安装

**当前防护**:
- 证书链验证
- HVB 二次验证

## 安全建议汇总

### 高优先级

1. **路径验证强化**
   - 所有文件操作前验证路径前缀
   - 使用 `O_NOFOLLOW` 防止符号链接攻击

2. **审计日志**
   - 所有特权操作记录审计日志
   - 包括调用者 UID、PID、操作类型

3. **资源限制**
   - 包大小限制
   - 解压深度限制
   - 并发请求限制

### 中优先级

4. **竞争条件防护**
   - 使用原子文件操作
   - 文件锁保护

5. **输入验证**
   - 所有 IPC 参数严格验证
   - 字符串长度限制

6. **错误处理**
   - 不暴露内部错误细节
   - 统一错误码

### 低优先级

7. **代码混淆**
   - 关键验证逻辑混淆
   - 防逆向分析

8. **运行时防护**
   - SELinux 策略强化
   - seccomp 沙箱

## 安全检查清单

- [ ] 所有文件操作都有路径验证
- [ ] 所有 IPC 接口都有权限检查
- [ ] 所有包都有签名验证
- [ ] 所有错误都有审计日志
- [ ] 所有资源都有使用限制
- [ ] 所有并发都有同步保护

## 相关链接

- [对外 API](03_External_APIs.md)
- [内部 API](04_Internal_APIs.md)
- [架构说明](02_Architecture.md)

---

*证据来源*:
- `frameworks/ipc_server/src/sys_installer_server.cpp`: 权限检查
- `services/module_update/util/src/module_utils.cpp`: 路径处理
- `services/module_update/util/src/module_zip_helper.cpp`: ZIP 处理
- `frameworks/actions/verify_action/src/pkg_verify.cpp`: 签名验证
