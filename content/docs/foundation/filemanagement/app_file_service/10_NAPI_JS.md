# JS N-API 接口

## 6.1 接口概述

应用文件服务提供三个 JS N-API 模块：fileshare（文件分享）、fileuri（URI 管理）、backup（备份恢复）。这些模块通过标准 N-API 接口注册，为 JS 应用提供编程能力。

### 6.1.1 模块列表

| 模块 | 命名空间 | 注册位置 |
|------|----------|----------|
| fileshare | `@ohos.file.fileshare` | `interfaces/kits/js/file_share/fileshare_n_exporter.cpp:226` |
| fileuri | `@ohos.file.fileuri` | `interfaces/kits/js/file_uri/module.cpp:49` |
| backup | `@ohos.file.backup` | `interfaces/kits/js/backup/module.cpp:49` |

## 6.2 FileShare 模块

### 6.2.1 模块注册

**文件**：`interfaces/kits/js/file_share/fileshare_n_exporter.cpp`

```cpp
NAPI_MODULE(fileshare, FileShareExport)
```

### 6.2.2 导出函数

| 函数名 | 异步模式 | 功能描述 |
|--------|----------|----------|
| `grantUriPermission` | Promise | 授予 URI 权限 |
| `persistPermission` | Promise | 持久化权限 |
| `revokePermission` | Promise | 撤销权限 |
| `activatePermission` | Promise | 激活权限 |
| `deactivatePermission` | Promise | 停用权限 |
| `checkPersistentPermission` | Promise | 检查持久化权限 |
| `checkPathPermission` | Promise | 检查路径权限 |

### 6.2.3 导出枚举

**OperationMode 操作模式**：

```javascript
enum OperationMode {
    READ_MODE = 1 << 0,       // 读模式
    WRITE_MODE = 1 << 1,      // 写模式
    CREATE_MODE = 1 << 2,     // 创建模式
    DELETE_MODE = 1 << 3,     // 删除模式
    RENAME_MODE = 1 << 4,     // 重命名模式
}
```

### 6.2.4 主要 API 详解

**grantUriPermission 授予 URI 权限**：

```typescript
function grantUriPermission(
    uri: string,
    bundleName: string,
    mode: OperationMode
): Promise<void>
```

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `uri` | string | 是 | 目标文件 URI |
| `bundleName` | string | 是 | 被授权应用包名 |
| `mode` | OperationMode | 是 | 授权模式 |

**persistPermission 持久化权限**：

```typescript
function persistPermission(
    uri: string,
    bundleName: string,
    mode: OperationMode
): Promise<void>
```

### 6.2.5 使用示例

```javascript
import fileShare from '@ohos.file.fileshare';

async function shareFile() {
    try {
        // 授予临时读权限
        await fileShare.grantUriPermission(
            'file:///data/storage/el2/base/files/document.txt',
            'com.example.receiver',
            fileShare.OperationMode.READ_MODE
        );
        console.log('权限授予成功');
    } catch (error) {
        console.error('权限授予失败:', error);
    }
}
```

## 6.3 FileURI 模块

### 6.3.1 模块注册

**文件**：`interfaces/kits/js/file_uri/module.cpp`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Export,
    .nm_modname = "file.fileuri",
    .nm_priv = ((void *)0),
    .reserved = {0}
};
```

### 6.3.2 FileUri 类

**构造方法**：

```typescript
constructor(uri: string)
```

**实例方法**：

| 方法名 | 返回类型 | 功能描述 |
|--------|----------|----------|
| `toString()` | string | 获取 URI 字符串 |
| `name` | string | 获取文件名 |
| `path` | string | 获取路径 |
| `getFullDirectoryUri()` | string | 获取目录 URI |
| `isRemoteUri()` | boolean | 判断是否为远程 URI |
| `normalize()` | boolean | 标准化 URI |
| `scheme` | string | 获取协议部分 |
| `authority` | string | 获取权限部分 |
| `ssp` | string | 获取特殊权限部分 |
| `userInfo` | string | 获取用户信息 |
| `host` | string | 获取主机名 |
| `port` | number | 获取端口号 |
| `query` | string | 获取查询部分 |
| `fragment` | string | 获取片段部分 |

### 6.3.3 静态函数

| 函数名 | 返回类型 | 功能描述 |
|--------|----------|----------|
| `getUriFromPath(path: string)` | string | 从路径获取 URI |

### 6.3.4 使用示例

```javascript
import fileUri from '@ohos.file.fileuri';

