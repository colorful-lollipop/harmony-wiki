# 关键调用链 (Callgraphs)

## 目的

本文档列出 DLP 权限服务关键入口到核心逻辑的调用链，帮助开发者理解代码执行路径。

## 适用范围

- 目标读者：代码审核员、性能优化者
- 覆盖内容：N-API 到 Service 的关键调用路径

---

## generateDLPFile 调用链

### JavaScript → N-API → SDK → Service 层

```
JavaScript 应用
    ↓ generateDLPFile(policy, uri)
N-API 层
    ↓ NapiDlpPermission::GenerateDlpFile()
    ↓ 参数校验（NapiCheckArgc, GetStringValue）
    ↓ GenerateDlpFileExecute()
    ↓ DlpPermissionClient::GenerateDlpCertificate(policy, callback)
SDK 层
    ↓ DlpPermissionClient::GetService()
    ↓ iface_cast<IDlpPermissionService>(proxy)
    ↓ GenerateDlpCertificate(policy, callback) IPC 调用
Service 层
    ↓ DlpPermissionService::GenerateDlpCertificate(policy, callback)
    ↓ PermissionManagerAdapter::CheckPermission("ohos.permission.ACCESS_DLP_FILE")
    ↓ 生成证书
    ↓ 调用 HUKS 加密
    ↓ 生成 DLP 文件
    ↓ 回调通知结果
```

### 关键代码路径

| 步骤 | 文件路径 | 行号/符号 |
|------|----------|-----------|
| JS 入口 | interfaces/kits/dlp_permission/napi/src/napi_dlp_permission.cpp | GenerateDlpFile() |
| 参数校验 | interfaces/kits/napi_common/src/napi_common.cpp | GetStringValue(), NapiCheckArgc() |
| 客户端调用 | interfaces/inner_api/dlp_permission/src/dlp_permission_client.cpp | GenerateDlpCertificate() |
| 服务入口 | services/dlp_permission/sa/sa_main/dlp_permission_service.cpp | GenerateDlpCertificate() |
| 权限检查 | services/dlp_permission/sa/sa_common/permission_manager_adapter.cpp | CheckPermission() |

---

## openDLPFile 调用链

### JavaScript → N-API → SDK → Service → Ability Manager

```
JavaScript 应用
    ↓ openDLPFile(uri, userId)
N-API 层
    ↓ NapiDlpPermission::OpenDlpFile()
    ↓ OpenDlpFileExecute()
    ↓ DlpPermissionClient::ParseDlpCertificate(cert, callback, appId, offlineAccess)
SDK 层
    ↓ ParseDlpCertificate() IPC 调用
Service 层
    ↓ DlpPermissionService::ParseDlpCertificate(cert, callback, appId, offlineAccess)
    ↓ PermissionManagerAdapter::CheckPermission()
    ↓ 解析证书
    ↓ 检查访问策略
    ↓ AbilityManager::InstallDlpSandbox(bundleName, access, userId)
Ability Manager
    ↓ 启动沙箱应用
    ↓ 返回沙箱 PID/UID
Service 层
    ↓ 设置沙箱权限（InsertDlpSandboxInfo）
    ↓ 触发沙箱变化回调
    ↓ 创建 FUSE mount
    ↓ 回调通知结果
```

### 关键代码路径

| 步骤 | 文件路径 | 行号/符号 |
|------|----------|-----------|
| JS 入口 | interfaces/kits/dlp_permission/napi/src/napi_dlp_permission.cpp | OpenDlpFile() |
| 客户端调用 | interfaces/inner_api/dlp_permission/src/dlp_permission_client.cpp | ParseDlpCertificate() |
| 服务入口 | services/dlp_permission/sa/sa_main/dlp_permission_service.cpp | ParseDlpCertificate() |
| 权限检查 | services/dlp_permission/sa/sa_common/permission_manager_adapter.cpp | CheckPermission() |
| 沙箱安装 | services/dlp_permission/sa/sa_main/dlp_permission_service.cpp | InstallDlpSandbox() |

---

## FUSE 文件操作调用链

### 沙箱应用 → FUSE → DLP 解析

```
沙箱应用
    ↓ open/read/write FUSE 文件
FUSE Daemon
    ↓ fuse_loop() 接收请求
    ↓ dlp_fuse_operations (open, read, write, etc.)
FUSE 层
    ↓ DlpFuseHelper 处理请求
    ↓ 检查权限（CheckSandboxFlagWithService）
DLP 解析层
    ↓ DlpFile::Read() / DlpFile::Write()
    ↓ DlpFileManager::GetFile()
    ↓ DlpCrypt::Decrypt() / Encrypt()
    ↓ 返回数据
FUSE 层
    ↓ 返回数据给沙箱应用
```

### 关键代码路径

| 步骤 | 文件路径 | 行号/符号 |
|------|----------|-----------|
| FUSE daemon | interfaces/inner_api/dlp_fuse/src/fuse_daemon.cpp | DlpFuseStart() |
| FUSE 操作 | interfaces/inner_api/dlp_fuse/src/dlp_fuse_helper.cpp | dlp_fuse_operations |
| DLP 文件读取 | interfaces/inner_api/dlp_parse/src/dlp_file.cpp | Read() |
| 加密/解密 | interfaces/inner_api/dlp_parse/src/dlp_crypt.cpp | Decrypt() / Encrypt() |

---

## 权限检查调用链

### IPC 请求 → PermissionManagerAdapter → AccessTokenKit

```
IPC 请求
    ↓ DlpPermissionService::GenerateDlpCertificate()
Service 层
    ↓ PermissionManagerAdapter::CheckPermission(permission)
    ↓ IPCSkeleton::GetCallingTokenID()
    ↓ IsSaCall() 检查是否为 SA 调用
    ↓ 如果非 SA 调用
        ↓ CheckPermissionForConnect()
        ↓ AccessTokenKit::GetTokenType(callingToken)
        ↓ GetHapTokenInfo(callingToken)
        ↓ GetBundleInfoV9(bundleName, signature_info)
        ↓ 验证 appIdentifier
        ↓ AccessTokenKit::VerifyAccessToken(callingToken, permission)
    ↓ 返回权限检查结果
```

### 关键代码路径

| 步骤 | 文件路径 | 行号/符号 |
|------|----------|-----------|
| 服务入口 | services/dlp_permission/sa/sa_main/dlp_permission_service.cpp | GenerateDlpCertificate() |
| 权限适配器 | services/dlp_permission/sa/sa_common/permission_manager_adapter.cpp | CheckPermission() |
| SA 调用检测 | services/dlp_permission/sa/sa_main/dlp_permission_service.cpp | IsSaCall() |
| Token 类型获取 | services/dlp_permission/sa/sa_common/permission_manager_adapter.cpp | AccessTokenKit::GetTokenType() |

---

## 相关跳转链接

- [架构说明](02_Architecture.md) - 查看完整架构和数据流
- [对外 N-API](03_NAPI.md) - 查看 N-API 接口
- [内部 API](04_Internal_API.md) - 查看 SDK 接口

---

最后更新时间：2026-02-06
