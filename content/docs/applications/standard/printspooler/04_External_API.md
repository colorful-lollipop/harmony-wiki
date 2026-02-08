# 对外 API

## 目的

本文档列出 PrintSpooler 使用的所有 OpenHarmony 系统 JS API，包括 API 清单、参数说明、调用模式和错误处理策略。

## 适用范围

本文档适用于：

- 了解系统 API 使用的开发者
- 进行 API 集成的工程师
- 进行架构设计的技术人员

**注意**：PrintSpooler 为纯 ArkTS/TypeScript 应用，不涉及 N-API 绑定，直接使用 OpenHarmony 系统提供的 JS API。

## 关键结论

1. **主要依赖**：@kit.BasicServicesKit（print API）
2. **Wifi P2P**：使用 @ohos.wifi 进行打印机发现和连接
3. **文件操作**：使用 @ohos.file.fs 进行文件读写
4. **图像处理**：使用 @ohos.multimedia.image 进行图像处理
5. **事件驱动**：大量使用事件监听模式（print.on(), wifi.on()）

---

## API 清单

### 1. 打印框架 API (@kit.BasicServicesKit - print)

#### 1.1 打印机能力查询

**API**: `print.queryPrinterCapabilityByUri(uri: string, id: string): Promise<PrinterCapability>`

**调用位置**：
- `feature/ippPrint/src/main/ets/common/napi/NativeApi.ts:43`

**用途**：查询指定打印机的打印能力

**参数**：
- `uri`: 打印机 URI（如 `ipp://192.168.1.100:631/ipp/print`）
- `id`: 打印机 ID

**返回值**：
- `Promise<PrinterCapability>` - 打印机能力对象

**错误处理**：
```typescript
print.queryPrinterCapabilityByUri(uri, id)
  .then((result) => {
    // 处理能力信息
    Log.debug(TAG, 'result: ' + JSON.stringify(result));
  })
  .catch((error) => {
    // 处理错误
    Log.error(TAG, 'error: ' + JSON.stringify(error));
  });
```

**证据**：`feature/ippPrint/src/main/ets/common/napi/NativeApi.ts:34-58`

---

#### 1.2 添加打印机到 CUPS

**API**: `print.addPrinterToCups(uri: string, name: string, make: string): Promise<void>`

**调用位置**：
- `feature/ippPrint/src/main/ets/common/napi/NativeApi.ts:67`

**用途**：向 CUPS 服务添加打印机配置

**参数**：
- `uri`: 打印机 URI
- `name`: 打印机名称
- `make`: 打印机制造商

**返回值**：
- `Promise<void>`

**错误处理**：
```typescript
print.addPrinterToCups(uri, name, make)
  .then((result) => {
    Log.debug(TAG, 'result: ' + JSON.stringify(result));
  })
  .catch((error) => {
    Log.error(TAG, 'error: ' + JSON.stringify(error));
  });
```

**证据**：`feature/ippPrint/src/main/ets/common/napi/NativeApi.ts:60-72`

---

#### 1.3 打印任务状态监听

**API**: `print.on(event: 'jobStateChange', callback: (state: PrintJobState, job: PrintJob) => void): void`

**调用位置**：
- `entry/src/main/ets/Controller/PrintJobManager.ts:173`
- `entry/src/main/ets/Controller/PrintJobController.ets:26`

**用途**：监听打印任务状态变化事件

**参数**：
- `event`: 事件类型，固定为 `'jobStateChange'`
- `callback`: 状态变化回调
  - `state`: 任务状态（PREPARED, QUEUED, RUNNING, BLOCKED, COMPLETED）
  - `job`: 打印任务对象

**任务状态枚举**（推断）：
| 状态 | 说明 |
|------|------|
| PRINT_JOB_PREPARED | 任务已准备 |
| PRINT_JOB_QUEUED | 任务已排队 |
| PRINT_JOB_RUNNING | 任务运行中 |
| PRINT_JOB_BLOCKED | 任务阻塞 |
| PRINT_JOB_COMPLETED | 任务完成 |

**使用示例**：
```typescript
public registerPrintJobCallback(): void {
  print.on('jobStateChange', this.onJobStateChanged);
}

private onJobStateChanged = (state: print.PrintJobState, job: print.PrintJob): void => {
  if (state === null || job === null) {
    Log.error(TAG, 'state changed null data');
    return;
  }
  // 更新任务状态
  this.getModel().printJobStateChange(job.jobId, job.jobState, job.jobSubState);
};
```

**证据**：`entry/src/main/ets/Controller/PrintJobController.ets:173-201`

---

#### 1.4 打印任务管理

**API**: `printJobMgr` (从 @kit.BasicServicesKit 导入)

**调用位置**：
- `entry/src/main/ets/Controller/PrintJobManager.ts`
- `entry/src/main/ets/Model/JobViewModel/PrintJobViewModel.ets:25`

**用途**：打印任务管理器接口