async function uriExample() {
    // 从路径创建 URI
    const uri = fileUri.getUriFromPath(
        '/data/storage/el2/base/files/document.txt'
    );
    console.log('URI:', uri); // file:///data/storage/el2/base/files/document.txt
    
    // 创建 FileUri 对象
    const uriObj = new fileUri.Uri(uri);
    
    // 获取文件名
    console.log('文件名:', uriObj.name); // document.txt
    
    // 判断是否为远程 URI
    console.log('是否远程:', uriObj.isRemoteUri()); // false
    
    // 标准化 URI
    const normalized = uriObj.normalize();
    console.log('标准化结果:', normalized);
}
```

## 6.4 Backup 模块

### 6.4.1 模块注册

**文件**：`interfaces/kits/js/backup/module.cpp`

```cpp
static napi_module _module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = Export,
    .nm_modname = "file.backup",
    .nm_priv = ((void *)0),
    .reserved = {0}
};
```

### 6.4.2 静态函数

| 函数名 | 返回类型 | 功能描述 |
|--------|----------|----------|
| `getLocalCapabilities()` | Capabilities | 获取本地备份能力 |
| `getBackupInfo(bundleName: string)` | BackupInfo | 获取应用备份信息 |
| `updateTimer(bundleName: string, timeout: number)` | boolean | 更新超时定时器 |
| `updateSendRate(bundleName: string, sendRate: number)` | boolean | 更新发送速率 |
| `getBackupVersion()` | string | 获取备份版本 |

### 6.4.3 SessionBackup 类

**构造方法**：

```typescript
constructor()
```

**实例方法**：

| 方法名 | 返回类型 | 功能描述 |
|--------|----------|----------|
| `onFileReady(cb: FileReadyCallback)` | void | 设置文件就绪回调 |
| `onBundleStarted(cb: BundleStartedCallback)` | void | 设置包开始回调 |
| `onBundleFinished(cb: BundleFinishedCallback)` | void | 设置包完成回调 |
| `appendBundles(bundleNames: string[])` | Promise<void> | 追加待备份应用 |
| `finish()` | Promise<void> | 完成追加 |
| `release()` | Promise<void> | 释放会话 |
| `cancel()` | Promise<void> | 取消备份 |

### 6.4.4 SessionRestore 类

**构造方法**：

```typescript
constructor(file: fileUri)
```

**实例方法**：

| 方法名 | 返回类型 | 功能描述 |
|--------|----------|----------|
| `onFileReady(cb: FileReadyCallback)` | void | 设置文件就绪回调 |
| `onBundleStarted(cb: BundleStartedCallback)` | void | 设置包开始回调 |
| `onBundleFinished(cb: BundleFinishedCallback)` | void | 设置包完成回调 |
| `appendBundles(bundleNames: string[])` | Promise<void> | 追加待恢复应用 |
| `finish()` | Promise<void> | 完成追加 |
| `release()` | Promise<void> | 释放会话 |
| `restore()` | Promise<void> | 开始恢复 |

### 6.4.5 使用示例

```javascript
import backup from '@ohos.file.backup';
import fileUri from '@ohos.file.fileuri';

async function backupExample() {
    try {
        // 获取本地备份能力
        const capabilities = await backup.getLocalCapabilities();
        console.log('支持的备份类型:', capabilities);
        
        // 创建备份会话
        const session = new backup.SessionBackup();
        
        // 设置回调
        session.onFileReady((err, data) => {
            if (err) {
                console.error('文件就绪错误:', err);
            } else {
                console.log('文件已就绪:', data);
            }
        });
        
        session.onBundleStarted((bundleName) => {
            console.log('开始备份应用:', bundleName);
        });
        
        session.onBundleFinished((bundleName, result) => {
            console.log('应用备份完成:', bundleName, result);
        });
        
        // 追加待备份应用
        await session.appendBundles(['com.example.app']);
        
        // 完成追加
        await session.finish();
        
        // 释放会话
        await session.release();
        
        console.log('备份完成');
    } catch (error) {
        console.error('备份失败:', error);
    }
}

