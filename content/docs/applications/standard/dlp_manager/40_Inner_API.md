# 内部接口 (Inner API)

## 目的

本文档描述DLP Manager内部模块间的接口定义、依赖关系和调用约定，帮助开发者理解模块协作方式。

## 适用范围

- 进行模块开发的开发人员
- 需要理解模块间依赖的架构师
- 进行代码评审的人员

---

## 接口概览

DLP Manager内部接口主要分为以下几类：

| 接口类型 | 说明 | 稳定性 |
|---------|------|--------|
| RPC接口 | 跨进程通信接口 | 稳定 |
| Manager接口 | 业务管理器接口 | 中等 |
| Handler接口 | 处理器接口 | 中等 |
| 工具类接口 | 通用工具接口 | 稳定 |

---

## 1. RPC接口

### 1.1 CredConnectService - 凭证服务连接

**文件位置**: `entry/src/main/ets/rpc/CredConnectService.ets:25`

**职责**: 管理与DLP Credential Service的RPC连接

#### 接口定义

```typescript
export default class CredConnectService {
  // 构造函数
  constructor(context: common.UIExtensionContext | common.ServiceExtensionContext);
  
  // 设置云手机号
  setCloudPhone(cloudPhone: string): void;
  
  // 连接服务Ability
  connectServiceShareAbility(code: number): void;
}
```

#### 命令码

| 命令码 | 常量 | 功能 |
|--------|------|------|
| 1 | COMMAND_SEARCH_USER_INFO | 搜索用户信息 |
| 2 | COMMAND_GET_ACCOUNT_INFO | 获取账号信息 |
| 3 | COMMAND_GET_DOMAIN_ACCOUNT_INFO | 获取域账号信息 |
| 4 | COMMAND_BATCH_REFRESH | 批量刷新 |

**定义位置**: `entry/src/main/ets/common/constant.ets:295-299`

#### 依赖关系

```
CredConnectService
    ├── CredCallbackStub (回调处理)
    ├── Constants (命令码定义)
    └── DLP Credential Service (外部服务)
```

### 1.2 DlpPermissionAbilityServiceStub - DLP权限服务Stub

**文件位置**: `entry/src/main/ets/rpc/DlpPermissionAbilityServiceStub.ets:25`

**职责**: 处理DLP权限服务的RPC请求

#### 接口定义

```typescript
export default class DlpPermissionAbilityServiceStub extends rpc.RemoteObject {
  constructor(des: string);
  
  // 设置缓存路径
  setPathDir(pathDir: string): void;
  
  // RPC请求处理
  onRemoteMessageRequest(
    code: number, 
    data: rpc.MessageSequence, 
    reply: rpc.MessageSequence,
    options: rpc.MessageOption
  ): boolean;
}
```

#### 身份验证

```typescript
private checkCallerIdentity(data: rpc.MessageSequence): boolean {
  try {
    let token = data.readInterfaceToken();
    if (token !== Constant.SA_INTERFACE_TOKEN) {
      HiLog.error(TAG, `Interface token is invalid`);
      return false;
    }
    return true;
  }
}
```

**接口令牌**: `OHOS.Security.DlpCredentialAbility`（`entry/src/main/ets/common/constant.ets:124`）

### 1.3 IDL定义 - DLP RPC服务

**文件位置**: `entry/src/main/ets/Ability/data/IIdlDlpRpcServiceTs/`

#### 接口结构

```
IIdlDlpRpcServiceTs/
├── i_id_dlpRpc_service.ets    # 接口定义
├── id_dlpRpc_service_proxy.ets # 客户端代理
└── id_dlpRpc_service_stub.ets  # 服务端Stub
```

#### 核心接口方法

| 方法 | 说明 | 代码位置 |
|------|------|----------|
| `openDlpFile()` | 打开DLP文件 | `id_dlpRpc_service_proxy.ets` |
| `closeDlpFile()` | 关闭DLP文件 | `id_dlpRpc_service_proxy.ets` |
| `sandBoxLinkFile()` | 沙箱链接文件 | `id_dlpRpc_service_proxy.ets` |
| `linkSet()` | 链接设置 | `id_dlpRpc_service_proxy.ets` |
| `fileOpenHistory()` | 文件打开历史 | `id_dlpRpc_service_proxy.ets` |

---

## 2. Manager接口

### 2.1 OpenDlpFileManager - DLP文件打开管理器

**文件位置**: `entry/src/main/ets/OpenDlpFile/manager/OpenDlpFileManager.ets:38`

**设计模式**: 单例（Singleton）

**职责**: 管理所有DLP文件打开的状态和内容

#### 接口定义

