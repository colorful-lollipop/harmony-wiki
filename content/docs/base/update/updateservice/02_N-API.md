# 02_N-API 参考

> JavaScript API 完整参考文档。

## 1. 模块导入

### 1.1 导入方式

```javascript
import client from 'libupdateclient.z.so'
```

### 1.2 获取 Updater 实例

```javascript
// 在线升级 (OTA)
let updater = client.getUpdater('OTA')

// 本地升级
let localUpdater = client.getLocalUpdater()

// 恢复出厂设置
let restorer = client.getRestorer()
```

**证据**: `frameworks/js/napi/update/src/update_module.cpp:353-357`

---

## 2. UpdateClient API

> 在线升级客户端，包含版本检查、下载、升级等核心功能。

### 2.1 类信息

| 属性 | 值 |
|------|-----|
| **类名** | UpdateClient |
| **构造函数** | `getUpdater(type: string)` |
| **命名空间** | 无 (默认导出) |

### 2.2 API 清单

#### 2.2.1 checkNewVersion

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `checkNewVersion()` |
| **C++ 实现** | `CheckNewVersion()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_CHECK_VERSION |

**参数**:
```typescript
interface CheckOptions {
    // 当前为空，保留扩展
}
```

**返回值**:
```typescript
Promise<CheckResult> | callback(error: BusinessError, result: CheckResult)

interface CheckResult {
    isExistNewVersion: boolean;           // 是否有新版本
    newVersionInfo: NewVersionInfo;        // 新版本信息
}
```

**示例**:
```javascript
// Promise 方式
try {
    const result = await updater.checkNewVersion()
    if (result.isExistNewVersion) {
        console.log('有新版本:', result.newVersionInfo)
    }
} catch (error) {
    console.error('检查失败:', error)
}

// Callback 方式
updater.checkNewVersion((error, result) => {
    if (error) {
        console.error('检查失败:', error)
        return
    }
    if (result.isExistNewVersion) {
        console.log('有新版本')
    }
})
```

**证据**: `update_module.cpp:153-158`, `session_type.h:0` (SESSION_CHECK_VERSION=0)

---

#### 2.2.2 download

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `download()` |
| **C++ 实现** | `Download()` |
| **同步/异步** | Promise / Callback (带进度事件) |
| **会话类型** | SESSION_DOWNLOAD |

**参数**:
```typescript
interface DownloadOptions {
    allowNetwork: NetType;      // 允许的网络类型
    order: Order;               // 下载顺序
}
```

**返回值**:
```typescript
Promise<void> | callback(error: BusinessError)
```

**事件**: `downloadProgress`

```typescript
interface DownloadProgress {
    downloadProgress: number;    // 0-100
    curBytes: number;            // 当前字节数
    totalBytes: number;         // 总字节数
}
```

**示例**:
```javascript
// 监听下载进度
updater.on('downloadProgress', (progress) => {
    console.log(`下载进度: ${progress.downloadProgress}%`)
    console.log(`${progress.curBytes}/${progress.totalBytes} bytes`)
})

// 开始下载
updater.download({
    allowNetwork: 'WIFI',                    // 仅 WiFi
    order: 'DOWNLOAD_AND_INSTALL'            // 下载并安装
})
```

**证据**: `update_module.cpp:174-179`, `session_type.h:1` (SESSION_DOWNLOAD=1)

---

#### 2.2.3 upgrade

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `upgrade()` |
| **C++ 实现** | `Upgrade()` |
| **同步/异步** | Promise / Callback (带进度事件) |
| **会话类型** | SESSION_UPGRADE |

**参数**:
```typescript
interface UpgradeOptions {
    order: Order;           // 升级顺序
}
```

**返回值**:
```typescript
Promise<void> | callback(error: BusinessError)
```

**事件**: `upgradeProgress`

```typescript
interface UpgradeProgress {
    upgradeProgress: number;    // 0-100
    subStatus: SubStatus;       // 子状态
}
```

**示例**:
```javascript
// 监听升级进度
updater.on('upgradeProgress', (progress) => {
    console.log(`升级进度: ${progress.upgradeProgress}%`)
})

// 触发升级
updater.upgrade({
    order: 'INSTALL_AND_APPLY'      // 安装并激活
})
```

**证据**: `update_module.cpp:204-210`, `session_type.h:4` (SESSION_UPGRADE=4)

---

#### 2.2.4 getNewVersionInfo

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `getNewVersionInfo()` |
| **C++ 实现** | `GetNewVersionInfo()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_GET_NEW_VERSION |

**返回值**:
```typescript
Promise<NewVersionInfo> | callback(error: BusinessError, result: NewVersionInfo)

interface NewVersionInfo {
    versionComponents: VersionComponent[];    // 版本组件列表
}
```

---

#### 2.2.5 getCurrentVersionInfo

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `getCurrentVersionInfo()` |
| **C++ 实现** | `GetCurrentVersionInfo()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_GET_CUR_VERSION |

**返回值**:
```typescript
Promise<CurrentVersionInfo> | callback(error: BusinessError, result: CurrentVersionInfo)
```

---

#### 2.2.6 setUpgradePolicy

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `setUpgradePolicy()` |
| **C++ 实现** | `SetUpgradePolicy()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_SET_POLICY |

