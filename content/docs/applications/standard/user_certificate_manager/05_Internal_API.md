# 内部 API 清单

> 本文档描述用户证书管理部件的内部模块接口和依赖关系

---

## 目的

本文档帮助开发者理解内部模块间的接口、依赖关系和稳定性约定。

## 适用范围

- 需要修改内部代码的开发者
- 架构师
- 需要理解模块依赖的技术人员

## 关键结论

1. **分层架构**: Model-Presenter-Page 三层分离，单向依赖
2. **模块职责**: Model 层负责业务逻辑，Presenter 层负责页面逻辑，Page 层负责 UI
3. **全局状态**: 通过 GlobalContext 单例管理全局状态
4. **无循环依赖**: 模块间依赖清晰，无循环依赖
5. **稳定接口**: Model 层接口相对稳定，Presenter 层和 Page 层可能变化

## 相关跳转

- [02_Directory_Structure.md](wiki/02_Directory_Structure.md) - 目录结构
- [03_Architecture.md](wiki/03_Architecture.md) - 架构设计
- [04_External_API.md](wiki/04_External_API.md) - 外部 API

---

## 1. 内部架构分层

### 1.1 分层模型

```
┌─────────────────────────────────────────────────────────────┐
│                    Pages 层 (UI)                        │
│  - 组件化 UI                                          │
│  - 事件处理                                            │
│  - 数据展示                                            │
├─────────────────────────────────────────────────────────────┤
│                  Presenter 层 (页面逻辑)                   │
│  - 业务编排                                            │
│  - 数据转换                                            │
│  - 调用 Model 层接口                                  │
├─────────────────────────────────────────────────────────────┤
│                    Model 层 (业务逻辑)                   │
│  - 核心业务逻辑                                        │
│  - 调用外部 API                                       │
│  - 数据封装                                            │
├─────────────────────────────────────────────────────────────┤
│                  Common 层 (公共组件)                     │
│  - 全局上下文                                          │
│  - 公共组件                                            │
│  - 工具函数                                            │
└─────────────────────────────────────────────────────────────┘
```

**依赖方向**: Pages → Presenter → Model → Common

**证据位置**: `certmanager/src/main/ets/` 目录结构

---

## 2. Model 层接口

### 2.1 CertMangerModel - 证书管理核心

#### 2.1.1 公开接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getCertOrCredList(optType, callback)` | `CMModelOptType, Function` | `void` | 获取证书/凭据列表 |
| `getCertOrCred(optType, uri, callback)` | `CMModelOptType, string, Function` | `void` | 获取证书/凭据详情 |
| `deleteCertOrCred(optType, uri, callback)` | `CMModelOptType, string, Function` | `void` | 删除证书/凭据 |
| `setCertStatus(optType, uri, status, callback)` | `CMModelOptType, string, boolean, Function` | `void` | 设置证书状态 |
| `delAllCertOrCred(optType, callback)` | `CMModelOptType, Function` | `void` | 删除所有证书/凭据 |
| `getAuthAppList(optType, uri, callback)` | `CMModelOptType, string, Function` | `void` | 获取授权应用列表 |
| `setAppAuth(optType, uri, appUid, status, callback)` | `CMModelOptType, string, string, boolean, Function` | `void` | 设置应用授权 |
| `installCertOrCred(optType, alias, data, pwd, callback)` | `CMModelOptType, string, Uint8Array, string, Function` | `void` | 安装证书/凭据 |

#### 2.1.2 扩展接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `setAppAuthPromise(optType, uri, appUid, status)` | `CMModelOptType, string, string, boolean` | `Promise<void>` | 设置应用授权 (Promise) |
| `getAllAppPrivateCertificates(appUid)` | `number` | `Promise<CredentialAbstractVo[]>` | 获取应用私钥凭据 |
| `getUkeyCertificateList(certPurpose)` | `CertificatePurpose` | `Promise<CredentialAbstractVo[]>` | 获取 UKey 证书列表 |

**稳定性**: ⭐⭐⭐ (稳定)
**调用方**: Presenter 层

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:89-834`

---

### 2.2 CheckUserAuthModel - 用户认证

#### 2.2.1 公开接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `isAuthTypeSupported(authType)` | `UserAuthType` | `boolean` | 检查认证类型是否支持 |
| `auth(titleStr, callback)` | `string, Function` | `void` | 执行用户认证 |

**稳定性**: ⭐⭐⭐ (稳定)
**调用方**: Presenter 层

**证据位置**: `certmanager/src/main/ets/model/CheckUserAuthModel.ets:31-117`

---

### 2.3 BundleModel - 包信息获取

#### 2.3.1 公开接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getAppInfoList(appUid, callback)` | `number, Function` | `Promise<void>` | 获取应用信息 |

**返回数据**: `AppInfoVo { appImage: string, appName: string }`

**稳定性**: ⭐⭐ (较稳定)
**调用方**: Presenter 层

**证据位置**: `certmanager/src/main/ets/model/BundleModel.ets:24-49`

---

### 2.4 FileIoModel - 文件 I/O

