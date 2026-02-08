# 内部 API (Internal API)

## 目的

本文档描述 DLP 权限管理服务的内部接口，包括模块间接口、依赖关系、稳定性和可替换点。

## 适用范围

- 目标读者：DLP 权限管理服务的内部开发者
- 覆盖内容：模块接口、依赖方向、生命周期、错误传播、稳定性标注

---

## 模块接口概览

### Inner API 库列表

| 库名称 | 头文件路径 | 稳定性 | 说明 |
|---------|----------|----------|------|
| `libdlp_permission_sdk.so` | interfaces/inner_api/dlp_permission/include/ | 稳定 | 主 SDK，包含 DLP 权限管理接口 |
| `libdlp_permission_common_interface.so` | interfaces/inner_api/dlp_permission/include/ | 稳定 | 通用接口库 |
| `libdlpparse.so` | interfaces/inner_api/dlp_parse/include/ | 稳定 | DLP 文件解析库（公共） |
| `libdlpparse_inner.so` | interfaces/inner_api/dlp_parse/include/ | 稳定 | DLP 文件解析库（内部） |
| `libdlp_fuse.so` | interfaces/inner_api/dlp_fuse/include/ | 稳定 | FUSE 文件系统库 |
| `libdlp_setconfig_sdk.so` | interfaces/inner_api/dlp_set_config/include/ | 稳定 | 配置设置 SDK |

---

## dlp_permission SDK 接口

### SDK 导出头文件

**证据**：bundle.json:76-89

| 头文件 | 路径 | 用途 |
|--------|------|------|
| `dlp_permission_callback.h` | interfaces/inner_api/dlp_permission/include/ | 回调接口定义 |
| `dlp_permission_kit.h` | interfaces/inner_api/dlp_permission/include/ | SDK 主接口 |
| `dlp_sandbox_callback_info.h` | interfaces/inner_api/dlp_permission/include/ | 沙箱回调信息 |
| `dlp_sandbox_change_callback_customize.h` | interfaces/inner_api/dlp_permission/include/ | 沙箱变化回调自定义接口 |
| `open_dlp_file_callback_customize.h` | interfaces/inner_api/dlp_permission/include/ | 打开文件回调自定义接口 |
| `open_dlp_file_callback_info.h` | interfaces/inner_api/dlp_permission/include/ | 打开文件回调信息 |
| `cert_parcel.h` | frameworks/common/include/ | 证书 Parcel 结构 |
| `permission_policy.h` | frameworks/common/include/ | 权限策略定义 |
| `retention_sandbox_info.h` | frameworks/common/include/ | 保留沙箱信息 |
| `visited_dlp_file_info.h` | frameworks/common/include/ | 访问记录信息 |

### DlpPermissionClient 类

**文件**：`interfaces/inner_api/dlp_permission/src/dlp_permission_client.cpp`

**职责**：SDK 客户端单例，管理与服务连接

**关键方法**：

| 方法 | 说明 | 代码证据 |
|------|------|----------|
| `GetInstance()` | 获取客户端单例 | dlp_permission_client.cpp |
| `GetService()` | 获取服务代理，延迟加载 | dlp_permission_client.cpp |
| `InitClient()` | 初始化客户端，注册 death recipient | dlp_permission_client.cpp |
| `OnRemoteDied()` | 服务死亡回调，重新连接 | dlp_permission_client.cpp |

**连接模式**：
```cpp
// 1. 获取 SystemAbilityManager
sptr<ISystemAbilityManager> samgr = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();

// 2. 获取服务
sptr<IRemoteObject> object = samgr->GetSystemAbility(SA_ID_DLP_PERMISSION_SERVICE);

// 3. 转换为代理
proxy = iface_cast<IDlpPermissionService>(object);
```

---

## IPC 接口定义

### IDlpPermissionService 接口

**文件**：`interfaces/inner_api/dlp_permission/IDlpPermissionService.idl`

**接口描述**：DLP 权限管理服务 IPC 接口

**方法分组**：

#### 1. 证书操作
| 方法 | IDL 行号 | 参数 | 返回值 |
|------|----------|------|--------|
| `GenerateDlpCertificate` | 31-33 | DlpPolicyParcel, callback | void (async) |
| `ParseDlpCertificate` | 34-38 | CertParcel, callback, appId, offlineAccess | void (async) |

