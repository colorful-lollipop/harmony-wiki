# OpenHarmony Print Scan Framework - 对外 API (N-API)

**目的**: 详细说明面向应用开发的 JavaScript API，包括方法、参数、返回值、权限和错误码

**适用范围**: OpenHarmony Print Scan Framework 3.1 - N-API 模块

---

## 目录

- [打印 N-API 模块 (print)](#打印-n-api-模块-print)
- [扫描 N-API 模块 (scan)](#扫描-napi-模块-scan)
- [打印扩展 N-API 模块 (printextensionability)](#打印扩展-napi-模块-printextensionability)
- [打印扩展上下文 N-API 模块 (printextensioncontext)](#打印扩展上下文-napi-模块-printextensioncontext)
- [权限说明](#权限说明)
- [错误码定义](#错误码定义)

---

## 打印 N-API 模块 (print)

**模块名称**: `print`
**注册位置**: `interfaces/kits/napi/print_napi/src/print_module.cpp:538`
**模块类型**: ohos_shared_library
**产物**: `libprint_napi.so`

### 导出枚举常量（12 个）

| 枚举类型 | 常量 | 值 | 说明 |
|-----------|------|-----|------|
| **PrintDirectionMode** | DIRECTION_MODE_AUTO | 0 | 自动方向 |
| | DIRECTION_MODE_PORTRAIT | 1 | 纵向 |
| | DIRECTION_MODE_LANDSCAPE | 2 | 横向 |
| **PrintColorMode** | COLOR_MODE_MONOCHROME | 0 | 单色 |
| | COLOR_MODE_COLOR | 1 | 彩色 |
| **PrintDuplexMode** | DUPLEX_MODE_NONE | 0 | 不双面 |
| | DUPLEX_MODE_LONG_EDGE | 1 | 长边双面 |
| | DUPLEX_MODE_SHORT_EDGE | 2 | 短边双面 |
| **PrintQuality** | QUALITY_DRAFT | 3 | 草稿质量 |
| | QUALITY_NORMAL | 4 | 正常质量 |
| | QUALITY_HIGH | 5 | 高质量 |
| **PrintDocumentFormat** | DOCUMENT_FORMAT_AUTO | 0 | 自动格式 |
| | DOCUMENT_FORMAT_JPEG | 1 | JPEG 格式 |
| | DOCUMENT_FORMAT_PDF | 2 | PDF 格式 |
| | DOCUMENT_FORMAT_POSTSCRIPT | 3 | PostScript 格式 |
| | DOCUMENT_FORMAT_TEXT | 4 | 文本格式 |
| | DOCUMENT_FORMAT_RAW | 5 | 原始格式 |
| **DocFlavor** | FILE_DESCRIPTOR | 0 | 文件描述符 |
| | BYTES | 1 | 字节数组 |
| **PrintOrientationMode** | ORIENTATION_MODE_PORTRAIT | 0 | 纵向 |
| | ORIENTATION_MODE_LANDSCAPE | 1 | 横向 |
| | ORIENTATION_MODE_REVERSE_LANDSCAPE | 2 | 反向横向 |
| | ORIENTATION_MODE_REVERSE_PORTRAIT | 3 | 反向纵向 |
| | ORIENTATION_MODE_NONE | 4 | 无方向 |
| **PrintPageType** | PAGE_ISO_A4 | 1 | ISO A4 纸张 |
| (更多纸张类型见代码） | PAGE_LETTER | 6 | Letter 纸张 |
| | PAGE_PHOTO_4X6 | 8 | 4x6 照片纸 |
| **PrintJobState** | PRINT_JOB_PREPARED | 0 | 已准备 |
| | PRINT_JOB_QUEUED | 1 | 已排队 |
| | PRINT_JOB_RUNNING | 2 | 运行中 |
| | PRINT_JOB_BLOCKED | 3 | 已阻塞 |
| | PRINT_JOB_COMPLETED | 4 | 已完成 |
| **PrintJobSubState** | PRINT_JOB_COMPLETED_SUCCESS | 0 | 打印成功 |
| | PRINT_JOB_BLOCKED_OFFLINE | 4 | 打印机离线 |
| | PRINT_JOB_BLOCKED_OUT_OF_PAPER | 7 | 缺纸 |
| | PRINT_JOB_BLOCKED_OUT_OF_INK | 8 | 缺墨 |
| | PRINT_JOB_BLOCKED_JAMMED | 10 | 卡纸 |
| | PRINT_JOB_BLOCKED_NETWORK_ERROR | 21 | 网络错误 |
| (更多子状态见代码） |  |  |  |

### 导出方法（40+ 个）

#### 主要打印方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `print()` | NapiPrintTask::Print | fileList, fdList | taskId | 异步 | PRINT | 提交打印任务 |
| `startDiscoverPrinter()` | NapiInnerPrint::StartDiscovery | extensionList | void | 异步 | PRINT | 开始发现打印机 |
| `stopDiscoverPrinter()` | NapiInnerPrint::StopDiscovery | - | void | 异步 | PRINT | 停止发现打印机 |
| `connectPrinter()` | NapiInnerPrint::ConnectPrinter | printerId | void | 异步 | PRINT | 连接打印机 |
| `disconnectPrinter()` | NapiInnerPrint::DisconnectPrinter | printerId | void | 异步 | PRINT | 断开打印机 |
| `startPrintJob()` | NapiInnerPrint::StartPrintJob | jobinfo | void | 异步 | PRINT, MANAGE_PRINT_JOB | 开始打印任务 |
| `cancelPrintJob()` | NapiInnerPrint::CancelPrintJob | jobId | void | 异步 | PRINT, MANAGE_PRINT_JOB | 取消打印任务 |
| `restartPrintJob()` | NapiInnerPrint::RestartPrintJob | jobId | void | 异步 | PRINT, MANAGE_PRINT_JOB | 重启打印任务 |
| `requestPreview()` | NapiInnerPrint::RequestPreview | jobinfo | previewResult | 异步 | PRINT | 请求打印预览 |

#### 查询方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `queryAllPrinterExtensionInfos()` | NapiInnerPrint::QueryExtensionInfo | - | extensionInfos | 异步 | PRINT | 查询所有打印扩展 |
| `queryPrinterCapability()` | NapiInnerPrint::QueryCapability | printerId | PrinterCapability | 异步 | PRINT | 查询打印机能力 |
| `queryAllPrintJobs()` | NapiInnerPrint::QueryAllPrintJob | - | printJobs | 异步 | PRINT | 查询所有打印任务 |
| `queryAllActivePrintJob()` | NapiInnerPrint::QueryAllActivePrintJob | - | printJobs | 异步 | PRINT | 查询所有活动打印任务 |
| `queryPrintJobById()` | NapiInnerPrint::QueryPrintJobById | printJobId | PrintJob | 异步 | PRINT | 按 ID 查询打印任务 |
| `queryAddedPrinter()` | NapiInnerPrint::QueryAddedPrinter | - | printerInfos | 异步 | PRINT | 查询已添加打印机 |
| `getAddedPrinterInfoById()` | NapiInnerPrint::GetAddedPrinterInfoById | printerId | PrinterInfo | 异步 | PRINT | 按打印机 ID 查询信息 |
| `getPrinterInfoById()` | NapiInnerPrint::GetAddedPrinterInfoById | printerId | PrinterInfo | 异步 | PRINT | 按打印机 ID 查询信息 |
| `queryAllPrinterPpds()` | NapiInnerPrint::QueryAllPrinterPpds | - | ppdList | 异步 | PRINT | 查询所有打印机 PPD |

#### 事件方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `on()` | NapiInnerPrint::On | taskId, type, listener | void | 异步 | PRINT | 注册打印事件回调 |
| `off()` | NapiInnerPrint::Off | taskId, type | void | 异步 | PRINT | 取消注册打印事件回调 |

#### 系统扩展方法（仅系统应用）

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `addPrinters()` | NapiPrintExt::AddPrinters | printerInfos | void | 异步 | - | 添加打印机（扩展用） |
| `removePrinters()` | NapiPrintExt::RemovePrinters | printerIds | void | 异步 | - | 删除打印机（扩展用） |
| `updatePrinters()` | NapiPrintExt::UpdatePrinters | printerInfos | void | 异步 | - | 更新打印机（扩展用） |
| `updatePrinterState()` | NapiPrintExt::UpdatePrinterState | printerId, state | void | 异步 | - | 更新打印机状态（扩展用） |
| `updatePrintJobState()` | NapiPrintExt::UpdatePrintJobStateOnlyForSystemApp | jobId, state, subState | void | 异步 | - | 更新打印任务状态（扩展用） |
| `updateExtensionInfo()` | NapiPrintExt::UpdateExtensionInfo | extInfo | void | 异步 | - | 更新扩展信息 |
| `addPrinterToCups()` | NapiPrintExt::AddPrinterToCups | printerUri, printerName, printerMake | void | 异步 | - | 添加打印机到 CUPS |
| `deletePrinterFromCups()` | NapiPrintExt::DeletePrinterFromCups | printerUri | void | 异步 | - | 从 CUPS 删除打印机 |
| `queryPrinterCapabilityByUri()` | NapiPrintExt::QueryPrinterCapabilityByUri | printerUri, printerId | PrinterCapability | 异步 | - | 按 URI 查询打印机能力 |

#### 设置方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `setPrinterPreference()` | NapiInnerPrint::SetPrinterPreference | printerId, preference | void | 异步 | PRINT | 设置打印机首选项 |
| `setDefaultPrinter()` | NapiInnerPrint::SetDefaultPrinter | printerId | void | 异步 | PRINT | 设置默认打印机 |
| `setPrinterPreferences()` | NapiInnerPrint::SetPrinterPreference | printerId, preferences | void | 异步 | PRINT | 设置打印机首选项 |

#### USB/IP 连接方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `discoverUsbPrinters()` | NapiPrintExt::DiscoverUsbPrinters | - | printerInfos | 异步 | - | 发现 USB 打印机 |
| `queryPrinterInfoByIp()` | NapiInnerPrint::QueryPrinterInfoByIp | ip | PrinterInfo | 异步 | - | 按 IP 查询打印机信息 |
| `connectPrinterByIpAndPpd()` | NapiInnerPrint::ConnectPrinterByIpAndPpd | ip, port, pdd | void | 异步 | - | 按 IP 和 PPD 连接打印机 |
| `connectPrinterByIdAndPpd()` | NapiInnerPrint::ConnectPrinterByIdAndPpd | printerId, pdd | void | 异步 | - | 按 ID 和 PPD 连接打印机 |
| `queryRecommendDriversById()` | NapiInnerPrint::QueryRecommendDriversById | printerId | driverList | 异步 | - | 查询推荐驱动 |

#### SMB 打印机方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `getSharedHosts()` | NapiInnerPrint::GetSharedHosts | - | hostList | 异步 | - | 获取共享主机列表 |
| `authSmbDeviceAsGuest()` | NapiInnerPrint::AuthSmbDeviceAsGuest | host | void | 异步 | - | 作为访客认证 SMB 设备 |
| `authSmbDeviceAsRegisteredUser()` | NapiInnerPrint::AuthSmbDeviceAsRegisteredUser | host, username, password | void | 异步 | - | 作为注册用户认证 SMB 设备 |

#### 其他方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `notifyPrintService()` | NapiInnerPrint::NotifyPrintService | jobId, type | void | 异步 | - | 通知打印服务 |
| `notifyPrintServiceEvent()` | NapiInnerPrint::NotifyPrintServiceEvent | type, info | void | 异步 | - | 通知打印服务事件 |
| `startGetPrintFile()` | NapiInnerPrint::StartGetPrintFile | jobId, attributes, fd | void | 异步 | PRINT, MANAGE_PRINT_JOB | 开始获取打印文件 |
| `savePdfFileJob()` | NapiInnerPrint::SavePdfFileJob | jobId, fd | void | 异步 | PRINT, MANAGE_PRINT_JOB | 保存 PDF 文件任务 |
| `analyzePrintEvents()` | NapiInnerPrint::AnalyzePrintEvents | events | analysisResult | 异步 | - | 分析打印事件 |
| `authPrintJob()` | NapiInnerPrint::AuthPrintJob | jobId, password | void | 异步 | PRINT, MANAGE_PRINT_JOB | 认证打印任务 |
| `checkPreferencesConflicts()` | NapiInnerPrint::CheckPreferencesConflicts | preference | conflictList | 异步 | - | 检查首选项冲突 |
| `checkPrintJobConflicts()` | NapiInnerPrint::CheckPrintJobConflicts | printJob | conflictList | 异步 | - | 检查打印任务冲突 |
| `getPrinterDefaultPreferences()` | NapiInnerPrint::GetPrinterDefaultPreferences | printerId | preferences | 异步 | - | 获取打印机默认首选项 |

### 调用链示例

```
JS 应用
  ↓
N-API 模块 (print)
  ↓
PrintServiceProxy (IPC 客户端)
  ↓
PrintServiceAbility (System Ability)
  ↓
CUPS / Vendor Driver
  ↓
打印机硬件
```

---

## 扫描 N-API 模块 (scan)

**模块名称**: `scan`
**注册位置**: `interfaces/kits/napi/scan_napi/src/scan_module.cpp:196`
**模块类型**: ohos_shared_library
**产物**: `libscan_napi.so`

### 导出枚举常量（6 个）

| 枚举类型 | 常量 | 值 | 说明 |
|-----------|------|-----|------|
| **ScanErrorCode** | SCAN_ERROR_NO_PERMISSION | 201 | 无权限 |
| | SCAN_ERROR_NOT_SYSTEM_APPLICATION | 202 | 非系统应用 |
| | SCAN_ERROR_INVALID_PARAMETER | 401 | 参数无效 |
| | SCAN_ERROR_GENERIC_FAILURE | 13100001 | 通用失败 |
| | SCAN_ERROR_RPC_FAILURE | 13100002 | RPC 失败 |
| | SCAN_ERROR_SERVER_FAILURE | 13100003 | 服务器失败 |
| | SCAN_ERROR_UNSUPPORTED | 13100004 | 不支持 |
| | SCAN_ERROR_CANCELLED | 13100005 | 已取消 |
| | SCAN_ERROR_DEVICE_BUSY | 13100006 | 设备忙 |
| | SCAN_ERROR_INVAL | 13100007 | 数据无效 |
| | SCAN_ERROR_JAMMED | 13100008 | 卡纸 |
| | SCAN_ERROR_NO_DOCS | 13100009 | 无文档 |
| | SCAN_ERROR_COVER_OPEN | 13100010 | 盖板打开 |
| | SCAN_ERROR_IO_ERROR | 13100011 | I/O 错误 |
| | SCAN_ERROR_NO_MEM | 13200012 | 无内存 |
| **ConstraintType** | SCAN_CONSTRAINT_NONE | 0 | 无约束 |
| | SCAN_CONSTRAINT_RANGE | 1 | 范围约束 |
| | SCAN_CONSTRAINT_WORD_LIST | 2 | 单词列表 |
| | SCAN_CONSTRAINT_STRING_LIST | 3 | 字符串列表 |
| **PhysicalUnit** | SCANNER_UNIT_NONE | 0 | 无单位 |
| | SCANNER_UNIT_PIXEL | 1 | 像素 |
| | SCANNER_UNIT_BIT | 2 | 位 |
| | SCANNER_UNIT_MM | 3 | 毫米 |
| | SCANNER_UNIT_DPI | 4 | DPI |
| | SCANNER_UNIT_PERCENT | 5 | 百分比 |
| | SCANNER_UNIT_MICROSECOND | 6 | 微秒 |
| **OptionValueType** | SCAN_VALUE_BOOL | 0 | 布尔值 |
| | SCAN_VALUE_NUM | 1 | 数值 |
| | SCAN_VALUE_FIXED | 2 | 固定值 |
| | SCAN_VALUE_STR | 3 | 字符串 |
| | SCAN_VALUE_BUTTON | 4 | 按钮 |
| | SCAN_VALUE_GROUP | 5 | 组 |

### 导出方法（15 个）

#### 初始化方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `init()` | NapiInnerScan::InitScan | - | void | 异步 | PRINT | 初始化扫描服务 |
| `exit()` | NapiInnerScan::ExitScan | - | void | 异步 | PRINT | 退出扫描服务 |

#### 设备管理方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `startScannerDiscovery()` | NapiInnerScan::GetScannerList | - | scannerList | 异步 | PRINT | 开始发现扫描器 |
| `openScanner()` | NapiInnerScan::OpenScanner | scannerId | void | 异步 | PRINT | 打开扫描器 |
| `closeScanner()` | NapiInnerScan::CloseScanner | scannerId | void | 异步 | PRINT | 关闭扫描器 |
| `addScanner()` | NapiInnerScan::AddScanner | uniqueId, discoverMode | void | 异步 | PRINT | 添加扫描器 |
| `deleteScanner()` | NapiInnerScan::DeleteScanner | serialNumber, discoverMode | void | 异步 | PRINT | 删除扫描器 |
| `getAddedScanners()` | NapiInnerScan::GetAddedScanner | - | scannerList | 异步 | PRINT | 获取已添加扫描器 |

#### 扫描控制方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `startScan()` | NapiInnerScan::StartScan | scannerId, batchMode | void | 异步 | PRINT | 开始扫描 |
| `cancelScan()` | NapiInnerScan::CancelScan | scannerId | void | 异步 | PRINT | 取消扫描 |
| `getPictureScanProgress()` | NapiInnerScan::GetScanProgress | scannerId | ScanProgress | 异步 | PRINT | 获取扫描进度 |

#### 参数配置方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `setScanAutoOption()` | NapiInnerScan::SetScanAutoOption | scannerId, optionIndex | void | 异步 | PRINT | 设置扫描自动选项 |
| `setScanOption()` | NapiInnerScan::SetScanOption | scannerId, optionIndex, value | void | 异步 | PRINT | 设置扫描选项 |
| `getScanOption()` | NapiInnerScan::GetScanOption | scannerId, optionIndex | ScanOptionDescriptor | 异步 | PRINT | 获取扫描选项 |
| `getScanParameters()` | NapiInnerScan::GetScanParameters | scannerId | ScanParameters | 异步 | PRINT | 获取扫描参数 |

#### 事件方法

| JS 方法 | C++ 入口 | 参数 | 返回值 | 同步/异步 | 权限 | 说明 |
|---------|----------|------|--------|---------|------|
| `on()` | NapiInnerScan::On | taskId, type, listener | void | 异步 | PRINT | 注册扫描事件回调 |
| `off()` | NapiInnerScan::Off | taskId, type | void | 异步 | PRINT | 取消注册扫描事件回调 |

### 调用链示例

```
JS 应用
  ↓
N-API 模块 (scan)
  ↓
ScanServiceProxy (IPC 客户端)
  ↓
ScanServiceAbility (System Ability)
  ↓
SaneServerManager (SANE 服务)
  ↓
SANE Backend
  ↓
扫描器硬件
```

---

## 打印扩展 N-API 模块 (printextensionability)

**模块名称**: `app.ability.PrintExtensionAbility`
**注册位置**: `interfaces/kits/jsnapi/print_extension/print_extension_module.cpp:23`
**模块类型**: ohos_shared_library
**产物**: `libprintextensionability_napi.so`
**特殊**: 包含 JS 代码（`print_extension.js` 和 `print_extension.abc`）

此模块是提供给第三方打印扩展应用的 JS API，用于实现自定义打印扩展。

---

## 打印扩展上下文 N-API 模块 (printextensioncontext)

**模块名称**: `PrintExtensionContext`
**注册位置**: `interfaces/kits/jsnapi/print_extensionctx/print_extension_context_module.cpp:23`
**模块类型**: ohos_shared_library
**产物**: `libprintextensioncontext_napi.so`

此模块提供打印扩展上下文的 JS API，用于扩展应用获取打印相关信息。

---

## 权限说明

### 权限列表

| 权限名称 | 说明 | 使用场景 |
|-----------|------|---------|
| `ohos.permission.PRINT` | 基础打印权限 | 所有打印/扫描 API 调用 |
| `ohos.permission.MANAGE_PRINT_JOB` | 管理打印任务权限 | 管理（取消/重启）打印任务 |
| `ohos.permission.MANAGE_USB_CONFIG` | USB 配置权限 | 扫描服务发现 USB 设备 |

### 权限检查位置

- **常量定义**: `utils/include/print_constant.h:291-292`
- **打印服务**: `services/print_service/src/print_service_ability.cpp` - `CheckPermission()`
- **扫描服务**: `services/scan_service/src/scan_service_ability.cpp:72-73`

---

## 错误码定义

### 打印错误码

| 错误码 | 名称 | 说明 | 触发条件 |
|---------|------|------|----------|
| 0 | E_PRINT_NONE | 无错误 | 操作成功 |
| 201 | E_PRINT_NO_PERMISSION | 无权限 | 应用未申请必要权限 |
| 202 | E_PRINT_ILLEGAL_USE_OF_SYSTEM_API | 非法使用系统 API | 非系统应用调用系统 API |
| 401 | E_PRINT_INVALID_PARAMETER | 参数无效 | 参数类型/范围不正确 |
| 13100001 | E_PRINT_GENERIC_FAILURE | 通用失败 | 通用操作失败 |
| 13100002 | E_PRINT_RPC_FAILURE | RPC 失败 | IPC 调用失败 |
| 13100003 | E_PRINT_SERVER_FAILURE | 服务器失败 | 打印服务失败 |
| 13100004 | E_PRINT_INVALID_EXTENSION | 无效扩展 | 扩展无效 |
| 13100005 | E_PRINT_INVALID_PRINTER | 无效打印机 | 打印机无效 |
| 13100006 | E_PRINT_INVALID_PRINTJOB | 无效打印任务 | 打印任务无效 |
| 13100007 | E_PRINT_FILE_IO | 文件 I/O 错误 | 文件操作失败 |
| 13100008 | E_PRINT_INVALID_TOKEN | 无效令牌 | 访问令牌无效 |
| 13100009 | E_PRINT_INVALID_USERID | 无效用户 ID | 用户 ID 无效 |
| 13100010 | E_PRINT_TOO_MANY_FILES | 文件过多 | 超过最大文件数 |
| 13100255 | E_PRINT_UNKNOWN | 未知错误 | 未知错误 |

**常量定义位置**: `utils/include/print_constant.h:79-97`

### 扫描错误码

| 错误码 | 名称 | 说明 | 触发条件 |
|---------|------|------|----------|
| 0 | E_SCAN_NONE | 无错误 | 操作成功 |
| 201 | E_SCAN_NO_PERMISSION | 无权限 | 应用未申请必要权限 |
| 202 | E_SCAN_ERROR_NOT_SYSTEM_APPLICATION | 非系统应用 | 非系统应用调用系统 API |
| 401 | E_SCAN_INVALID_PARAMETER | 参数无效 | 参数类型/范围不正确 |
| 13100001 | E_SCAN_GENERIC_FAILURE | 通用失败 | 通用操作失败 |
| 13100002 | E_SCAN_RPC_FAILURE | RPC 失败 | IPC 调用失败 |
| 13100003 | E_SCAN_SERVER_FAILURE | 服务器失败 | 扫描服务失败 |
| 13100004 | E_SCAN_UNSUPPORTED | 不支持 | 操作不支持 |
| 13100005 | E_SCAN_CANCELLED | 已取消 | 操作被取消 |
| 13100006 | E_SCAN_DEVICE_BUSY | 设备忙 | 扫描器忙 |
| 13100007 | E_SCAN_INVAL | 数据无效 | 数据无效（包括无设备） |
| 13100008 | E_SCAN_JAMMED | 卡纸 | 文档馈送器卡纸 |
| 13100009 | E_SCAN_NO_DOCS | 无文档 | 文档馈送器无文档 |
| 13100010 | E_SCAN_COVER_OPEN | 盖板打开 | 扫描器盖板打开 |
| 13100011 | E_SCAN_IO_ERROR | I/O 错误 | 设备 I/O 错误 |
| 13200012 | E_SCAN_NO_MEM | 无内存 | 内存不足 |
| 132000013 | E_SCAN_EOF | 文件结束 | 无更多数据 |
| 13200014 | E_SCAN_ACCESS_DENIED | 访问被拒绝 | 访问资源被拒绝 |

**常量定义位置**: `utils/include/scan_constant.h:92-111`

---

## API 使用示例

### 打印示例

```javascript
import print from '@ohos.print';

// 开始发现打印机
print.startDiscoverPrinter();

// 注册打印机发现事件
print.on('printerDiscover', (printers) => {
    console.log('发现打印机:', printers);
});

// 连接打印机
await print.connectPrinter('printer-id-123');

// 提交打印任务
const printJob = {
    jobId: 'job-001',
    printerId: 'printer-id-123',
    fileList: ['/data/files/document.pdf'],
    printAttributes: {
        pageSize: print.PrintPageType.PAGE_ISO_A4,
        colorMode: print.PrintColorMode.COLOR_MODE_COLOR,
        duplexMode: print.PrintDuplexMode.DUPLEX_MODE_LONG_EDGE
    }
};
await print.startPrintJob(printJob);
```

### 扫描示例

```javascript
import scan from '@ohos.scan';

// 初始化扫描服务
await scan.init();

// 发现扫描器
const scanners = await scan.startScannerDiscovery();
console.log('发现扫描器:', scanners);

// 打开扫描器
await scan.openScanner('scanner-id-123');

// 设置扫描参数
await scan.setScanOption('scanner-id-123', 0, {
    value: 300,
    type: scan.OptionValueType.SCAN_VALUE_NUM
});

// 开始扫描
await scan.startScan('scanner-id-123', false);

// 注册扫描进度事件
scan.on('scanProgress', (progress) => {
    console.log('扫描进度:', progress.completed, '/', progress.total);
});
```

---

**相关链接**:
- [项目概览](00_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构设计](03_Architecture.md)
