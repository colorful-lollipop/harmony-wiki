# 对外 N-API (JavaScript API)

## 目的

本文档列出 DLP 权限管理服务的所有 JavaScript (N-API) 接口，包括参数、返回值、对应的 C++ 实现、错误码和权限要求。

## 适用范围

- 目标读者：三方应用开发者、N-API 绑定开发者
- 覆盖内容：所有 JS 模块、方法、枚举、参数校验、错误码

---

## N-API 模块概览

### 模块列表

| 模块名 | 注册文件 | .so 文件 | 描述 |
|---------|----------|----------|------|
| `dlpPermission` | napi_dlp_permission_manager.cpp:49 | libdlppermission_napi.so | 主 DLP 权限接口 |
| `dlpSetDlpFeature` | napi_dlp_feature.cpp:197 | libdlpsetdlpfeature_napi.so | 特性开关接口 |
| `security.identifySensitiveContent` | napi_identify_sensitive_content.cpp:301 | identifysensitivecontent_napi.so | 敏感内容识别接口 |

### 模块注册点

#### dlpPermission 模块

```cpp
// 文件: interfaces/kits/dlp_permission/napi/src/napi_dlp_permission_manager.cpp:44-52
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "dlpPermission",
    .nm_priv = ((void*)0),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void DlpPermissionModuleRegister(void)
{
    napi_module_register(&_module);
}
```

#### dlpSetDlpFeature 模块

```cpp
// 文件: interfaces/kits/dlp_permission/napi/src/napi_dlp_feature.cpp:191-199
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Init,
    .nm_modname = "dlpSetDlpFeature",
    .nm_priv = ((void*)0),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void DlpFeatureModuleRegister(void)
{
    napi_module_register(&_module);
}
```

---

## dlpPermission 模块 - 静态方法

### API 清单表

| JS 方法名 | C++ 实现 | 同步/异步 | 权限要求 | 描述 |
|----------|----------|-----------|----------|------|
| `isDLPFile(uri: string): boolean` | `NapiDlpPermission::IsDlpFile()` | 同步 | - | 检查文件是否为 DLP 文件 |
| `getDLPPermissionInfo(uri: string): Promise<DLPPermissionInfo>` | `NapiDlpPermission::GetDLPPermissionInfo()` | 异步 | ACCESS_DLP_FILE | 获取 DLP 文件权限信息 |
| `getDLPSuffix(): string` | `NapiDlpPermission::GetDLPSuffix()` | 同步 | - | 获取 DLP 文件后缀 |
| `getOriginalFileName(dlpFileName: string): string` | `NapiDlpPermission::GetOriginalFileName()` | 同步 | - | 从 DLP 文件名获取原始文件名 |
| `isInSandbox(): boolean` | `NapiDlpPermission::IsInSandbox()` | 同步 | - | 检查是否在 DLP 沙箱中 |
| `getDlpSupportFileType(): string[]` | `NapiDlpPermission::GetDlpSupportFileType()` | 同步 | ACCESS_DLP_FILE | 获取支持的 DLP 文件类型 |
| `getDLPSupportedFileTypes(): string[]` | `NapiDlpPermission::GetDlpSupportFileType()` | 同步 | ACCESS_DLP_FILE | getDlpSupportFileType 别名 |
| `setRetentionState(docUriVec: string[]): Promise<void>` | `NapiDlpPermission::SetRetentionState()` | 异步 | ACCESS_DLP_FILE | 设置保留状态 |
| `cancelRetentionState(docUriVec: string[]): Promise<void>` | `NapiDlpPermission::CancelRetentionState()` | 异步 | ACCESS_DLP_FILE | 取消保留状态 |
| `getRetentionSandboxList(bundleName: string): Promise<RetentionSandboxInfo[]>` | `NapiDlpPermission::GetRetentionSandboxList()` | 异步 | ACCESS_DLP_FILE | 获取保留沙箱列表 |
| `getDLPFileAccessRecords(): Promise<VisitedDLPFileInfo[]>` | `NapiDlpPermission::GetDLPFileVisitRecord()` | 异步 | ACCESS_DLP_FILE | 获取 DLP 文件访问记录 |
| `startDLPManagerForResult(intent: Want): Promise<void>` | `NapiDlpPermission::StartDLPManagerForResult()` | 异步 | - | 启动 DLP 管理器 UI |
| `generateDLPFile(policy: DlpPolicy, uri: string): Promise<string>` | `NapiDlpPermission::GenerateDlpFile()` | 异步 | ACCESS_DLP_FILE 或 ENTERPRISE_ACCESS_DLP_FILE | 生成 DLP 文件 |
| `generateDlpFileForEnterprise(policy: DlpPolicy, uri: string): Promise<string>` | `NapiDlpPermission::GenerateDlpFileForEnterprise()` | 异步 | ENTERPRISE_ACCESS_DLP_FILE | 企业模式生成 DLP 文件 |
| `openDLPFile(uri: string, userId: number): Promise<DLPFile>` | `NapiDlpPermission::OpenDlpFile()` | 异步 | ACCESS_DLP_FILE | 打开 DLP 文件 |
| `installDLPSandbox(bundleName: string, dlpFileAccess: DLPFileAccess, userId: number): Promise<SandboxInfo>` | `NapiDlpPermission::InstallDlpSandbox()` | 异步 | ACCESS_DLP_FILE | 安装 DLP 沙箱 |
| `uninstallDLPSandbox(bundleName: string, appIndex: number, userId: number): Promise<void>` | `NapiDlpPermission::UninstallDlpSandbox()` | 异步 | ACCESS_DLP_FILE | 卸载 DLP 沙箱 |
| `getDLPGatheringPolicy(): Promise<boolean>` | `NapiDlpPermission::GetDlpGatheringPolicy()` | 异步 | ACCESS_DLP_FILE | 获取聚合策略 |
| `setSandboxAppConfig(configInfo: string): Promise<void>` | `NapiDlpPermission::SetSandboxAppConfig()` | 异步 | ACCESS_DLP_FILE | 设置沙箱应用配置 |
| `cleanSandboxAppConfig(): Promise<void>` | `NapiDlpPermission::CleanSandboxAppConfig()` | 异步 | ACCESS_DLP_FILE | 清理沙箱应用配置 |
| `getSandboxAppConfig(): Promise<string>` | `NapiDlpPermission::GetSandboxAppConfig()` | 异步 | ACCESS_DLP_FILE | 获取沙箱应用配置 |
| `isDLPFeatureProvided(): Promise<boolean>` | `NapiDlpPermission::IsDLPFeatureProvided()` | 异步 | - | 检查 DLP 特性是否提供 |
| `decryptDlpFile(uri: string, outputUri: string): Promise<void>` | `NapiDlpPermission::DecryptDlpFile()` | 异步 | ACCESS_DLP_FILE | 解密 DLP 文件 |
| `queryDlpPolicy(uri: string): Promise<DlpPolicy>` | `NapiDlpPermission::QueryDlpPolicy()` | 异步 | ACCESS_DLP_FILE | 查询 DLP 策略 |
| `setEnterprisePolicy(policy: string): Promise<void>` | `NapiDlpPermission::SetEnterprisePolicy()` | 异步 | ENTERPRISE_ACCESS_DLP_FILE | 设置企业策略 |
| `on(event: string, callback: Function): void` | `NapiDlpPermission::Subscribe()` | 同步 | - | 订阅 DLP 事件 |
| `off(event: string, callback?: Function): void` | `NapiDlpPermission::UnSubscribe()` | 同步 | - | 取消订阅 DLP 事件 |