**参数**:
```typescript
interface UpgradePolicy {
    downloadStrategy: number;       // 下载策略
    upgradeStrategy: number;        // 升级策略
    abnormalStrategy: string;       // 异常策略
}
```

---

#### 2.2.7 getUpgradePolicy

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `getUpgradePolicy()` |
| **C++ 实现** | `GetUpgradePolicy()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_GET_POLICY |

**返回值**:
```typescript
Promise<UpgradePolicy> | callback(error: BusinessError, result: UpgradePolicy)
```

---

#### 2.2.8 cancel

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `cancel()` |
| **C++ 实现** | `CancelUpgrade()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_CANCEL_UPGRADE |

**参数**:
```typescript
interface CancelOptions {
    service: number;    // 服务类型
}
```

---

#### 2.2.9 其他方法

| 方法名 | C++ 实现 | 会话类型 |
|--------|----------|----------|
| `pauseDownload` | `PauseDownload()` | SESSION_PAUSE_DOWNLOAD |
| `resumeDownload` | `ResumeDownload()` | SESSION_RESUME_DOWNLOAD |
| `clearError` | `ClearError()` | SESSION_CLEAR_ERROR |
| `terminateUpgrade` | `TerminateUpgrade()` | SESSION_TERMINATE_UPGRADE |
| `getTaskInfo` | `GetTaskInfo()` | SESSION_GET_TASK_INFO |
| `getNewVersionDescription` | `GetNewVersionDescription()` | SESSION_GET_NEW_VERSION_DESCRIPTION |
| `getCurrentVersionDescription` | `GetCurrentVersionDescription()` | SESSION_GET_CUR_VERSION_DESCRIPTION |

---

### 2.3 事件订阅

#### on(event, callback)

```typescript
updater.on(event: string, callback: Function)
```

**支持的事件**:
| 事件名 | 描述 | 回调参数 |
|--------|------|----------|
| `downloadProgress` | 下载进度 | `DownloadProgress` |
| `upgradeProgress` | 升级进度 | `UpgradeProgress` |

**示例**:
```javascript
// 订阅事件
updater.on('downloadProgress', (progress) => {
    console.log(`下载: ${progress.downloadProgress}%`)
})

updater.on('upgradeProgress', (progress) => {
    console.log(`升级: ${progress.upgradeProgress}%`)
})
```

**证据**: `update_module.cpp:332` (FUNCTION_ON)

#### off(event, callback)

```typescript
updater.off(event: string, callback?: Function)
```

**示例**:
```javascript
// 取消单个回调
updater.off('downloadProgress', specificCallback)

// 取消所有监听
updater.off('downloadProgress')
```

**证据**: `update_module.cpp:333` (FUNCTION_OFF)

---

## 3. Restorer API

> 恢复出厂设置相关功能。

### 3.1 类信息

| 属性 | 值 |
|------|-----|
| **类名** | Restorer |
| **获取方式** | `client.getRestorer()` |

### 3.2 API 清单

#### 3.2.1 factoryReset

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `factoryReset()` |
| **C++ 实现** | `Restorer::Napi::FactoryReset()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_FACTORY_RESET |
| **所需权限** | `ohos.permission.FACTORY_RESET` |

**返回值**:
```typescript
Promise<void> | callback(error: BusinessError)
```

**示例**:
```javascript
const restorer = client.getRestorer()

restorer.factoryReset((error) => {
    if (error) {
        console.error('恢复出厂设置失败:', error)
        return
    }
    console.log('恢复出厂设置完成')
})
```

**证据**: `update_module.cpp:282` (FUNCTION_FACTORY_RESET)

#### 3.2.2 forceFactoryReset

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `forceFactoryReset()` |
| **C++ 实现** | `Restorer::Napi::ForceFactoryReset()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_FORCE_FACTORY_RESET |
| **所需权限** | `ohos.permission.FORCE_FACTORY_RESET` |

**返回值**:
```typescript
Promise<void> | callback(error: BusinessError)
```

**证据**: `update_module.cpp:283` (FUNCTION_FORCE_FACTORY_RESET)

---

## 4. LocalUpdater API

> 本地升级相关功能，用于验证和应用本地升级包。

### 4.1 类信息

| 属性 | 值 |
|------|-----|
| **类名** | LocalUpdater |
| **获取方式** | `client.getLocalUpdater()` |

### 4.2 API 清单

#### 4.2.1 verifyUpgradePackage

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `verifyUpgradePackage()` |
| **C++ 实现** | `LocalUpdater::Napi::NapiVerifyUpgradePackage()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_VERIFY_PACKAGE |

**参数**:
```typescript
interface VerifyOptions {
    packagePath: string;     // 升级包路径
    keyPath?: string;        // 签名密钥路径 (可选)
}
```

**返回值**:
```typescript
Promise<boolean> | callback(error: BusinessError, result: boolean)
```

**示例**:
```javascript
const localUpdater = client.getLocalUpdater()