```typescript
export class OpenDlpFileManager {
  // 获取单例
  static getInstance(): OpenDlpFileManager;
  
  // 状态管理
  setStatus(uri: string, status: DecryptStatus): Promise<Result<void>>;
  getStatus(uri: string): Result<DecryptStatus>;
  deleteStatus(uri: string): Promise<Result<void>>;
  
  // 内容管理
  addDecryptContent(uri: string, content: DecryptContent): Promise<Result<void>>;
  getHasDecryptedContent(uri: string): Result<DecryptContent>;
  getHasDecryptedContentByUriAndTokenId(uri: string, callerTokenId: number): Result<DecryptContent>;
  getHasDecryptedContentByLinkFileName(linkFileName: string): Result<DecryptContent>;
  getHasDecryptedContentByTokenId(tokenId: number): Result<DecryptContent>;
  
  // 清理
  removeByBundleNameAndAppIndex(bundleName: string, sandboxAppIndex: number): Promise<Result<Set<DecryptContent>>>;
  removeAllByUriAndTokenId(uri: string, tokenId: number): Promise<Result<void>>;
}
```

#### 状态枚举

```typescript
export enum DecryptState {
  NOT_STARTED = 1,  // 未开始
  DECRYPTING = 2,   // 解密中
  DECRYPTED = 3,    // 已解密
  ENCRYPTING = 4,   // 加密中
}
```

#### 线程安全

使用 `AsyncLock` 保证线程安全：

```typescript
private lock: ArkTSUtils.locks.AsyncLock;

public async setStatus(uri: string, status: DecryptStatus): Promise<Result<void>> {
  await this.lock.lockAsync(() => {
    this.statusMap.set(uri, status);
  });
}
```

### 2.2 OpeningDialogManager - 打开对话框管理器

**文件位置**: `entry/src/main/ets/OpenDlpFile/manager/OpeningDialogManager.ets`

**设计模式**: 单例（Singleton）

#### 核心方法

| 方法 | 说明 |
|------|------|
| `getInstance()` | 获取单例 |
| `loadOpeningDialogByFileTypeAndSize()` | 根据文件类型和大小加载对话框 |
| `unLoadOpeningDialogNormal()` | 正常卸载对话框 |
| `unLoadOpeningDialogAbnormal()` | 异常卸载对话框 |
| `hideOpeningDialogByFailed()` | 失败时隐藏对话框 |
| `setIsChargeDecrypting()` | 设置解密中状态 |
| `getIsChargeDecrypting()` | 获取解密中状态 |

### 2.3 AccountManager - 账号管理器

**文件位置**: `entry/src/main/ets/manager/AccountManager.ets`

#### 核心方法

| 方法 | 说明 |
|------|------|
| `connectAbility()` | 连接账号服务Ability |
| `checkAccountInfo()` | 检查账号信息 |
| `getLocalAccountInfo()` | 获取本地账号信息 |

---

## 3. Handler接口

### 3.1 处理器工厂

#### FileParseFactory

**文件位置**: `entry/src/main/ets/OpenDlpFile/handler/FileParseHandler.ets`

```typescript
class FileParseFactory {
  static createFileParse(openDlpFileData: OpenDlpFileData): Promise<Result<FileParseHandler>>;
}
```

#### AccountHandlerFactory

**文件位置**: `entry/src/main/ets/OpenDlpFile/handler/AccountHandler.ets`

```typescript
class AccountHandlerFactory {
  static createAccountHandler(decryptContent: DecryptContent): Promise<Result<AccountHandler>>;
}
```

### 3.2 FileParseHandler - 文件解析处理器

**职责**: 解析DLP文件头，提取元信息

#### 接口定义

```typescript
interface FileParseHandler {
  // 文件大小
  fileSize: number;
  
  // 解析类型
  parseType: ParseType;
  
  // 解析文件
  parse(uri: string, filesDir: string): Promise<Result<FileMetaInfo>>;
}
```

### 3.3 DecryptHandler - 解密处理器

**文件位置**: `entry/src/main/ets/OpenDlpFile/handler/DecryptHandler.ets`

#### 核心方法

```typescript
class DecryptHandler {
  // 获取解密数据
  getDecryptData(
    decryptContent: DecryptContent, 
    context: common.ServiceExtensionContext
  ): Promise<Result<DecryptContent>>;
  
  // 安装沙箱
  installSandbox(decryptContent: DecryptContent): Promise<Result<void>>;
  
  // 授予URI权限
  grantUriPermission(decryptContent: DecryptContent): Promise<Result<void>>;
}
```

### 3.4 StartSandboxHandler - 沙箱启动处理器

**文件位置**: `entry/src/main/ets/OpenDlpFile/handler/StartSandboxHandler.ets`

**设计模式**: 单例

#### 核心方法

```typescript
class StartSandboxHandler {
  static getInstance(): StartSandboxHandler;
  
  // 启动沙箱
  startSandbox(decryptContent: DecryptContent): Promise<Result<void>>;
  
  // 启动Ability
  startAbility(decryptContent: DecryptContent): Promise<Result<void>>;
}
```

---

## 4. 工具类接口

### 4.1 FileUtils - 文件工具

**文件位置**: `entry/src/main/ets/common/FileUtils/utils.ets`

#### 核心函数

