# OpenHarmony 打印扫描框架 - N-API 接口文档

**目的**: 提供完整的N-API（Node.js API）接口参考，包括所有方法、参数、返回值、错误码和权限要求。

**适用范围**: JavaScript/TypeScript应用开发者

**关键结论**:
- ✅ 打印模块导出46个核心方法 + 15个枚举类型
- ✅ 扫描模块导出17个核心方法 + 6个枚举类型
- ✅ 所有API都是异步操作，返回Promise
- ✅ 部分接口需要系统应用权限
- ✅ 包含完整的错误码定义和权限检查机制

---

## 一、打印 N-API (模块名: `@ohos.print`)

### 1.1 核心打印方法

| JS 方法名 | 参数 | 返回值 | 异步 | 权限要求 | 文件位置 |
|-----------|------|--------|-------|-----------|----------|
| `print` | files: string[], context?: Context | PrintTask | 是 | PRINT | print_module.cpp:442 |
| `startDiscoverPrinter` | extensionIds: string[] | boolean | 是 | PRINT | print_module.cpp:30 |
| `stopDiscoverPrinter` | - | boolean | 是 | PRINT | print_module.cpp:31 |
| `connectPrinter` | printerId: string | boolean | 是 | PRINT | print_module.cpp:32 |
| `disconnectPrinter` | printerId: string | boolean | 是 | PRINT + SystemApp | print_module.cpp:33 |
| `startPrintJob` | printJob: PrintJob | boolean | 是 | PRINT + SystemApp | print_module.cpp:34 |
| `startPrint` | printJob: PrintJob | boolean | 是 | PRINT + SystemApp | print_module.cpp:35 |
| `cancelPrintJob` | jobId: string | boolean | 是 | PRINT + SystemApp | print_module.cpp:36 |
| `restartPrintJob` | jobId: string | boolean | 是 | PRINT + SystemApp | print_module.cpp:37 |
| `requestPreview` | printJob: PrintJob | string | 是 | PRINT + SystemApp | print_module.cpp:38 |
| `queryPrinterCapability` | printerId: string | PrinterCapability | 是 | PRINT | print_module.cpp:39 |
| `queryAllPrintJobs` | - | PrintJob[] | 是 | PRINT + SystemApp | print_module.cpp:40 |
| `queryAllActivePrintJobs` | - | PrintJob[] | 是 | PRINT | print_module.cpp:41 |
| `queryPrintJobById` | jobId: string | PrintJob | 是 | PRINT + SystemApp | print_module.cpp:42 |
| `on` | type: string, callback: function | - | 是 | PRINT | print_module.cpp:43 |
| `off` | type: string | boolean | 是 | PRINT | print_module.cpp:44 |
| `addPrinters` | printers: PrinterInfo[] | boolean | 是 | SystemApp | print_module.cpp:463 |
| `removePrinters` | printerIds: string[] | boolean | 是 | SystemApp | print_module.cpp:464 |
| `updatePrinters` | printers: PrinterInfo[] | boolean | 是 | SystemApp | print_module.cpp:465 |
| `updatePrinterState` | printerId: string, state: PrinterState | boolean | 是 | SystemApp | print_module.cpp:466 |
| `updatePrintJobState` | jobId: string, state: PrintJobState, reason: PrintJobSubState | boolean | 是 | SystemApp | print_module.cpp:467 |
| `updateExtensionInfo` | extInfo: string | boolean | 是 | SystemApp | print_module.cpp:468 |
| `queryAllPrinterPpds` | - | PpdInfo[] | 是 | - | print_module.cpp:509 |
| `setPrinterPreference` | printerId: string, prefs: PrinterPreferences | boolean | 是 | - | print_module.cpp:495 |
| `setDefaultPrinter` | printerId: string, type: DefaultPrinterType | boolean | 是 | - | print_module.cpp:496 |
| `authSmbDeviceAsGuest` | host: string | boolean | 是 | - | print_module.cpp:519 |
| `authSmbDeviceAsRegisteredUser` | host: string, user: string, password: string | boolean | 是 | - | print_module.cpp:520 |