#### 2.4.1 公开接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getMediaFileData(mediaUri, callback)` | `string, Function` | `void` | 读取文件内容 |
| `getMediaFileSuffix(mediaUri, callback)` | `string, Function` | `void` | 获取文件后缀 |

**返回数据**:
- `getMediaFileData`: `Uint8Array`
- `getMediaFileSuffix`: `string` (小写后缀)

**稳定性**: ⭐⭐⭐ (稳定)
**调用方**: Presenter 层

**证据位置**: `certmanager/src/main/ets/model/FileIoModel.ets:20-64`

---

### 2.5 PreventScreenshotsModel - 防截屏

#### 2.5.1 公开接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `PreventScreenshots(flag, session)` | `boolean, UIExtensionContentSession \| undefined` | `void` | 设置/取消防截屏 |

**稳定性**: ⭐⭐⭐ (稳定)
**调用方**: Page 层

**证据位置**: `certmanager/src/main/ets/model/PreventScreenshotsModel.ets:24-57`

---

## 3. Presenter 层接口

### 3.1 CmFaPresenter - 主页面逻辑

#### 3.1.1 公开接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getInstance()` | - | `CmFaPresenter` | 获取单例 |
| `onAboutToAppear()` | - | `void` | 页面即将显示 |
| `aboutToDisappear()` | - | `void` | 页面即将消失 |
| `routeToNextInstallCert(fileUri)` | `string` | `void` | 路由到证书安装页面 |
| `routeToNextInstallEvidence(fileUri)` | `string` | `void` | 路由到凭据安装页面 |
| `startInstallCert(context)` | `Context` | `void` | 启动证书安装 |

**稳定性**: ⭐⭐ (可能变化)
**调用方**: 主页面 UI

**证据位置**: `certmanager/src/main/ets/presenter/CmFaPresenter.ets:30-100`

---

### 3.2 CmInstallPresenter - 安装页面逻辑

#### 3.2.1 公开接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getInstance()` | - | `CmInstallPresenter` | 获取单例 |
| `installCert(fileUri, alias, suffix, isCa)` | `string, string, string, boolean` | `void` | 安装证书 |
| `installEvidence(fileUri, alias, pwd, suffix)` | `string, string, string, string` | `void` | 安装凭据 |

**稳定性**: ⭐⭐ (可能变化)
**调用方**: 安装页面 UI

**证据位置**: `certmanager/src/main/ets/presenter/CmInstallPresenter.ets`

---

### 3.3 其他 Presenter

| Presenter | 主要方法 | 说明 |
|-----------|----------|------|
| `CmShowUserCaPresenter` | `onAboutToAppear()`, `loadUserCaList()` | 用户 CA 列表管理 |
| `CmShowSysCaPresenter` | `onAboutToAppear()`, `loadSysCaList()` | 系统 CA 列表管理 |
| `CmShowAppCredPresenter` | `onAboutToAppear()`, `loadAppCredList()` | 用户凭据列表管理 |
| `CmShowSysCredPresenter` | `onAboutToAppear()`, `loadSysCredList()` | 系统凭据列表管理 |
| `CmAppCredAuthPresenter` | `loadAuthorizedAppList()`, `setAuthorizedAppStatus()` | 授权应用管理 |

**稳定性**: ⭐⭐ (可能变化)
**调用方**: 对应页面 UI

**证据位置**: `certmanager/src/main/ets/presenter/` 目录

---

## 4. Common 层接口

### 4.1 GlobalContext - 全局上下文

#### 4.1.1 单例接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getContext()` | - | `GlobalContext` | 获取全局上下文实例 (单例) |

#### 4.1.2 Context 接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getCmContext()` | - | `UIAbilityContext` | 获取 Ability 上下文 |
| `setCmContext()` | `UIAbilityContext` | `void` | 设置 Ability 上下文 |

#### 4.1.3 Want 接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getAbilityWant()` | - | `Want` | 获取启动参数 |
| `setAbilityWant()` | `Want` | `void` | 设置启动参数 |
| `clearAbilityWantUri()` | - | `void` | 清除 Want URI |

#### 4.1.4 PwdStore 接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getPwdStore()` | - | `PwdStore` | 获取密码存储 |
| `setPwdStore()` | `PwdStore` | `void` | 设置密码存储 |

#### 4.1.5 Session 接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getSession()` | - | `UIExtensionContentSession` | 获取会话对象 |
| `setSession()` | `UIExtensionContentSession` | `void` | 设置会话对象 |
| `clearSession()` | - | `void` | 清除会话 |

#### 4.1.6 Flag 接口

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `getFlag()` | - | `Boolean` | 获取标记位 |
| `setFlag()` | `Boolean` | `void` | 设置标记位 |

#### 4.1.7 PwdStore - 密码存储

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `setCertPwd()` | `string` | `void` | 设置证书密码 |
| `getCertPwd()` | - | `string` | 获取证书密码 |
| `clearCertPwd()` | - | `void` | 清除证书密码 |

**稳定性**: ⭐⭐⭐ (稳定)
**调用方**: 所有层

**证据位置**: `certmanager/src/main/ets/common/GlobalContext.ts:36-105`

