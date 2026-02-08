# OpenHarmony 打印扫描框架 - 项目概览

**目的**: 提供项目定位、核心能力、运行环境和关键概念，让读者在5分钟内理解项目定位和用途。

**适用范围**: 新人学习者、应用开发者、系统集成者

**关键结论**:
- ✅ OpenHarmony 打印扫描框架是系统能力组件，为应用提供统一的打印和扫描功能
- ✅ 支持多种协议（CUPS、IPP、SANE、USB、SMB）
- ✅ 采用分层架构设计（接口层-框架层-服务层-驱动层）
- ✅ 通过N-API对外暴露JavaScript API，通过NDK对外暴露C/C++ API
- ✅ 三个系统服务协同工作（打印服务3707、扫描服务3708、SANE服务3709）

---

## 一、项目定位

### 1.1 一句话定义

**OpenHarmony 打印扫描框架**是OpenHarmony操作系统的系统能力组件，为应用程序提供统一的打印和扫描功能，支持多种设备连接方式和协议。

### 1.2 能力边界

**打印能力**:
- ✅ 打印机发现和管理（USB、网络、SMB）
- ✅ 打印任务创建、提交、监控和取消
- ✅ 驱动集成（CUPS、IPP Everywhere、PPD驱动、BSUNI驱动）
- ✅ 打印选项配置（页面大小、分辨率、颜色模式、双面打印）
- ✅ 打印状态监控（打印机状态、任务进度）
- ✅ 扩展机制（第三方打印机驱动扩展）

**扫描能力**:
- ✅ 扫描仪发现和管理（USB、网络）
- ✅ 扫描参数设置和获取
- ✅ 扫描控制（启动、停止、取消）
- ✅ 图像数据获取
- ✅ 扫描进度监控
- ✅ 扫描设备管理（添加、删除）

**不能做的事情**:
- ❌ 不直接支持网络打印协议实现（依赖CUPS）
- ❌ 不提供打印预览UI（由PrintSpooler应用提供）
- ❌ 不实现底层打印机驱动（由第三方扩展或CUPS提供）
- ❌ 不实现扫描仪驱动（由SANE后端提供）

### 1.3 运行环境

**系统要求**:
- OpenHarmony 3.0+ 标准系统
- 支持SystemAbility框架
- 支持IPC通信机制

**依赖组件**:
- CUPS打印系统（external dependency）
- SANE扫描后端（external dependency）
- OpenHarmony基础能力（ability_base、access_token等）

**权限要求**:
- `ohos.permission.PRINT` - 打印基础权限
- `ohos.permission.MANAGE_PRINT_JOB` - 打印任务管理权限
- 系统应用权限 - 某些高级功能

**资源需求**:
- ROM: 2MB
- RAM: 10MB

### 1.4 系统架构概览

打印扫描框架采用**四层架构设计**：

```
┌─────────────────────────────────────────────────┐
│         应用层 (Applications)               │
│  JS/TS应用  |  C/C++应用              │
└─────────────────┬─────────────────────────────┘
                │
┌─────────────────▼─────────────────────────────┐
│       接口层 (Interfaces)                 │
│  ┌──────────┬──────────┬──────────┐     │
│  │  N-API    │  ANI      │  NDK    │     │
│  │ (JS API)  │ (TS API)  │ (C API)  │     │
│  └──────────┴──────────┴──────────┘     │
└─────────────────┬─────────────────────────────┘
                │
┌─────────────────▼─────────────────────────────┐
│       框架层 (Frameworks)                  │
│  ┌───────────────────────────────────────┐   │
│  │  ohprint/ohscan (NDK)           │   │
│  │  models/helper (数据+工具)      │   │
│  │  innerkitsimpl (IPC客户端)        │   │
│  │  kits/extension (扩展框架)      │   │
│  └───────────────────────────────────────┘   │
└─────────────────┬─────────────────────────────┘
                │
┌─────────────────▼─────────────────────────────┐
│       服务层 (Services)                   │
│  ┌───────────────────────────────────────┐   │
│  │  print_service (SA:3707)       │   │
│  │  scan_service (SA:3708)        │   │
│  │  sane_service (SA:3709)         │   │
│  └───────────────────────────────────────┘   │
└─────────────────┬─────────────────────────────┘
                │
┌─────────────────▼─────────────────────────────┐
│       驱动层 (Backends)                 │
│  ┌───────────────────────────────────────┐   │
│  │  CUPS (打印后端)               │   │
│  │  SANE Backends (扫描后端)      │   │
│  │  IPP Everywhere (网络打印)       │   │
│  │  SMB (SMB打印机)               │   │
│  └───────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

---

## 二、核心概念

### 2.1 系统能力 (SystemAbility)

打印扫描框架通过OpenHarmony的SystemAbility机制注册为系统能力，可被应用跨进程调用。

| 服务ID | 服务名称 | 说明 |
|-------|----------|------|
| 3707 | PrintServiceAbility | 打印服务 |
| 3708 | ScanServiceAbility | 扫描服务 |
| 3709 | SaneServerManager | SANE后端服务 |

### 2.2 N-API vs NDK

**N-API (Native API)**:
- Node.js接口
- 提供给JavaScript/TypeScript应用
- 异步操作返回Promise
- 模块名：`@ohos.print`、`@ohos.scan`

**NDK (Native Development Kit)**:
- C/C++接口
- 提供给Native应用（C/C++）
- 同步操作
- 库名：`libohprint.so`、`libohscan.so`

### 2.3 打印扩展 (Print Extension)

打印框架支持第三方开发者开发打印机驱动扩展，通过PrintExtensionAbility机制。

**扩展类型**:
- IPP Everywhere驱动
- PPD驱动（PostScript描述文件）
- BSUNI驱动（特定厂商驱动）
- SMB/CIFS打印机驱动

### 2.4 SANE后端 (SANE Backends)

扫描框架基于SANE（Scanner Access Now Easy）库，支持多种扫描仪后端。

**特点**:
- 统一的扫描API
- 支持多种扫描协议（USB、网络）
- 设备参数查询和配置
- 图片数据获取

---

## 三、快速开始

### 3.1 最小打印示例

```javascript
import print from '@ohos.print';