**主要方法**（从使用推断）：
- 创建打印任务
- 查询任务状态
- 取消任务

**证据**：
- PrintJobManager.ts:8 - `import { printJobMgr } from '@kit.BasicServicesKit';`
- PrintJobViewModel.ets:25 - `import { printJobMgr } from '@kit.BasicServicesKit';`

---

### 2. Wifi P2P API (@ohos.wifi)

#### 2.1 P2P 设备发现

**API**: `wifi.startDiscoverDevices(): void`

**调用位置**：
- `feature/ippPrint/src/main/ets/common/discovery/P2pDiscoveryChannel.ts:223`

**用途**：开始 P2P 设备发现

**证据**：`feature/ippPrint/src/main/ets/common/discovery/P2pDiscoveryChannel.ts:223`

---

#### 2.2 获取 P2P 设备列表

**API**: `wifi.getP2pPeerDevices(): Promise<WifiP2pDevice[]>`

**调用位置**：
- `feature/ippPrint/src/main/ets/common/discovery/P2pDiscoveryChannel.ts:224`

**用途**：获取当前发现的 P2P 设备列表

**证据**：`feature/ippPrint/src/main/ets/common/discovery/P2pDiscoveryChannel.ts:224`

---

#### 2.3 P2P 设备变化监听

**API**: `wifi.on(event: 'p2pPeerDeviceChange', callback: (peers: WifiP2pDevice[]) => void): void`

**调用位置**：
- `feature/ippPrint/src/main/ets/common/discovery/P2pDiscoveryChannel.ts:211`

**用途**：监听 P2P 设备列表变化

**证据**：`feature/ippPrint/src/main/ets/common/discovery/P2pDiscoveryChannel.ts:211`

---

#### 2.4 连接打印机

**API**: `wifi.connectToPrinter(config: WifiP2PConfig): Promise<boolean>`

**调用位置**：
- `entry/src/main/ets/Common/Adapter/WifiP2pHelper.ts`

**用途**：连接到指定的 P2P 打印机

**参数**：
- `config`: P2P 连接配置

**证据**：推断自 README.md 示例和代码上下文

---

### 3. 文件操作 API (@ohos.file.fs)

#### 3.1 打开文件

**API**: `Fileio.openSync(path: string, mode: number): File`

**调用位置**：
- `entry/src/main/ets/Common/Utils/FileUtil.ts`
- README.md 示例代码

**用途**：同步打开文件

**参数**：
- `path`: 文件路径或 URI
- `mode`: 打开模式（只读、读写等）

**常量**（定义于 `common/src/main/ets/model/Constants.ts:129-134`）：
- `READ = 0o0`
- `READ_WRITE = 0o2`
- `CREATE = 0o100`
- `OPEN_SYNC1 = 0o102`
- `OPEN_SYNC2 = 0o640`
- `OPEN_FAIL = -1`

**使用示例**：
```typescript
import Fileio from '@ohos.file.fs';

let file = Fileio.openSync(uri, Constants.READ_WRITE);
let fd = file.fd;
```

**证据**：
- README.md:86-93
- Constants.ts:129-134

---

### 4. 图像处理 API (@ohos.multimedia.image)

#### 4.1 创建图像源

**API**: `image.createImageSource(fd: number): ImageSource`

**调用位置**：
- `entry/src/main/ets/Common/Utils/FileUtil.ts`
- README.md 示例代码

**用途**：从文件描述符创建图像源

**参数**：
- `fd`: 文件描述符

**使用示例**：
```typescript
import image from '@ohos.multimedia.image';

let imageSource = image.createImageSource(file.fd);
```

**证据**：README.md:86-93

---

### 5. Bundle 管理 API (@ohos.bundle.bundleManager)

#### 5.1 获取自身 Bundle 信息

**API**: `bundleManager.getBundleInfoForSelf(flags: BundleFlag): Promise<BundleInfo>`

**调用位置**：
- `entry/src/main/ets/MainAbility/MainAbility.ets:49`

**用途**：获取应用的 Bundle 信息

**参数**：
- `flags`: 查询标志

**使用示例**：
```typescript
bundleManager.getBundleInfoForSelf(bundleManager.BundleFlag.GET_BUNDLE_INFO_DEFAULT)
  .then((bundleInfo) => {
    let versionName = bundleInfo.versionName as string;
    // 使用版本信息
  });
```

**证据**：`entry/src/main/ets/MainAbility/MainAbility.ets:49-52`

---

### 6. Ability API (@ohos.app.ability)

#### 6.1 UIExtensionAbility

**基类**：`UIExtensionAbility`

**调用位置**：
- `entry/src/main/ets/MainAbility/MainAbility.ets:36`

**用途**：UI 扩展能力基类

**关键生命周期方法**：
- `onCreate(): void` - 创建时调用
- `onSessionCreate(want, session): void` - 会话创建时调用
- `onDestroy(): void` - 销毁时调用

**证据**：`entry/src/main/ets/MainAbility/MainAbility.ets:36`

