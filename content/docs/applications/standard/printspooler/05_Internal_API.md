# 内部 API

## 目的

本文档描述 PrintSpooler 模块间的内部接口，包括接口定义、依赖关系、稳定性和可替换点。

## 适用范围

本文档适用于：

- 模块开发人员
- 架构师
- 需要理解模块接口的开发者

## 关键结论

1. **模块化设计**：entry、common、ippPrint、driverEntry 四个独立模块
2. **依赖方向**：entry → ippPrint → common，driverEntry 独立
3. **接口稳定性**：common 模块接口最稳定，ippPrint 模块次之
4. **可替换点**：打印扩展、发现服务、连接服务支持替换

---

## 模块接口图

```
┌─────────────────────────────────────────────────┐
│              entry 模块                      │
│  (UI、业务逻辑、Ability)                  │
│                                             │
│  ┌────────────────────────────────────────┐ │
│  │     Controller 层                   │ │
│  │                                      │ │
│  │  ┌──────────┐  ┌────────────┐ │ │
│  │  │ PrintJob │  │ PrinterDisc │ │ │
│  │  │ Manager  │  │ Controller  │ │ │
│  │  └────┬─────┘  └─────┬──────┘ │ │
│  │       │                 │         │ │
│  └───────┼─────────────────┼─────────┘ │
│          │                 │         │
└──────────┼─────────────────┼─────────┘
           │                 │
           ▼                 ▼
┌─────────────────────────────────────────────────┐
│              ippPrint 模块                   │
│  (IPP 协议、发现、连接)                    │
│                                             │
│  ┌────────────────────────────────────────┐ │
│  │        Service 层                 │ │
│  │                                      │ │
│  │  ┌──────┐  ┌──────────┐    │ │
│  │  │ P2P   │  │  Local   │    │ │
│  │  │ Disco- │  │ Discovery│    │ │
│  │  │ very   │  │ Session  │    │ │
│  │  └────┬───┘  └─────┬────┘    │ │
│  │       │              │           │ │
│  │       ▼              ▼           │ │
│  │  ┌────────────┐              │ │
│  │  │ Connection │              │ │
│  │  │ Service    │              │ │
│  │  └────────────┘              │ │
│  │       │                       │ │
│  └───────┼───────────────────────┘ │
└──────────┼───────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│              common 模块                       │
│  (工具类、常量、模型)                      │
│                                             │
│  ┌────────────────────────────────────────┐ │
│  │  Utils  │  Models  │  Constants  │ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

---

## Common 模块接口

### 导出接口

#### 工具类（Utils）

| 类 | 路径 | 职责 | 稳定性 |
|-----|------|------|--------|
| Log | `common/src/main/ets/utils/Log.ts` | 统一日志接口 | ✅ 稳定 |
| CheckEmptyUtils | `common/src/main/ets/utils/CheckEmptyUtils.ts` | 空值检查 | ✅ 稳定 |
| StringUtil | `common/src/main/ets/utils/StringUtil.ts` | 字符串处理 | ✅ 稳定 |
| DateUtils | `common/src/main/ets/utils/DateUtils.ts` | 日期处理 | ✅ 稳定 |
| UuidGenerator | `common/src/main/ets/utils/UuidGenerator.ts` | UUID 生成 | ✅ 稳定 |
| CopyUtil | `common/src/main/ets/utils/CopyUtil.ts` | 复制工具 | ✅ 稳定 |
| PrinterUtils | `common/src/main/ets/utils/PrinterUtils.ts` | 打印机工具 | ✅ 稳定 |
| PrintUtil | `common/src/main/ets/utils/PrintUtil.ts` | 打印工具 | ✅ 稳定 |
| MediaSizeHelper | `common/src/main/ets/utils/MediaSizeUtil.ts` | 纸张尺寸工具 | ✅ 稳定 |
| PermissionUtils | `common/src/main/ets/utils/PermissionUtils.ts` | 权限工具 | ✅ 稳定 |
| SingletonHelper | `common/src/main/ets/utils/SingletonHelper.ts` | 单例辅助 | ✅ 稳定 |
| GlobalObject | `common/src/main/ets/utils/GlobalObject.ts` | 全局对象 | ✅ 稳定 |

**证据**：所有类位于 `common/src/main/ets/utils/` 目录

#### 数据模型（Models）

| 类 | 路径 | 职责 | 稳定性 |
|-----|------|------|--------|
| Constants | `common/src/main/ets/model/Constants.ts` | 全局常量和枚举 | ✅ 稳定 |
| PrintConstants | `common/src/main/ets/model/PrintConstants.ts` | 打印相关常量 | ✅ 稳定 |
| PrintBean | `common/src/main/ets/model/PrintBean.ts` | 打印 Bean 定义 | ✅ 稳定 |
| MediaSize | `common/src/main/ets/model/MediaSize.ts` | 纸张尺寸定义 | ✅ 稳定 |
| MediaType | `common/src/main/ets/model/MediaType.ts` | 纸张类型定义 | ✅ 稳定 |
| ErrorMessage | `common/src/main/ets/model/ErrorMessage.ts` | 错误消息定义 | ✅ 稳定 |
| GlobalThisHelper | `common/src/main/ets/model/GlobalThisHelper.ts` | 全局对象辅助 | ✅ 稳定 |
| GlobalThisStorageKey | `common/src/main/ets/model/GlobalThisStorageKey.ts` | 存储键定义 | ✅ 稳定 |

**证据**：所有类位于 `common/src/main/ets/model/` 目录

#### 辅助类（Helpers）

| 类 | 路径 | 职责 | 稳定性 |
|-----|------|------|--------|
| AppStorageHelper | `entry/src/main/ets/Common/Adapter/AppStorageHelper.ts` | AppStorage 操作辅助 | ✅ 稳定 |
| PreferencesAdapter | `entry/src/main/ets/Common/Adapter/PreferencesAdapter.ts` | 首选项存储适配 | ✅ 稳定 |

**证据**：`entry/src/main/ets/Common/Adapter/` 目录

---

## IPPPrint 模块接口

### 导出接口

#### 发现服务（Discovery）

| 类 | 路径 | 职责 | 稳定性 |
|-----|------|------|--------|
| P2PDiscovery | `feature/ippPrint/src/main/ets/common/discovery/P2pDiscovery.ts` | P2P 打印机发现 | ⚠️ 可替换 |
| MdnsDiscovery | `feature/ippPrint/src/main/ets/common/discovery/MdnsDiscovery.ts` | mDNS 打印机发现 | ⚠️ 可替换 |
| Discovery | `feature/ippPrint/src/main/ets/common/discovery/Discovery.ts` | 发现抽象接口 | ✅ 稳定 |
| LocalDiscoverySession | `feature/ippPrint/src/main/ets/common/LocalDiscoverySession.ts` | 本地发现会话管理 | ⚠️ 可替换 |
| DiscoveredPrinter | `feature/ippPrint/src/main/ets/common/discovery/DiscoveredPrinter.ts` | 已发现打印机模型 | ✅ 稳定 |

**证据**：所有类位于 `feature/ippPrint/src/main/ets/common/discovery/` 目录

#### 连接服务（Connect）

| 类 | 路径 | 职责 | 稳定性 |
|-----|------|------|--------|
| P2pPrinterConnection | `feature/ippPrint/src/main/ets/common/connect/P2pPrinterConnection.ts` | P2P 打印机连接 | ⚠️ 可替换 |
| ConnectionListener | `feature/ippPrint/src/main/ets/common/connect/ConnectionListener.ts` | 连接状态监听 | ✅ 稳定 |

**证据**：所有类位于 `feature/ippPrint/src/main/ets/common/connect/` 目录

#### IPP 协议（IPP）

| 类 | 路径 | 职责 | 稳定性 |
|-----|------|------|--------|
| Backend | `feature/ippPrint/src/main/ets/common/ipp/Backend.ts` | IPP 后端实现 | ⚠️ 可替换 |
| CapabilitiesCache | `feature/ippPrint/src/main/ets/common/ipp/CapabilitiesCache.ts` | 打印机能力缓存 | ⚠️ 可替换 |
| LocalPrinterCapabilities | `feature/ippPrint/src/main/ets/common/napi/LocalPrinterCapabilities.ts` | 本地打印机能力 | ⚠️ 可替换 |

**证据**：所有类位于 `feature/ippPrint/src/main/ets/common/ipp/` 和 `common/napi/` 目录

#### 适配器（Adapters）

| 类 | 路径 | 职责 | 稳定性 |
|-----|------|------|--------|
| PrintServiceAdapter | `feature/ippPrint/src/main/ets/common/PrintServiceAdapter.ts` | 打印服务适配器 | ✅ 稳定 |
| WifiModel | `feature/ippPrint/src/main/ets/common/model/WifiModel.ts` | WiFi 模型 | ⚠️ 可替换 |

**证据**：`feature/ippPrint/src/main/ets/common/` 目录

---

## Entry 模块接口

### Ability 接口

| Ability | 路径 | 类型 | 稳定性 |
|---------|------|------|--------|
| MainAbility | `entry/src/main/ets/MainAbility/MainAbility.ets` | UIExtensionAbility | ✅ 稳定 |
| JobManagerAbility | `entry/src/main/ets/MainAbility/JobManagerAbility.ts` | UIExtensionAbility | ✅ 稳定 |
| PrintExtension | `entry/src/main/ets/ServiceExtAbility/PrintExtension.ts` | PrintExtensionAbility | ⚠️ 可替换 |
| PrintServiceExtAbility | 在 MainAbility 中定义 | sysDialog/print | ✅ 稳定 |

**证据**：`entry/src/main/module.json5:24-64`

### Controller 接口

| Controller | 路径 | 职责 | 稳定性 |
|-----------|------|------|--------|
| PrintJobManager | `entry/src/main/ets/Controller/PrintJobManager.ts` | 打印任务管理 | ⚠️ 可替换 |
| PrintExtensionController | `entry/src/main/ets/Controller/PrintExtensionController.ts` | 打印扩展控制 | ⚠️ 可替换 |
| PrinterDiscController | `entry/src/main/ets/Controller/PrinterDiscController.ts` | 打印机发现控制 | ⚠️ 可替换 |

**证据**：`entry/src/main/ets/Controller/` 目录

### Model 接口

| Model | 路径 | 职责 | 稳定性 |
|-------|------|------|--------|
| PrintPageModel | `entry/src/main/ets/Model/JobViewModel/PrintPageModel.ets` | 打印页面模型 | ⚠️ 可替换 |
| PrintJobViewModel | `entry/src/main/ets/Model/JobViewModel/PrintJobViewModel.ets` | 任务视图模型 | ⚠️ 可替换 |
| SelectionModel | `entry/src/main/ets/Model/SelectionModel.ets` | 选择模型 | ⚠️ 可替换 |
| PrinterDiscModel | `entry/src/main/ets/Model/PrinterDiscModel.ts` | 打印机发现模型 | ⚠️ 可替换 |
| FileModel | `entry/src/main/ets/Model/FileModel.ts` | 文件模型 | ✅ 稳定 |
| PrintExtensionModel | `entry/src/main/ets/Model/PrintExtensionModel.ts` | 打印扩展模型 | ✅ 稳定 |

**证据**：`entry/src/main/ets/Model/` 目录

---

## DriverEntry 模块接口

### ExtensionAbility

| ExtensionAbility | 路径 | 类型 | 稳定性 |
|---------------|------|------|--------|
| DriverExtensionAbility | `driverEntry/src/main/ets/driverentryability/MyDriverExtensionAbility.ts` | Driver (type="driver") | ⚠️ 可替换 |

**配置**：`driverEntry/src/main/module.json5:14-53`

**证据**：`driverEntry/src/main/ets/driverentryability/` 目录

---

## 依赖关系

### 模块依赖矩阵

| 模块 | 依赖的模块 | 依赖类型 |
|------|-----------|---------|
| entry | common | HAR |
| entry | ippPrint | HAR |
| ippPrint | common | HAR |
| driverEntry | 无 | - |

**证据**：
- entry/oh-package.json5
- ippPrint/oh-package.json5

### 依赖方向说明

```
┌─────────────┐
│  driverEntry │  (独立模块)
└─────────────┘