async function restoreExample() {
    try {
        // 创建恢复会话
        const restoreSession = new backup.SessionRestore(
            new fileUri.Uri('file:///path/to/backup.tar')
        );
        
        // 设置回调
        restoreSession.onFileReady((err, data) => {
            console.log('恢复文件:', data);
        });
        
        // 追加待恢复应用
        await restoreSession.appendBundles(['com.example.app']);
        
        // 开始恢复
        await restoreSession.restore();
        
        // 释放会话
        await restoreSession.release();
        
        console.log('恢复完成');
    } catch (error) {
        console.error('恢复失败:', error);
    }
}
```

## 6.5 错误码定义

### 6.5.1 通用错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 0 | SUCCESS | 成功 |
| 401 | ERR_INVALID_PARAMETER | 参数错误 |
| 13900001 | EPERM | 操作不允许 |
| 13900002 | ENOENT | 文件不存在 |
| 13900003 | ESRCH | 进程不存在 |
| 13900004 | EINTR | 系统调用中断 |
| 13900005 | EIO | I/O 错误 |
| 13900006 | ENXIO | 设备不存在 |
| 13900007 | E2BIG | 参数列表过长 |
| 13900008 | ENOEXEC | 格式错误 |
| 13900009 | EBADF | 文件描述符错误 |
| 13900010 | ECHILD | 子进程不存在 |
| 13900011 | EAGAIN | 重试 |
| 13900012 | ENOMEM | 内存不足 |
| 13900013 | EACCES | 权限不足 |
| 13900014 | EFAULT | 地址错误 |
| 13900015 | ENOTBLK | 块设备要求 |
| 13900016 | EBUSY | 设备忙 |
| 13900017 | EEXIST | 文件已存在 |
| 13900018 | EXDEV | 跨设备链接 |
| 13900019 | ENODEV | 设备不存在 |
| 13900020 | ENOTDIR | 要求目录 |
| 13900021 | EISDIR | 是目录 |
| 13900022 | EINVAL | 参数无效 |
| 13900023 | ENFILE | 文件表溢出 |
| 13900024 | EMFILE | 打开文件过多 |
| 13900025 | ENOTTY | 非终端设备 |
| 13900026 | ETXTBSY | 文本文件忙 |
| 13900027 | EFBIG | 文件过大 |
| 13900028 | ENOSPC | 设备无空间 |
| 13900029 | ESPIPE | 定位错误 |
| 13900030 | EROFS | 只读文件系统 |
| 13900031 | EMLINK | 链接过多 |
| 13900032 | EPIPE | 管道断开 |
| 13900033 | EDOM | 数学参数超出范围 |
| 13900034 | ERANGE | 结果超出范围 |
| 13900035 | EDEADLK | 死锁 |
| 13900036 | ENAMETOOLONG | 文件名过长 |
| 13900040 | EL2NSYNC | 非同步层 |
| 13900041 | ECANCELED | 操作取消 |

### 6.5.2 权限错误码

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 14300001 | PERMISSION_DENIED | 权限被拒绝 |
| 14300002 | PERSISTENCE_FORBIDDEN | 不允许持久化 |
| 14300003 | INVALID_MODE | 无效模式 |
| 14300004 | INVALID_PATH | 无效路径 |
| 14300005 | PERMISSION_NOT_PERSISTED | 权限未持久化 |

## 6.6 相关文档

| 文档 | 说明 |
|------|------|
| [NDK 接口](11_NAPI_NDK.md) | Native 开发接口 |
| [内部 API](20_Inner_API.md) | 内部编程接口 |
| [系统架构](01_Architecture.md) | N-API 架构位置 |
| [项目概览](00_Overview.md) | 接口层概述 |
| [安全评审](05_Security_Review.md) | API 安全风险 |