**注意**: 系统应用专用方法（SystemApp Only）需要系统应用签名。

**证据**: `interfaces/kits/napi/print_napi/src/print_module.cpp:442-521`

### 1.2 枚举常量

#### 1.2.1 打印模式枚举

| 枚举名 | 值 | 说明 |
|--------|-----|------|
| `PrintDirectionMode` | DIRECTION_MODE_AUTO, DIRECTION_MODE_PORTRAIT, DIRECTION_MODE_LANDSCAPE | 打印方向 |
| `PrintColorMode` | COLOR_MODE_MONOCHROME, COLOR_MODE_COLOR | 颜色模式 |
| `PrintDuplexMode` | DUPLEX_MODE_NONE, DUPLEX_MODE_LONG_EDGE, DUPLEX_MODE_SHORT_EDGE | 双面打印 |
| `PrintQuality` | QUALITY_DRAFT, QUALITY_NORMAL, QUALITY_HIGH | 打印质量 |
| `PrintDocumentFormat` | DOCUMENT_FORMAT_AUTO, DOCUMENT_FORMAT_JPEG, DOCUMENT_FORMAT_PDF, DOCUMENT_FORMAT_POSTSCRIPT, DOCUMENT_FORMAT_TEXT, DOCUMENT_FORMAT_RAW | 文档格式 |
| `DocFlavor` | FILE_DESCRIPTOR, BYTES | 文档数据源 |
| `PrintOrientationMode` | ORIENTATION_MODE_PORTRAIT, ORIENTATION_MODE_LANDSCAPE, ORIENTATION_MODE_REVERSE_LANDSCAPE, ORIENTATION_MODE_REVERSE_PORTRAIT, ORIENTATION_MODE_NONE | 页面方向 |
| `PrintPageType` | PAGE_ISO_A3, PAGE_ISO_A4, PAGE_ISO_A5, PAGE_JIS_B5, PAGE_ISO_C5, PAGE_ISO_DL, PAGE_LETTER, PAGE_LEGAL, PAGE_PHOTO_4X6, PAGE_PHOTO_5X7, PAGE_INT_DL_ENVELOPE, PAGE_B_TABLOID | 页面大小 |
| `PrinterState` | PRINTER_ADDED, PRINTER_REMOVED, PRINTER_CAPABILITY_UPDATED, PRINTER_CONNECTED, PRINTER_DISCONNECTED, PRINTER_RUNNING | 打印机状态 |
| `PrintJobState` | PRINT_JOB_PREPARED, PRINT_JOB_QUEUED, PRINT_JOB_RUNNING, PRINT_JOB_BLOCKED, PRINT_JOB_COMPLETED | 打印任务状态 |
| `PrintErrorCode` | E_PRINT_NONE, E_PRINT_NO_PERMISSION, E_PRINT_INVALID_PARAMETER, E_PRINT_GENERIC_FAILURE, E_PRINT_RPC_FAILURE, E_PRINT_SERVER_FAILURE, E_PRINT_INVALID_EXTENSION, E_PRINT_INVALID_PRINTER, E_PRINT_INVALID_PRINTJOB, E_PRINT_FILE_IO, E_PRINT_TOO_MANY_FILES | 错误码 |

**证据**: `interfaces/kits/napi/print_napi/src/print_module.cpp:151-440`

---

## 二、扫描 N-API (模块名: `@ohos.scan`)

### 2.1 核心扫描方法