┌─────────────┐
│   entry     │  ← 最高层
│   (HAP)      │
└──────┬──────┘
       │ depends on
       ├──────────┐
       │          │
       │    ┌─────▼─────────┐
       │    │    ippPrint    │
       │    │     (HAR)      │
       │    └─────┬─────────┘
       │          │ depends on
       │    ┌─────▼─────────┐
       │    │    common      │
       │    │     (HAR)      │
       │    └─────────────────┘
       │
       └──> 无循环依赖
```

---

## 接口稳定性标注

### 稳定接口（✅）

以下接口经过验证，不建议随意修改：

1. **Common 模块的所有工具类**：
   - Log, CheckEmptyUtils, StringUtil, DateUtils 等
   - 证据：位于 `common/src/main/ets/utils/`

2. **Common 模块的所有数据模型**：
   - Constants, PrintConstants, PrintBean 等
   - 证据：位于 `common/src/main/ets/model/`

3. **MainAbility 和 JobManagerAbility**：
   - 应用主入口，影响系统启动
   - 证据：`entry/src/main/ets/MainAbility/`

### 可替换接口（⚠️）

以下接口支持替换或扩展：

1. **发现服务**：
   - P2PDiscovery, MdnsDiscovery
   - 可替换为其他发现方式（如 USB、蓝牙）
   - 证据：`feature/ippPrint/src/main/ets/common/discovery/`

2. **连接服务**：
   - P2pPrinterConnection
   - 可替换为其他连接方式
   - 证据：`feature/ippPrint/src/main/ets/common/connect/`

3. **IPP 后端**：
   - Backend
   - 可替换为其他 IPP 实现
   - 证据：`feature/ippPrint/src/main/ets/common/ipp/`

4. **打印扩展**：
   - PrintExtension
   - 可实现自定义打印扩展
   - 证据：`entry/src/main/ets/ServiceExtAbility/`

5. **驱动扩展**：
   - DriverExtensionAbility
   - 可添加新的打印机驱动
   - 证据：`driverEntry/src/main/ets/driverentryability/`

---

## 接口使用示例

### 使用 Common 模块的 Log

```typescript
import { Log } from '@ohos/common';

const TAG = 'MyComponent';

// 输出日志
Log.info(TAG, 'message');
Log.debug(TAG, 'message');
Log.error(TAG, 'error message');
```

**证据**：`common/src/main/ets/utils/Log.ts`

### 使用 Common 模块的 Constants

```typescript
import { Constants } from '@ohos/common';

// 使用常量
let bundleName = Constants.BUNDLE_NAME;
let jobIdKey = Constants.WANT_JOB_ID_KEY;
```

**证据**：`common/src/main/ets/model/Constants.ts`

### 使用 ippPrint 模块的发现服务

```typescript
import { LocalDiscoverySession } from '@ohos/ippprint';

// 创建发现会话
let session = new LocalDiscoverySession(printServiceAdapter);

// 开始发现
session.startPrinterDiscovery();

// 停止发现
session.stopPrinterDiscovery();
```

**证据**：`feature/ippPrint/src/main/ets/common/LocalDiscoverySession.ts`

---

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 模块组织细节
- [架构设计](03_Architecture.md) - 组件关系和数据流
- [对外 API](04_External_API.md) - OpenHarmony 系统 API 使用