---

## dlpPermission 模块 - DLPFile 类

### DLPFile 类方法

| JS 方法名 | C++ 实现 | 同步/异步 | 描述 |
|----------|----------|-----------|------|
| `constructor(uri: string)` | `NapiDlpPermission::DlpFile()` | 同步 | 创建 DLPFile 实例 |
| `addDLPLinkFile(linkFileUri: string, originalFileName: string): Promise<void>` | `NapiDlpPermission::AddDlpLinkFile()` | 异步 | 添加 DLP link 文件 |
| `stopFuseLink(): Promise<void>` | `NapiDlpPermission::StopDlpLinkFile()` | 异步 | 停止 FUSE link |
| `resumeFuseLink(): Promise<void>` | `NapiDlpPermission::RestartDlpLinkFile()` | 异步 | 恢复 FUSE link |
| `replaceDLPLinkFile(oldLinkFileUri: string, newLinkFileUri: string, originalFileName: string): Promise<void>` | `NapiDlpPermission::ReplaceDlpLinkFile()` | 异步 | 替换 DLP link 文件 |
| `deleteDLPLinkFile(linkFileUri: string): Promise<void>` | `NapiDlpPermission::DeleteDlpLinkFile()` | 异步 | 删除 DLP link 文件 |
| `recoverDLPFile(): Promise<void>` | `NapiDlpPermission::RecoverDlpFile()` | 异步 | 恢复 DLP 文件 |
| `closeDLPFile(): Promise<void>` | `NapiDlpPermission::CloseDlpFile()` | 异步 | 关闭 DLP 文件 |

---

## 导出的枚举

### ActionFlagType

操作权限标志枚举。