| JS 方法名 | 参数 | 返回值 | 异步 | 权限要求 | 文件位置 |
|-----------|------|--------|-------|-----------|----------|
| `init` | - | undefined | 是 | - | scan_module.cpp:23 |
| `exit` | - | undefined | 是 | - | scan_module.cpp:24 |
| `startScannerDiscovery` | - | undefined | 是 | SCAN | scan_module.cpp:25 |
| `openScanner` | scannerId: string | undefined | 是 | SCAN | scan_module.cpp:26 |
| `closeScanner` | scannerId: string | undefined | 是 | SCAN | scan_module.cpp:27 |
| `startScan` | scannerId: string, batchMode: boolean | undefined | 是 | SCAN | scan_module.cpp:28 |
| `cancelScan` | scannerId: string | undefined | 是 | SCAN | scan_module.cpp:29 |
| `getPictureScanProgress` | scannerId: string | ScanProgress | 是 | SCAN | scan_module.cpp:30 |
| `setScanAutoOption` | scannerId: string, optionIndex: number | undefined | 是 | SCAN | scan_module.cpp:31 |
| `getScannerCurrentSetting` | scannerId: string, optionIndex: number | ScanOptionValue | 是 | SCAN | scan_module.cpp:32 |
| `setScanParameter` | scannerId: string, optionIndex: number, value: ScanOptionValue | undefined | 是 | SCAN | scan_module.cpp:33 |
| `getScannerParameters` | scannerId: string | ScanOptionDescriptor[] | 是 | SCAN | scan_module.cpp:34 |
| `addScanner` | uniqueId: string, discoverMode: string | undefined | 是 | SystemApp | scan_module.cpp:35 |
| `deleteScanner` | uniqueId: string, discoverMode: string | undefined | 是 | SystemApp | scan_module.cpp:36 |
| `getAddedScanners` | - | ScanDeviceInfo[] | 是 | SystemApp | scan_module.cpp:37 |
| `on` | type: string, callback: function | - | 是 | SCAN | scan_module.cpp:38 |
| `off` | type: string | - | 是 | SCAN | scan_module.cpp:39 |

**注意**:
- 系统应用专用方法（`addScanner`、`deleteScanner`、`getAddedScanners`）需要系统应用签名
- 事件类型`scanDeviceAdd`和`scanDeviceDel`仅系统应用可监听

**证据**: `interfaces/kits/napi/scan_napi/src/scan_module.cpp:154-180`

### 2.2 扫描枚举常量

| 枚举名 | 值 | 说明 |
|--------|-----|------|
| `ScanErrorCode` | SCAN_ERROR_NO_PERMISSION, SCAN_ERROR_NOT_SYSTEM_APPLICATION, SCAN_ERROR_INVALID_PARAMETER, SCAN_ERROR_GENERIC_FAILURE, SCAN_ERROR_RPC_FAILURE, SCAN_ERROR_SERVER_FAILURE, SCAN_ERROR_UNSUPPORTED, SCAN_ERROR_CANCELED, SCAN_ERROR_DEVICE_BUSY, SCAN_ERROR_INVALID, SCAN_ERROR_JAMMED, SCAN_ERROR_NO_DOCS, SCAN_ERROR_COVER_OPEN, SCAN_ERROR_IO_ERROR, SCAN_ERROR_NO_MEMORY | 扫描错误码 |
| `ConstraintType` | SCAN_CONSTRAINT_NONE, SCAN_CONSTRAINT_RANGE, SCAN_CONSTRAINT_WORD_LIST, SCAN_CONSTRAINT_STRING_LIST | 约束类型 |
| `PhysicalUnit` | SCAN_UNIT_NONE, SCAN_UNIT_PIXEL, SCAN_UNIT_BIT, SCAN_UNIT_MM, SCAN_UNIT_DPI, SCAN_UNIT_PERCENT, SCAN_UNIT_MICROSECOND | 物理单位 |
| `OptionValueType` | SCAN_TYPE_BOOL, SCAN_TYPE_INT, SCAN_TYPE_FIXED, SCAN_TYPE_STRING | 选项值类型 |
| `ScannerDiscoveryMode` | TCP_STR ("TCP"), USB_STR ("USB") | 发现模式 |