const isValid = await localUpdater.verifyUpgradePackage({
    packagePath: '/data/ota_package/update.zip',
    keyPath: '/data/ota_package/signing_key.pem'
})
```

**证据**: `update_module.cpp:298-299`

#### 4.2.2 applyNewVersion

| 属性 | 值 |
|------|-----|
| **JS 方法名** | `applyNewVersion()` |
| **C++ 实现** | `LocalUpdater::Napi::NapiApplyNewVersion()` |
| **同步/异步** | Promise / Callback |
| **会话类型** | SESSION_APPLY_NEW_VERSION |

**参数**:
```typescript
interface ApplyOptions {
    miscFile: string;              // misc 分区路径
    packageNames: string[];       // 升级包名称列表
}
```

**返回值**:
```typescript
Promise<void> | callback(error: BusinessError)
```

**示例**:
```javascript
await localUpdater.applyNewVersion({
    miscFile: '/dev/block/misc',
    packageNames: ['update.zip']
})
```

**证据**: `update_module.cpp:300`

#### 4.2.3 on/off

与 UpdateClient 类似，支持事件订阅。

**证据**: `update_module.cpp:301-302`

---

## 5. 错误码

### 5.1 错误码列表

| 错误名 | 错误码 | 描述 |
|--------|--------|------|
| `SUCCESS` | 0 | 成功 |
| `APP_NOT_GRANTED` | 201 | 无权限 |
| `NOT_SYSTEM_APP` | 202 | 非系统应用 |
| `PARAM_ERR` | 401 | 参数错误 |
| `UN_SUPPORT` | 801 | 不支持 |
| `FAIL` | 100 | 通用失败 |

### 5.2 错误处理示例

```javascript
try {
    await updater.checkNewVersion()
} catch (error) {
    // error.code: 错误码
    // error.message: 错误信息
    
    switch (error.code) {
        case 201:
            console.error('无权限执行此操作')
            break
        case 202:
            console.error('仅系统应用可调用此 API')
            break
        case 401:
            console.error('参数错误')
            break
        case 801:
            console.error('不支持此操作')
            break
        default:
            console.error('未知错误:', error)
    }
}
```

**证据**: `define_property.cpp:67-81`, `napi_common_utils.cpp:333-340`

---

## 6. 枚举常量

### 6.1 UpgradeStatus

| 常量名 | 值 | 描述 |
|--------|-------|------|
| `INIT` | 0 | 初始状态 |
| `CHECKING_VERSION` | 1 | 检查版本中 |
| `DOWNLOADING` | 2 | 下载中 |
| `INSTALLING` | 3 | 安装中 |
| ... | ... | ... |

### 6.2 ComponentType

| 常量名 | 值 | 描述 |
|--------|-------|------|
| `OTA` | 0 | OTA 升级 |
| `PATCH` | 1 | 补丁升级 |
| `COTA` | 2 | 配置 OTA |
| `PARAM` | 3 | 参数升级 |

### 6.3 其他枚举

| 枚举 | 用途 |
|------|------|
| `NetType` | 网络类型 |
| `Order` | 操作顺序 |
| `EffectiveMode` | 生效模式 |
| `DescriptionFormat` | 描述格式 |

**证据**: `define_property.cpp:67-81`

---

## 7. 权限要求

| API | 所需权限 |
|-----|----------|
| `checkNewVersion` | 无 |
| `download` | 无 |
| `upgrade` | 无 |
| `setUpgradePolicy` | 无 |
| `getUpgradePolicy` | 无 |
| `factoryReset` | `ohos.permission.FACTORY_RESET` |
| `forceFactoryReset` | `ohos.permission.FORCE_FACTORY_RESET` |

**注意**: HAP 应用调用任何 API 都需要是系统应用。

**证据**: `update_service.cpp:594-631`

---

## 8. 完整示例

```javascript
import client from 'libupdateclient.z.so'

// 获取 Updater
const updater = client.getUpdater('OTA')

// 监听事件
updater.on('downloadProgress', (progress) => {
    console.log(`下载: ${progress.downloadProgress}%`)
})

updater.on('upgradeProgress', (progress) => {
    console.log(`升级: ${progress.upgradeProgress}%`)
})

// 检查新版本
async function checkUpdate() {
    try {
        const result = await updater.checkNewVersion()
        if (result.isExistNewVersion) {
            console.log('发现新版本!')
            // 下载升级包
            await updater.download({
                allowNetwork: 'WIFI',
                order: 'DOWNLOAD_AND_INSTALL'
            })
            // 触发升级
            await updater.upgrade({
                order: 'INSTALL_AND_APPLY'
            })
        } else {
            console.log('当前已是最新版本')
        }
    } catch (error) {
        console.error('升级过程出错:', error)
    }
}

checkUpdate()
```

---

## 9. 下一步

- **架构设计**: [01_Architecture.md](./01_Architecture.md)
- **Inner API**: [03_Inner_API.md](./03_Inner_API.md)
- **安全评审**: [05_Security.md](./05_Security.md)