#### 2. 沙箱管理
| 方法 | IDL 行号 | 参数 | 返回值 |
|------|----------|------|--------|
| `InstallDlpSandbox` | 49-54 | bundleName, dlpFileAccess, userId, uri | SandboxInfo |
| `UninstallDlpSandbox` | 55 | bundleName, appIndex, userId | void |
| `GetSandboxExternalAuthorization` | 56-59 | sandboxUid, want | SandBoxExternalAuthorType |

#### 3. 权限查询
| 方法 | IDL 行号 | 参数 | 返回值 |
|------|----------|------|--------|
| `QueryDlpFileCopyableByTokenId` | 61 | tokenId | boolean |
| `QueryDlpFileAccess` | 62 | - | DLPPermissionInfoParcel |
| `IsInDlpSandbox` | 63 | - | boolean |

#### 4. 回调注册
| 方法 | IDL 行号 | 参数 | 返回值 |
|------|----------|------|--------|
| `RegisterDlpSandboxChangeCallback` | 65 | cb (IRemoteObject) | - |
| `UnRegisterDlpSandboxChangeCallback` | 66 | - | boolean |
| `RegisterOpenDlpFileCallback` | 67 | cb (IRemoteObject) | - |
| `UnRegisterOpenDlpFileCallback` | 68 | cb (IRemoteObject) | - |

#### 5. MDM 策略
| 方法 | IDL 行号 | 参数 | 返回值 |
|------|----------|------|--------|
| `SetMDMPolicy` | 81 | appIdList | void |
| `GetMDMPolicy` | 82 | - | String[] |
| `RemoveMDMPolicy` | 83 | - | void |

#### 6. 其他
| 方法 | IDL 行号 | 参数 | 返回值 |
|------|----------|------|--------|
| `GetWaterMark` | 39-41 | waterMarkConfig, callback | void (async) |
| `SetWaterMark` | 60 | pid | void |
| `GetAbilityInfos` | 44-48 | want, flags, userId | AbilityInfo[] |
| `GetDomainAccountNameInfo` | 42-43 | - | String (out) |
| `GetDlpSupportFileType` | 64 | - | String[] |
| `GetDlpGatheringPolicy` | 69 | - | boolean |
| `SetRetentionState` | 70 | docUriVec | void |
| `CancelRetentionState` | 71 | docUriVec | void |
| `GetRetentionSandboxList` | 72 | bundleName | RetentionSandBoxInfo[] |
| `ClearUnreservedSandbox` | 73 | - | void |
| `GetDLPFileVisitRecord` | 74 | - | VisitedDLPFileInfo[] |
| `SetSandboxAppConfig` | 75 | configInfo | void |
| `CleanSandboxAppConfig` | 76 | - | void |
| `GetSandboxAppConfig` | 77 | - | String |
| `IsDLPFeatureProvided` | 78 | - | boolean |
| `SetDlpFeature` | 79 | dlpFeatureInfo | boolean |
| `SetReadFlag` | 80 | uid | void |
| `SetEnterprisePolicy` | 84 | policy | void |
| `SetFileInfo` | 85 | uri, fileInfo | void |

---

## dlp_parse SDK 接口

### SDK 导出头文件

**证据**：bundle.json:102-110

| 头文件 | 路径 | 用途 |
|--------|------|------|
| `dlp_crypt.h` | interfaces/inner_api/dlp_parse/include/ | 加密/解密接口 |
| `dlp_file_kits.h` | interfaces/inner_api/dlp_parse/include/ | DLP 文件工具集 |
| `dlp_file_manager.h` | interfaces/inner_api/dlp_parse/include/ | DLP 文件管理器 |
| `dlp_file.h` | interfaces/inner_api/dlp_parse/include/ | DLP 文件类 |
| `dlp_raw_file.h` | interfaces/inner_api/dlp_parse/include/ | 原始文件接口 |
| `dlp_zip_file.h` | interfaces/inner_api/dlp_parse/include/ | ZIP 文件处理 |

### 关键类

| 类 | 文件 | 职责 |
|-----|------|------|
| `DlpFile` | dlp_file.h/.cpp | DLP 文件封装，支持读写操作 |
| `DlpFileManager` | dlp_file_manager.h/.cpp | DLP 文件管理器，跟踪打开的文件 |
| `DlpCrypt` | dlp_crypt.h/.cpp | 加密/解密操作 |

---

## dlp_fuse SDK 接口

### SDK 导出头文件

**证据**：bundle.json:119

| 头文件 | 路径 | 用途 |
|--------|------|------|
| `dlp_fuse_fd.h` | interfaces/inner_api/dlp_fuse/include/ | FUSE 文件描述符接口 |