| 枚举值 | 数值 | 说明 |
|---------|------|------|
| `ACTION_INVALID` | 0 | 无效操作 |
| `ACTION_VIEW` | 1 | 查看文档 |
| `ACTION_SAVE` | 2 | 保存文档 |
| `ACTION_SAVE_AS` | 4 | 另存为新文件 |
| `ACTION_EDIT` | 8 | 编辑内容 |
| `ACTION_SCREEN_CAPTURE` | 16 | 截屏 |
| `ACTION_SCREEN_SHARE` | 32 | 屏幕共享 |
| `ACTION_SCREEN_RECORD` | 64 | 屏幕录制 |
| `ACTION_COPY` | 128 | 复制内容 |
| `ACTION_PRINT` | 256 | 打印文档 |
| `ACTION_EXPORT` | 512 | 导出内容 |
| `ACTION_PERMISSION_CHANGE` | 1024 | 修改权限 |

**代码证据**：
- 创建函数：`CreateEnumActionFlags()` (napi_dlp_permission.cpp)
- C++ 定义：`permission_policy.h` 中的 `ActionFlags`

### DLPFileAccess

DLP 文件访问权限枚举。

| 枚举值 | 数值 | 说明 |
|---------|------|------|
| `NO_PERMISSION` | 0 | 无权限 |
| `READ_ONLY` | 1 | 只读 |
| `CONTENT_EDIT` | 2 | 可编辑内容但不能另存 |
| `FULL_CONTROL` | 3 | 完全控制 |

**代码证据**：
- 创建函数：`CreateEnumDLPFileAccess()` (napi_dlp_permission.cpp)
- C++ 定义：`permission_policy.h` 中的 `DLPFileAccess` 枚举

### AccountType

账户类型枚举。

| 枚举值 | 数值 | 说明 |
|---------|------|------|
| `CLOUD_ACCOUNT` | 0 | 云账户 |
| `DOMAIN_ACCOUNT` | 1 | 域账户 |
| `APPLICATION_ACCOUNT` | 2 | 应用账户 |
| `ENTERPRISE_ACCOUNT` | 3 | 企业账户 |

**代码证据**：
- 创建函数：`CreateEnumAccountType()` (napi_dlp_permission.cpp)

### GatheringPolicyType

聚合策略枚举。

| 枚举值 | 数值 | 说明 |
|---------|------|------|
| `GATHERING` | 0 | 聚合模式 |
| `NON_GATHERING` | 1 | 非聚合模式 |

**代码证据**：
- 创建函数：`CreateEnumGatheringPolicy()` (napi_dlp_permission.cpp)

### ActionType

操作类型枚举。

| 枚举值 | 数值 | 说明 |
|---------|------|------|
| `NOT_OPEN` | 0 | 未打开 |
| `OPEN` | 1 | 已打开 |

**代码证据**：
- 创建函数：`CreateEnumActionType()` (napi_dlp_permission.cpp)

---

## dlpSetDlpFeature 模块

### 方法列表

| JS 方法名 | C++ 实现 | 描述 |
|----------|----------|------|
| `setDlpFeature(dlpFeatureInfo: DlpFeatureStatus): Promise<boolean>` | `NapiDlpFeature::SetDlpFeature()` | 设置 DLP 特性开关 |

### DlpFeatureStatus 枚举

| 枚举值 | 数值 | 说明 |
|---------|------|------|
| `NOT_ENABLED_FEATURE` | 0 | 未启用特性 |
| `ENABLED_FEATURE` | 1 | 已启用特性 |

---

## security.identifySensitiveContent 模块

### 方法列表

| JS 方法名 | C++ 实现 | 描述 |
|----------|----------|------|
| `scanFile(fileUri: string): Promise<ScanResult>` | `NapiIdentifySensitiveContent::ScanFile()` | 扫描文件敏感内容 |

**代码证据**：
- 文件：`interfaces/kits/identify_sensitive_content/napi/src/napi_identify_sensitive_content.cpp:301`

---

## 参数解析与校验

### 参数校验函数

所有 N-API 方法使用统一的参数校验函数（napi_common.cpp）：

| 函数 | 功能 | 文件路径 |
|------|------|----------|
| `NapiCheckArgc(env, argc, expected)` | 检查参数数量 | interfaces/kits/napi_common/src/napi_common.cpp |
| `GetInt64Value(env, value, result)` | 提取 int64 | 同上 |
| `GetStringValue(env, value, result)` | 提取 string | 同上 |
| `GetStringValueByKey(env, object, key, result)` | 提取对象属性（string） | 同上 |
| `GetBoolValueByKey(env, object, key, result)` | 提取对象属性（boolean） | 同上 |
| `GetInt32ValueByKey(env, object, key, result)` | 提取对象属性（int32） | 同上 |
| `GetVectorAuthUser(env, value, result)` | 提取授权用户数组 | 同上 |
| `ParseCallback(env, value)` | 提取回调函数 | 同上 |
| `IsStringLengthValid(env, value, minLen, maxLen)` | 校验字符串长度 | 同上 |
| `ThrowParamError(env, paramType, paramName)` | 抛出参数错误 | 同上 |