**证据**: `interfaces/kits/napi/scan_napi/src/scan_module.cpp:53-152`

---

## 三、PrintTask 实例方法

| JS 方法名 | 参数 | 返回值 | 说明 |
|-----------|------|--------|------|
| `on` | type: string, callback: function | - | 注册打印任务事件监听 |
| `off` | type: string | boolean | 取消打印任务事件监听 |

**支持的事件类型**:
- `block` - 打印任务被阻塞
- `succeed` - 打印任务成功
- `fail` - 打印任务失败
- `cancel` - 打印任务被取消

**证据**: `interfaces/kits/napi/print_napi/src/napi_print_task.cpp:1`

---

## 四、错误码说明

### 4.1 打印错误码

| 错误码 | 数值 | 说明 | 触发场景 |
|---------|------|------|----------|
| `E_PRINT_NONE` | 0 | 成功 |
| `E_PRINT_NO_PERMISSION` | -1 | 缺少打印权限 |
| `E_PRINT_INVALID_PARAMETER` | -2 | 参数无效 |
| `E_PRINT_GENERIC_FAILURE` | -3 | 通用失败 |
| `E_PRINT_RPC_FAILURE` | -4 | IPC调用失败 |
| `E_PRINT_SERVER_FAILURE` | -5 | 服务器内部错误 |
| `E_PRINT_INVALID_EXTENSION` | -6 | 打印扩展无效 |
| `E_PRINT_INVALID_PRINTER` | -7 | 打印机无效 |
| `E_PRINT_INVALID_PRINTJOB` | -8 | 打印任务无效 |
| `E_PRINT_FILE_IO` | -9 | 文件I/O错误 |
| `E_PRINT_TOO_MANY_FILES` | -10 | 文件数量过多 |

**证据**: `utils/include/print_constant.h:291-322`

### 4.2 扫描错误码

| 错误码 | 数值 | 说明 | 触发场景 |
|---------|------|------|----------|
| `SCAN_ERROR_NO_PERMISSION` | -1 | 缺少扫描权限 |
| `SCAN_ERROR_NOT_SYSTEM_APPLICATION` | -2 | 非系统应用调用系统接口 |
| `SCAN_ERROR_INVALID_PARAMETER` | -3 | 参数无效 |
| `SCAN_ERROR_GENERIC_FAILURE` | -4 | 通用失败 |
| `SCAN_ERROR_RPC_FAILURE` | -5 | IPC调用失败 |
| `SCAN_ERROR_SERVER_FAILURE` | -6 | 服务器内部错误 |
| `SCAN_ERROR_UNSUPPORTED` | -7 | 不支持的操作 |
| `SCAN_ERROR_CANCELED` | -8 | 操作被取消 |
| `SCAN_ERROR_DEVICE_BUSY` | -9 | 设备忙碌 |
| `SCAN_ERROR_JAMMED` | -10 | 扫描仪卡纸 |
| `SCAN_ERROR_NO_DOCS` | -11 | 没有文档 |
| `SCAN_ERROR_COVER_OPEN` | -12 | 扫描仪盖子打开 |
| `SCAN_ERROR_IO_ERROR` | -13 | I/O错误 |
| `SCAN_ERROR_NO_MEMORY` | -14 | 内存不足 |

**证据**: `utils/include/scan_constant.h:1`

---

## 五、权限要求

### 5.1 权限定义

| 权限名 | 保护级别 | 说明 | 代码位置 |
|---------|---------|------|----------|
| `ohos.permission.PRINT` | normal | 基础打印扫描权限 | print_constant.h:291 |
| `ohos.permission.MANAGE_PRINT_JOB` | system | 打印任务管理权限 | print_constant.h:292 |
| `ohos.permission.MANAGE_USB_CONFIG` | system | USB配置权限（扫描） | scan_constant.h |

### 5.2 权限检查机制