### 关键函数

| 函数 | 文件 | 职责 |
|-----|------|------|
| `DlpFuseStart()` | fuse_daemon.cpp | 启动 FUSE daemon |
| `DlpCreateLinkFile()` | dlp_link_file.cpp | 创建 link 文件 |
| `DlpStopFuse()` | fuse_daemon.cpp | 停止 FUSE daemon |

---

## 模块依赖方向

### 依赖图

```
N-API 层 (interfaces/kits/napi*)
    ↓ 依赖
Inner API 层 (interfaces/inner_api/*)
    ↓ 依赖
Service 层 (services/*)
    ↓ 依赖
外部服务 (samgr, access_token, huks, etc.)
```

### 依赖方向原则

1. **单向依赖**：上层依赖下层，避免循环依赖
2. **接口隔离**：通过 IDL 定义接口，避免直接依赖实现
3. **SDK 模式**：Inner API 层通过 SDK 访问 Service 层

### 避免循环依赖

- Service 层**不依赖** N-API 层
- Service 层通过 IDL 接口与客户端通信
- Inner API 层**不直接依赖** Service 层实现，只依赖接口

---

## 稳定/不稳定接口标注

### 稳定接口 (Stable API)

以下接口承诺向后兼容，可长期使用：

| 接口 | 稳定性依据 |
|------|----------|
| `DlpPermissionClient` | SDK 导出，包含在 bundle.json 的 inner_kits 中 |
| `IDlpPermissionService` IDL | IDL 接口，版本化管理 |
| `DlpFile` 类 | 公共 SDK，头文件稳定 |
| `DlpFileAccess` 枚举 | permission_policy.h 中的核心枚举 |
| `ActionFlags` 枚举 | permission_policy.h 中的核心枚举 |

### 内部接口 (Internal API)

以下接口仅供内部使用，不建议外部依赖：

| 接口 | 稳定性依据 |
|------|----------|
| Service 层内部实现 (如 `DlpPermissionService` 类) | 仅通过 SDK 访问 |
| `libdlpparse_inner.so` | 内部解析库，bundle.json 中未标记为导出 |
| FUSE 内部实现 | 仅供沙箱使用 |
| 测试相关接口 | 仅用于测试 |

### 可替换点

| 位置 | 可替换内容 | 替换方式 |
|------|----------|----------|
| `libdlp_fuse.so` | FUSE 文件系统实现 | 实现相同的 `dlp_fuse_fd.h` 接口 |
| 加密适配器 | HUKS 调用 | 修改 `alg_adapt/` 目录下的适配器 |
| Bundle 适配器 | Bundle 信息获取 | 修改 `bundle_manager_adapter.cpp` |

---

## 线程与回调

### 回调机制

服务支持两类跨进程回调：

| 回调类型 | 接口 | 实现文件 |
|----------|------|----------|
| 沙箱变化回调 | `IDlpPermissionCallback` | callback/dlp_sandbox_change_callback/ |
| 打开文件回调 | `IDlpPermissionCallback` | callback/open_dlp_file_callback/ |

**回调模式**：
```
Client Process                Server Process
┌──────────────┐           ┌──────────────┐
│  Callback    │  回调     │  Service     │
│   Stub       │ ◄────────►│   Service    │
│              │  IPC      │              │
└──────────────┘           └──────────────┘
```

### 异步回调

支持异步 IPC 回调，用于耗时的证书生成/解析操作。

**代码证据**：
- `IDlpPermissionCallback` 接口 (IDlpPermissionService.idl:28)
- `DlpPermissionService::GenerateDlpCertificate()` 使用异步回调

---

## 错误传播机制

### 错误码定义

服务层错误码定义在 `dlp_permission_service.cpp` 或相关头文件中。

### 错误传播流程

```
Service 层
    ↓ 发生错误
    ↓ 返回错误码/抛出异常
    ↓ IPC 传递
Client 层 (SDK)
    ↓ 接收错误码
    ↓ 转换为业务异常
N-API 层
    ↓ 抛出 JS 异常
    ↓ napi_throw_error()
JavaScript 层
    ↓ 捕获异常
    ↓ 应用处理
```

---

## 相关跳转链接

- [架构说明](02_Architecture.md) - 深入理解模块间关系
- [对外 N-API](03_NAPI.md) - 查看 JS API 层接口
- [GN Targets](05_GN_Targets.md) - 查看构建配置

---

最后更新时间：2026-02-06