---

#### 6.2 PrintExtensionAbility

**基类**：`PrintExtensionAbility`

**调用位置**：
- `entry/src/main/ets/ServiceExtAbility/PrintExtension.ts:36`

**用途**：打印扩展能力基类

**关键生命周期方法**：
- `onCreate(want): void` - 创建时调用
- `onStartDiscoverPrinter(): void` - 开始发现打印机
- `onStopDiscoverPrinter(): void` - 停止发现
- `onConnectPrinter(printerId): void` - 连接打印机
- `onDisconnectPrinter(printerId): void` - 断开打印机
- `onStartPrintJob(printJob): void` - 开始打印任务
- `onCancelPrintJob(printJob): void` - 取消任务
- `onRequestPrinterCapability(printerId): PrinterCapability` - 查询能力
- `onDestroy(): void` - 销毁时调用

**证据**：`entry/src/main/ets/ServiceExtAbility/PrintExtension.ts:36`

---

### 7. 应用存储 API (@ohos.data.preferences)

#### 7.1 首选项存储

**API**：`preferences.getPreferences(context, name): Promise<Preferences>`

**调用位置**：
- `entry/src/main/ets/Common/Adapter/PreferencesAdapter.ts`

**用途**：获取应用首选项存储实例

**证据**：`entry/src/main/ets/Common/Adapter/PreferencesAdapter.ts`

---

## API 使用统计

| API 分类 | 使用次数 | 主要用途 |
|---------|---------|---------|
| @kit.BasicServicesKit (print) | 29+ | 打印任务管理、能力查询 |
| @ohos.wifi | 10+ | P2P 发现和连接 |
| @ohos.file.fs | 15+ | 文件操作 |
| @ohos.multimedia.image | 10+ | 图像处理和预览 |
| @ohos.bundle.bundleManager | 5+ | Bundle 信息获取 |
| @ohos.app.ability | 20+ | Ability 生命周期 |
| @ohos.data.preferences | 5+ | 应用数据存储 |

**证据**：基于 import 语句统计

---

## 错误码映射

PrintSpooler 使用的错误码定义在 `common/src/main/ets/model/Constants.ts:33-44`：

| 错误码 | 值 | 说明 | 可能原因 |
|-------|-----|------|---------|
| E_PRINT_NONE | 0 | 无错误 |
| E_PRINT_NO_PERMISSION | 201 | 无权限 | 权限未授予 |
| E_PRINT_INVALID_PARAMETER | 401 | 无效参数 | 参数类型或范围错误 |
| E_PRINT_GENERIC_FAILURE | 13100001 | 通用失败 | 未知错误 |
| E_PRINT_RPC_FAILURE | 13100002 | RPC 失败 | 进程间通信失败 |
| E_PRINT_SERVER_FAILURE | 13100003 | 打印服务失败 | 打印服务异常 |
| E_PRINT_INVALID_EXTENSION | 13100004 | 无效打印扩展 | 扩展配置错误 |
| E_PRINT_INVALID_PRINTER | 13100005 | 无效打印机 | 打印机不可用 |
| E_PRINT_INVALID_PRINTJOB | 13100006 | 无效打印任务 | 任务数据错误 |
| E_PRINT_FILE_IO | 13100007 | 文件 I/O 错误 | 文件读写失败 |

**证据**：`common/src/main/ets/model/Constants.ts:33-44`

---

## 权限要求

使用系统 API 需要以下权限（完整列表见 [安全风险评审](08_Security_Review.md)）：

| API | 需要权限 | 权限名称 |
|-----|---------|---------|
| print API | - | 无特殊权限 |
| @ohos.wifi | ✅ | ohos.permission.GET_WIFI_INFO |
| @ohos.wifi | ✅ | ohos.permission.SET_WIFI_INFO |
| @ohos.file.fs | ✅ | ohos.permission.FILE_ACCESS_MANAGER |
| @ohos.bundle.bundleManager | ✅ | ohos.permission.GET_BUNDLE_INFO_PRIVILEGED |

**证据**：`entry/src/main/module.json5:66-161`

---

## 调用模式

### Promise 模式

大部分系统 API 返回 Promise，推荐使用 Promise 链：

```typescript
// 正确：使用 Promise 链
api.method1()
  .then(result1 => {
    return api.method2(result1);
  })
  .then(result2 => {
    // 处理最终结果
  })
  .catch(error => {
    // 统一错误处理
  });
```

### 事件监听模式

对于事件驱动的 API（如状态变化），使用事件监听：

```typescript
// 注册监听
api.on('event', callback);

// 取消监听
api.off('event', callback);
```

**证据**：`entry/src/main/ets/Controller/PrintJobController.ets:173`

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目整体介绍
- [架构设计](03_Architecture.md) - 组件关系和数据流
- [内部 API](05_Internal_API.md) - 模块间接口
- [安全风险评审](08_Security_Review.md) - 权限和安全要求