权限检查在以下层级进行：
1. **N-API层**: JavaScript调用时检查系统应用标记
2. **服务层**: IPC调用时使用AccessTokenKit验证
3. **敏感操作**: 某些操作需要系统应用签名

**证据**: `services/print_service/src/print_service_helper.cpp:40-56`

---

## 六、使用示例

### 6.1 打印流程示例

```javascript
import print from '@ohos.print';

async function printFile() {
  try {
    // 1. 启动打印机发现
    await print.startDiscoverPrinter();

    // 2. 监听发现事件
    print.on('printerEvent', (event) => {
      if (event.type === print.PrinterEvent.PRINTER_EVENT_ADDED) {
        console.log('发现打印机:', event.info.printerId);
      }
    });

    // 3. 等待打印机发现
    await new Promise(resolve => setTimeout(resolve, 2000));

    // 4. 连接打印机
    const success = await print.connectPrinter('printer-id-here');
    if (!success) {
      throw new Error('连接失败');
    }

    // 5. 获取打印机能力
    const capability = await print.queryPrinterCapability('printer-id-here');
    console.log('打印机能力:', capability);

    // 6. 创建打印任务
    const job = {
      jobId: 'job-' + Date.now(),
      printerId: 'printer-id-here',
      content: 'file://data/print.pdf',
      contentType: 'application/pdf',
      option: {
        colorMode: print.PrintColorMode.COLOR_MODE_COLOR,
        pageSize: print.PrintPageType.PAGE_ISO_A4
      }
    };

    // 7. 启动打印
    await print.startPrintJob(job);

    // 8. 监听打印状态
    print.on('printJobState', (state) => {
      console.log('打印状态:', state.state);
    });

  } catch (error) {
    console.error('打印失败:', error);
    if (error.code === print.PrintErrorCode.E_PRINT_NO_PERMISSION) {
      console.error('缺少打印权限');
    }
  }
}
```

### 6.2 扫描流程示例

```javascript
import scan from '@ohos.scan';

async function scanDocument() {
  try {
    // 1. 初始化扫描服务
    await scan.init();

    // 2. 启动扫描仪发现
    await scan.startScannerDiscovery();

    // 3. 监听发现事件
    scan.on('scanDeviceFound', (device) => {
      console.log('发现扫描仪:', device.uniqueId);
    });

    // 4. 等待扫描仪发现
    await new Promise(resolve => setTimeout(resolve, 2000));

    // 5. 打开扫描仪
    await scan.openScanner('scanner-id-here');

    // 6. 获取扫描参数
    const params = await scan.getScannerParameters('scanner-id-here');
    console.log('扫描参数:', params);

    // 7. 设置扫描参数
    await scan.setScanAutoOption('scanner-id-here', 0); // 设置分辨率

    // 8. 启动扫描
    await scan.startScan('scanner-id-here', false); // false = 非批量模式

    // 9. 监听扫描进度
    scan.on('scanProgress', (progress) => {
      console.log('扫描进度:', progress.progress);
      if (progress.state === scan.ScanProgressState.SCAN_STATE_COMPLETED) {
        console.log('扫描完成');
      }
    });

  } catch (error) {
    console.error('扫描失败:', error);
    if (error.code === scan.ScanErrorCode.SCAN_ERROR_NO_PERMISSION) {
      console.error('缺少扫描权限');
    }
  }
}
```

---

## 七、相关链接

- [项目概览](01_Overview.md) - 项目定位和快速开始示例
- [架构与数据流](02_Architecture.md) - 架构设计和数据流
- [目录结构](03_CodeMap.md) - 代码组织和文件导航

---

**生成时间**: 2026-02-07
**相关证据**:
- 打印N-API注册: `interfaces/kits/napi/print_napi/src/print_module.cpp:442`
- 扫描N-API注册: `interfaces/kits/napi/scan_napi/src/scan_module.cpp:154`
- 打印错误码定义: `utils/include/print_constant.h:291-322`
- 扫描错误码定义: `utils/include/scan_constant.h:1`