### 参数校验示例

```cpp
// 文件: napi_common.cpp
size_t argc = 3;
napi_value args[3] = {nullptr};

// 获取回调信息
napi_get_cb_info(env, info, &argc, args, nullptr, nullptr);

// 检查参数数量
NapiCheckArgc(env, argc, paramsLen);

// 提取字符串参数
std::string uri;
GetStringValue(env, args[0], uri);

// 提取回调函数
napi_value callback = ParseCallback(env, args[2]);
```

---

## 同步/异步模式

### 同步方法

以下方法在 JS 主线程同步执行：
- `isDLPFile`
- `getDLPSuffix`
- `getOriginalFileName`
- `isInSandbox`
- `getDlpSupportFileType`
- `on`
- `off`

### 异步方法

其他所有方法使用 `napi_async_work` 在工作线程执行，返回 Promise：

```
JS 线程
    ↓ 调用异步方法
Worker 线程
    ↓ XXXExecute() 执行
    ↓ 调用 SDK / IPC
Worker 线程完成
    ↓ XXXComplete() 回调
JS 线程
    ↓ 返回 Promise 结果
```

**代码证据**：
- 异步上下文：`napi_common.h` 中的 `AsyncContext` 基类
- 执行/完成模式：每个异步方法都有 `XXXExecute()` 和 `XXXComplete()` 函数

---

## 错误码与异常

### 错误码映射

错误码映射到 JS 异常消息（napi_error_msg.cpp）：

| 错误码 | 说明 |
|---------|------|
| `ERR_PARAM_INVALID` | 参数无效 |
| `ERR_PERMISSION_DENIED` | 权限拒绝 |
| `ERR_FILE_NOT_FOUND` | 文件未找到 |
| `ERR_SERVICE_UNAVAILABLE` | 服务不可用 |
| ... | (其他错误码） |

**代码证据**：
- 文件：`interfaces/kits/napi_common/src/napi_error_msg.cpp`

### 异常抛出模式

```cpp
// 参数错误
ThrowParamError(env, "string", "uri");

// 标准错误
napi_throw_error(env, nullptr, "Service unavailable");

// 类型错误
napi_throw_type_error(env, nullptr, "Expected string argument");
```

---

## 关键 API 调用链

### generateDLPFile 调用链

```mermaid
graph LR
    A[JavaScript] -->|generateDLPFile()| B[NapiDlpPermission::GenerateDlpFile()]
    B -->|参数校验| C[GenerateDlpFileExecute()]
    C -->|DlpPermissionClient::GenerateDlpCertificate()| D[SDK]
    D -->|IPC| E[DlpPermissionService::GenerateDlpCertificate()]
    E -->|CheckPermission()| F[PermissionManagerAdapter]
    E -->|加密| G[HUKS]
    E -->|生成 DLP 文件| E
    E -->|回调| D
    D -->|Promise resolve| C
    C -->|GenerateDlpFileComplete()| B
    B -->|返回| A
```

### openDLPFile 调用链

```mermaid
graph LR
    A[JavaScript] -->|openDLPFile()| B[NapiDlpPermission::OpenDlpFile()]
    B -->|参数校验| C[OpenDlpFileExecute()]
    C -->|DlpPermissionClient::ParseDlpCertificate()| D[SDK]
    D -->|IPC| E[DlpPermissionService::ParseDlpCertificate()]
    E -->|InstallDlpSandbox()| F[AbilityManager]
    E -->|解密| G[HUKS]
    E -->|创建 DLPFile 实例| E
    E -->|回调| D
    D -->|Promise resolve| C
    C -->|OpenDlpFileComplete()| B
    B -->|返回| A
```

---

## 权限检查

### 权限常量

| 权限常量 | 说明 |
|----------|------|
| `ohos.permission.ACCESS_DLP_FILE` | 标准 DLP 文件访问权限 |
| `ohos.permission.ENTERPRISE_ACCESS_DLP_FILE` | 企业 DLP 文件访问权限 |

### 系统应用检查

某些操作要求调用者为系统应用：

```cpp
bool NapiDlpPermission::IsSystemApp(napi_env env) {
    uint64_t fullTokenId = GetSelfTokenID();
    bool isSystemApp = AccessToken::TokenIdKit::IsSystemAppByFullTokenID(fullTokenId);
    return isSystemApp;
}
```

**代码证据**：
- 函数：`NapiDlpPermission::IsSystemApp()` (napi_dlp_permission.cpp)

---

## 相关跳转链接

- [架构说明](02_Architecture.md) - 了解数据流和时序
- [内部 API](04_Internal_API.md) - 查看 SDK 和 IPC 接口
- [安全风险评审](07_Security_Review.md) - 了解安全机制

---

最后更新时间：2026-02-06