| 函数 | 签名 | 说明 |
|------|------|------|
| `getFileFd()` | `getFileFd(uri: string, mode?: number): Result<number>` | 获取文件描述符（`entry/src/main/ets/common/FileUtils/utils.ets:136`） |
| `getOsAccountInfo()` | `getOsAccountInfo(): Promise<OsAccountInfo>` | 获取OS账号信息（`entry/src/main/ets/common/FileUtils/utils.ets:148`） |
| `getAuthPerm()` | `getAuthPerm(accountName: string, dlpProperty: DLPProperty): DLPFileAccess` | 获取权限级别（`entry/src/main/ets/common/FileUtils/utils.ets:173`） |
| `judgeIsSandBox()` | `judgeIsSandBox(want: Want): Promise<boolean>` | 判断是否来自沙箱（`entry/src/main/ets/common/FileUtils/utils.ets:212`） |
| `getAppId()` | `getAppId(bundleName: string): Promise<string>` | 获取应用ID（`entry/src/main/ets/common/FileUtils/utils.ets:296`） |
| `isValidPath()` | `isValidPath(path: string): boolean` | 验证路径有效性（`entry/src/main/ets/common/FileUtils/utils.ets:507`） |

### 4.2 HuksCipherUtils - 加密工具

**文件位置**: `entry/src/main/ets/common/huks/HuksCipherUtil.ets:22`

#### 核心方法

| 方法 | 说明 |
|------|------|
| `isKeyExist()` | 检查密钥是否存在 |
| `generateKey()` | 生成密钥 |
| `encrypt()` | 加密数据 |
| `decrypt()` | 解密数据 |

### 4.3 HiLog - 日志工具

**文件位置**: `entry/src/main/ets/common/HiLog.ets`

#### 日志级别

| 方法 | 说明 |
|------|------|
| `info()` | 信息日志 |
| `debug()` | 调试日志 |
| `warn()` | 警告日志 |
| `error()` | 错误日志 |
| `wrapError()` | 包装错误日志 |

---

## 5. 数据类定义

### 5.1 DLPProperty - DLP属性

**文件位置**: `entry/src/main/ets/common/dlpClass.ets:69`

```typescript
export default class IDLDLPProperty extends rpc.MessageSequence {
  public ownerAccount: string;           // 所有者账号
  public ownerAccountID: string;         // 所有者账号ID
  public ownerAccountType: number;       // 所有者账号类型
  public authUserList: IAuthUser[];      // 授权用户列表
  public contactAccount: string;         // 联系人账号
  public offlineAccess: boolean;         // 是否支持离线访问
  public everyoneAccessList: number[];   // 所有人权限列表
  public expireTime: number;             // 过期时间
}
```

### 5.2 IAuthUser - 授权用户

**文件位置**: `entry/src/main/ets/common/dlpClass.ets:23`

```typescript
export class IAuthUser extends rpc.MessageSequence {
  public authAccount: string;       // 授权账号
  public authAccountType: number;   // 授权账号类型
  public dlpFileAccess: number;     // DLP文件访问权限
  public permExpiryTime: number;    // 权限过期时间
}
```

---

## 6. 模块依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                        OpenDlpFileProcessor                      │
│                    (entry/OpenDlpFile/ViewProcessor)             │
└──────────┬──────────────────────────────────────────────┬────────┘
           │                                              │
           ▼                                              ▼
┌────────────────────────┐                    ┌────────────────────────┐
│   OpenDlpFileManager   │                    │   OpeningDialogManager │
│       (单例)            │                    │        (单例)          │
└──────────┬─────────────┘                    └──────────┬─────────────┘
           │                                              │
           ▼                                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                           Handler层                              │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐             │
│  │ FileParse    │ │   Account    │ │   Decrypt    │             │
│  │   Handler    │ │   Handler    │ │   Handler    │             │
│  └──────────────┘ └──────────────┘ └──────────────┘             │
└─────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────┐
│                           RPC层                                  │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │ CredConnectService│  │  IdDlpRpcService  │                    │
│  │                  │  │     Proxy         │                    │
│  └──────────────────┘  └──────────────────┘                     │
└─────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────┐
│                       系统服务层                                  │
│     DLP Credential Service    |    DLP Permission Service        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. 接口稳定性说明

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| RPC接口（Stub/Proxy） | 稳定 | 对外服务接口，变更需兼容 |
| Manager单例接口 | 中等 | 内部使用，变更需内部协调 |
| Handler接口 | 中等 | 内部使用，随业务调整 |
| 工具类接口 | 稳定 | 通用工具，保持向后兼容 |
| 常量定义 | 稳定 | 错误码等不随意变更 |

---

## 关键结论

1. **单例模式**: 核心管理器（OpenDlpFileManager、OpeningDialogManager）使用单例模式
2. **工厂模式**: Handler创建使用工厂模式，支持类型扩展
3. **RPC分层**: RPC接口分层清晰（Service→Proxy/Stub→IDL）
4. **状态管理**: 统一使用OpenDlpFileManager管理DLP文件状态
5. **线程安全**: 状态管理使用AsyncLock保证线程安全

---

## 相关链接

- [架构设计](10_Architecture.md) - 了解架构设计
- [目录结构](20_Directory_Structure.md) - 查看文件组织
- [调用链附录](appendix/Callgraphs.md) - 查看调用流程