---

## 5. 模块依赖关系

### 5.1 依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                     Pages 层                         │
│  certManagerFa.ets, certInstallFromStorage.ets, etc.  │
└──────────────┬──────────────────────────────────────────┘
               ↓
┌─────────────────────────────────────────────────────────────┐
│                  Presenter 层                         │
│  CmFaPresenter, CmInstallPresenter, etc.              │
└──────────────┬──────────────────────────────────────────┘
               ↓
┌─────────────────────────────────────────────────────────────┐
│                    Model 层                           │
│  CertMangerModel ⭐, CheckUserAuthModel, etc.           │
└──────────────┬──────────────────────────────────────────┘
               ↓
┌─────────────────────────────────────────────────────────────┐
│                  Common 层                            │
│  GlobalContext ⭐, constants, components, utils          │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 依赖清单

| 调用方 | 被调用方 | 依赖类型 | 稳定性 |
|---------|-----------|---------|--------|
| Pages 层 | Presenter 层 | 方法调用 | ⭐⭐ (可能变化) |
| Pages 层 | Common 层 | 方法调用 | ⭐⭐⭐ (稳定) |
| Presenter 层 | Model 层 | 方法调用 | ⭐⭐⭐ (稳定) |
| Presenter 层 | Common 层 | 方法调用 | ⭐⭐⭐ (稳定) |
| Model 层 | Common 层 | 方法调用 | ⭐⭐⭐ (稳定) |
| Model 层 | 外部 API | API 调用 | ⭐⭐⭐⭐ (稳定) |
| Common 层 | 外部 API | API 调用 | ⭐⭐⭐⭐ (稳定) |

**证据位置**: `certmanager/src/main/ets/` import 语句

---

## 6. 接口稳定性约定

### 6.1 稳定性等级

| 等级 | 图标 | 说明 | 示例 |
|------|------|------|------|
| 稳定 | ⭐⭐⭐ | 接口基本不会变化 | Model 层公共方法 |
| 较稳定 | ⭐⭐ | 小改动可能，向后兼容 | Common 层工具函数 |
| 可能变化 | ⭐⭐ | 可能重构，注意版本 | Presenter 层方法 |
| 不稳定 | ⭐ | 频繁变化，不建议依赖 | Page 层内部实现 |

### 6.2 可替换点

| 模块 | 可替换性 | 说明 |
|------|---------|------|
| Model 层 | 低 | 核心业务逻辑，依赖外部 API |
| Presenter 层 | 中 | 页面逻辑，可能重构 |
| Page 层 | 高 | UI 界面，经常调整 |
| Common 层 | 低 | 公共组件，通用性强 |

---

## 7. 数据流向

### 7.1 典型数据流

#### 7.1.1 证书安装流程

```
用户操作 (UI)
  ↓
Pages 层 (certInstallFromStorage.ets)
  ↓ fileUri, alias, pwd
Presenter 层 (CmInstallPresenter.ets)
  ↓ 调用 installCert()
Model 层 (CertMangerModel.ets)
  ↓ installUserTrustedCertificate()
外部 API (@ohos.security.certManager)
  ↓ 返回结果
Model 层 → Presenter 层 → Pages 层
  ↓ 显示结果
```

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:697-730`

#### 7.1.2 证书列表加载流程

```
页面显示 (onAboutToAppear)
  ↓
Presenter 层 (CmShowUserCaPresenter.ets)
  ↓ loadUserCaList()
Model 层 (CertMangerModel.ets)
  ↓ getCertOrCredList()
Model 内部 → getUserTrustedCertificateList()
外部 API (@ohos.security.certManager)
  ↓ 返回列表
Model 层 → Presenter 层 → Pages 层
  ↓ 更新 UI
```

**证据位置**: `certmanager/src/main/ets/model/CertMangerModel.ets:423-453`

---

## 8. 单例模式

### 8.1 使用单例的模块

| 模块 | 实现方式 | 用途 |
|------|----------|------|
| `CertMangerModel` | `let certMangerModel = new CertMangerModel(); export default certMangerModel;` | 全局唯一实例 |
| `CheckUserAuthModel` | `let checkUserAuthModel = new CheckUserAuthModel(); export default checkUserAuthModel;` | 全局唯一实例 |
| `BundleModel` | `let bundleNameModel = new BundleNameModel(); export default bundleNameModel;` | 全局唯一实例 |
| `FileIoModel` | `let fileIoModel = new FileIoModel(); export default fileIoModel;` | 全局唯一实例 |
| `PreventScreenshotsModel` | 单例模式 `getInstance()` | 全局唯一实例 |
| `GlobalContext` | 单例模式 `getInstance()` | 全局唯一实例 |
| `CmFaPresenter` | 单例模式 `getInstance()` | 全局唯一实例 |
| `CmInstallPresenter` | 单例模式 `getInstance()` | 全局唯一实例 |

**证据位置**: `certmanager/src/main/ets/model/*.ets`, `certmanager/src/main/ets/common/GlobalContext.ts`, `certmanager/src/main/ets/presenter/*.ets`

---

**END OF 05_Internal_API.md**