try {
  // 1. 启动打印机发现
  await print.startDiscoverPrinter();

  // 2. 获取发现的打印机列表（通过事件）
  print.on('printerEvent', (event) => {
    console.log('发现打印机:', event.info);
  });

  // 3. 连接打印机
  await print.connectPrinter('printer-id-here');

  // 4. 创建打印任务
  const printJob = {
    jobId: 'job-' + Date.now(),
    printerId: 'printer-id-here',
    content: 'print-content-here',
    contentType: 'application/pdf',
    option: {
      colorMode: print.PrintColorMode.COLOR_MODE_COLOR,
      pageSize: print.PrintPageType.PAGE_ISO_A4,
      copies: 1
    }
  };

  // 5. 启动打印
  await print.startPrintJob(printJob);

  // 6. 监听打印状态
  print.on('printJobState', (state) => {
    console.log('打印状态:', state);
  });

} catch (error) {
  console.error('打印失败:', error);
}
```

### 3.2 最小扫描示例

```javascript
import scan from '@ohos.scan';

try {
  // 1. 初始化扫描服务
  await scan.init();

  // 2. 发现扫描仪
  await scan.startScannerDiscovery();

  // 3. 获取扫描仪列表（通过事件）
  scan.on('scanDeviceFound', (device) => {
    console.log('发现扫描仪:', device);
  });

  // 4. 打开扫描仪
  await scan.openScanner('scanner-id-here');

  // 5. 获取扫描参数
  const params = await scan.getScannerParameters('scanner-id-here');
  console.log('扫描参数:', params);

  // 6. 设置扫描参数
  await scan.setScanAutoOption('scanner-id-here', 0); // 设置分辨率等

  // 7. 启动扫描
  await scan.startScan('scanner-id-here', false); // false = 非批量模式

  // 8. 监听扫描进度
  scan.on('scanProgress', (progress) => {
    console.log('扫描进度:', progress);
  });

  // 9. 获取扫描结果
  // 扫描完成后通过回调获取图片数据

} catch (error) {
  console.error('扫描失败:', error);
}
```

### 3.3 权限配置

在`module.json5`中添加以下权限：

```json
{
  "requestPermissions": [
    {
      "name": "ohos.permission.PRINT",
      "reason": "需要打印权限"
    }
  ]
}
```

---

## 四、相关链接

- [架构与数据流](02_Architecture.md) - 详细架构说明和组件关系
- [目录结构](03_CodeMap.md) - 代码组织和文件导航
- [N-API接口文档](04_Interface.md) - 完整API参考
- [攻击面分析](06_AttackSurface.md) - 安全研究视角
- [快速使用指南](05_QuickStart.md) - 详细使用示例和常见问题
- [安全风险评估](07_SecurityReview.md) - 详细安全分析
- [常见问题](appendix/FAQ.md) - 构建、运行、调试问题

---

**生成时间**: 2026-02-07
**相关证据**: `bundle.json:1`、`README.md:5-105`、`print.gni:19-36`
