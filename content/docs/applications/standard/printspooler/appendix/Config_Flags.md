# 关键配置

## 目的

本文档汇总 PrintSpooler 项目的关键配置参数、常量和开关，供开发者快速查阅。

## 适用范围

本文档适用于：

- 需要了解配置参数的开发者
- 进行参数调优的技术人员
- 问题排查的工程师

---

## 全局常量

### 应用信息

| 常量名 | 值 | 定义位置 | 说明 |
|---------|-----|---------|------|
| BUNDLE_NAME | `'com.ohos.spooler'` | Constants.ts:141 | 应用包名 |
| MAIN_ABILITY_NAME | `'MainAbility'` | Constants.ts:142 | 主 Ability 名称 |
| JOB_MANAGER_ABILITY_NAME | `'JobManagerAbility'` | Constants.ts:143 | 任务管理 Ability 名称 |
| KEEP_ALIVE_ABILITY_NAME | `'KeepAliveAbility'` | Constants.ts:144 | 保活 Ability 名称 |

**证据**：`common/src/main/ets/model/Constants.ts:141-145`

---

### Want 参数键

| 键名 | 用途 | 定义位置 |
|------|------|---------|
| WANT_JOB_ID_KEY | 任务 ID | Constants.ts:147 |
| WANT_FILE_LIST_KEY | 文件列表 | Constants.ts:148 |
| WANT_CALLERPID_KEY | 调用者 PID | Constants.ts:149 |
| WANT_PKG_NAME_KEY | 调用者包名 | Constants.ts:150 |
| WANT_DOCUMENT_NAME_KEY | 文档名称 | Constants.ts:151 |
| wantPrintAttributeKey | 打印属性 | Constants.ts:152 |

**证据**：`common/src/main/ets/model/Constants.ts:147-152`

---

### AppStorage 键

| 存储键 | 数据类型 | 用途 | 定义位置 |
|---------|---------|------|---------|
| JOB_QUEUE_NAME | Array<PrintJob> | 打印任务队列 | AppStorageKeyName.ts:176 |
| PRINTER_QUEUE_NAME | Array<Printer> | 打印机队列 | AppStorageKeyName.ts:177 |
| PRINT_EXTENSION_LIST_NAME | Array<Extension> | 打印扩展列表 | AppStorageKeyName.ts:178 |
| CONFIG_LANGUAGE | string | 配置语言 | AppStorageKeyName.ts:179 |
| START_PRINT_TIME | number | 打印开始时间 | AppStorageKeyName.ts:180 |
| INGRESS_PACKAGE | string | 调用者包名 | AppStorageKeyName.ts:181 |
| APP_VERSION | string | 应用版本 | AppStorageKeyName.ts:182 |
| DOCUMENT_NAME | string | 文档名称 | AppStorageKeyName.ts:183 |
| PREVIEW_PAGE_INSTANCE | number | 预览页面实例数 | AppStorageKeyName.ts:184 |
| imageSourcesName | Array<FileModel> | 图像源列表 | AppStorageKeyName.ts:185 |

**证据**：`common/src/main/ets/model/Constants.ts:175-186`

---

### GlobalThis 键

| 存储键 | 数据类型 | 用途 | 定义位置 |
|---------|---------|------|---------|
| KEY_SERVICE_CONNECT_OPTIONS | Object | 服务连接选项 | GlobalThisStorageKey.ts:189 |
| KEY_JOB_ID | string | 任务 ID | GlobalThisStorageKey.ts:190 |
| KEY_MEDIA_SIZE_UTIL | MediaSizeUtil | 纸张尺寸工具 | GlobalThisStorageKey.ts:191 |
| KEY_PRINT_ADAPTER | PrintAdapter | 打印适配器 | GlobalThisStorageKey.ts:192 |
| KEY_PREFERENCES_ADAPTER | PreferencesAdapter | 首选项适配器 | GlobalThisStorageKey.ts:193 |
| KEY_MAIN_ABILITY_CONTEXT | UIAbilityContext | 主 Ability 上下文 | GlobalThisStorageKey.ts:194 |
| KEY_JOB_MANAGER_ABILITY_CONTEXT | UIAbilityContext | 任务管理 Ability 上下文 | GlobalThisStorageKey.ts:196 |
| KEY_CURRENT_PIXELMAP | PixelMap | 当前像素图 | GlobalThisStorageKey.ts:200 |

**证据**：`common/src/main/ets/model/Constants.ts:188-205`

---

## 打印相关配置

### IPP 协议配置

| 常量名 | 值 | 定义位置 | 说明 |
|---------|-----|---------|------|
| SCHEME_IPP | `'ipp'` | PrintConstants.ts:16 | IPP 协议方案 |
| SCHEME_IPPS | `'ipps'` | PrintConstants.ts:17 | IPPS 协议方案 |
| SERVICE_IPP | `'_ipp._tcp'` | PrintConstants.ts:18 | IPP 服务类型 |
| SERVICE_IPPS | `'_ipps._tcp'` | PrintConstants.ts:19 | IPPS 服务类型 |
| IPP_PORT | `631` | PrintConstants.ts:22 | IPP 端口 |
| IPP_PATH | `'ipp/print'` | PrintConstants.ts:23 | IPP 路径 |

**证据**：`common/src/main/ets/model/PrintConstants.ts:16-23`

### 打印机品牌

| 常量名 | 值 | 定义位置 | 说明 |
|---------|-----|---------|------|
| EPSON_PRINTER | `'EPSON'` | PrintConstants.ts:20 | EPSON 打印机 |
| BISHENG_PRINTER | `'PixLab V1'` | PrintConstants.ts:21 | 必升打印机 |

**证据**：`common/src/main/ets/model/PrintConstants.ts:20-21`

### 发现配置

| 常量名 | 值 | 定义位置 | 说明 |
|---------|-----|---------|------|
| P2P_DISCOVERY_EVENT_ID | `2` | PrintConstants.ts:24 | P2P 发现事件 ID |
| MDNS_EMITTER_EVENT_ID | `3` | PrintConstants.ts:25 | mDNS 事件 ID |
| WIFI_POWER_CLOSED | `1` | PrintConstants.ts:26 | WiFi 关闭状态 |
| P2P_DISCOVERY_DELAY | `500` | PrintConstants.ts:27 | P2P 发现延迟（ms） |

**证据**：`common/src/main/ets/model/PrintConstants.ts:24-27`

---

## 打印参数枚举

### 页面范围

| 枚举值 | 值 | 说明 | 定义位置 |
|---------|-----|------|---------|
| ALL | `0` | Constants.ts:53 | 全部页面 |
| RANGE | `1` | Constants.ts:54 | 指定范围 |
| CUSTOM | `2` | Constants.ts:55 | 自定义范围 |

**证据**：`common/src/main/ets/model/Constants.ts:53-57`

### 页面方向

| 枚举值 | 值 | 说明 | 定义位置 |
|---------|-----|------|---------|
| AUTO | `0` | Constants.ts:59 | 自适应 |
| VERTICAL | `1` | Constants.ts:60 | 竖向 |
| LANDSCAPE | `2` | Constants.ts:61 | 横向 |

**证据**：`common/src/main/ets/model/Constants.ts:59-63`

### 打印质量

| 枚举值 | 值 | 说明 | 定义位置 |
|---------|-----|------|---------|
| ECONOMY | `3` | Constants.ts:65 | 经济 |
| STANDARD | `4` | Constants.ts:66 | 标准 |
| BEST | `5` | Constants.ts:67 | 最佳 |

**证据**：`common/src/main/ets/model/Constants.ts:65-69`

### 双面打印

| 枚举值 | 值 | 说明 | 定义位置 |
|---------|-----|------|---------|
| SINGLE | `0` | Constants.ts:71 | 单面 |
| LONG | `1` | Constants.ts:72 | 双面沿长边 |
| SHORT | `2` | Constants.ts:73 | 双面沿短边 |

**证据**：`common/src/main/ets/model/Constants.ts:71-75`

### 彩色模式

| 枚举值 | 值 | 说明 | 定义位置 |
|---------|-----|------|---------|
| MONOCHROME | `0` | Constants.ts:77 | 黑白 |
| COLOR | `1` | Constants.ts:78 | 彩色 |

**证据**：`common/src/main/ets/model/Constants.ts:77-80`

### 纸张类型

| 枚举值 | 值 | 说明 | 定义位置 |
|---------|-----|------|---------|
| NORMAL | `0` | Constants.ts:82 | 普通纸 |
| PHOTO | `10` | Constants.ts:83 | 相片纸 |

**证据**：`common/src/main/ets/model/Constants.ts:82-85`

---

## 系统限制配置

### 数值限制

| 常量名 | 值 | 单位 | 定义位置 | 说明 |
|---------|-----|------|---------|
| MAX_PIXELMAP | `33554432` | bytes | Constants.ts:120 | 最大像素图（32MB） |
| MAX_PAGES | `100` | pages | Constants.ts:121 | 最大页数 |
| MAX_CUSTOM_PRINT_RANGE_LENGTH | `50` | chars | Constants.ts:122 | 最大自定义范围长度 |
| CONNECT_COUNT | `40` | times | Constants.ts:111 | 连接尝试次数 |
| COUNTDOWN_TO_FAIL | `25` | seconds | Constants.ts:117 | 倒计时超时 |

**证据**：`common/src/main/ets/model/Constants.ts:111,117,120-122`

### UI 尺寸

| 常量名 | 值 | 单位 | 定义位置 | 说明 |
|---------|-----|------|---------|
| CANVAS_MAX_WIDTH | `432` | pixels | Constants.ts:118 | 画布最大宽度 |
| CANVAS_MAX_HEIGHT | `302` | pixels | Constants.ts:119 | 画布最大高度 |
| MAIN_WINDOW_WIDTH | `480` | pixels | Constants.ts:124 | 主窗口宽度 |
| MAIN_WINDOW_HEIGHT | `853` | pixels | Constants.ts:125 | 主窗口高度 |
| JOB_WINDOW_WIDTH | `394` | pixels | Constants.ts:126 | 任务窗口宽度 |
| JOB_WINDOW_HEIGHT | `550` | pixels | Constants.ts:127 | 任务窗口高度 |

**证据**：`common/src/main/ets/model/Constants.ts:118,119,124-127`

---

## 文件操作配置

### 文件打开模式

| 常量名 | 值（八进制） | 说明 | 定义位置 |
|---------|-------------|------|---------|
| READ | `0o0` | 只读 | Constants.ts:129 |
| READ_WRITE | `0o2` | 读写 | Constants.ts:130 |
| CREATE | `0o100` | 创建 | Constants.ts:131 |
| OPEN_SYNC1 | `0o102` | 同步打开 1 | Constants.ts:132 |
| OPEN_SYNC2 | `0o640` | 同步打开 2 | Constants.ts:133 |
| OPEN_FAIL | `-1` | 打开失败 | Constants.ts:134 |

**证据**：`common/src/main/ets/model/Constants.ts:129-134`

### 文件后缀

| 常量名 | 值 | 说明 | 定义位置 |
|---------|-----|------|---------|
| TEMP_JOB_FOLDER | `'jobs'` | 临时任务文件夹 | Constants.ts:164 |
| JPEG_SUFFIX | `'.jpeg'` | JPEG 后缀 | Constants.ts:165 |

**证据**：`common/src/main/ets/model/Constants.ts:164-165`

---

## 公共事件

### 应用内公共事件

| 事件名 | 值 | 用途 | 定义位置 |
|------|-----|------|---------|
| PRINTER_STATE_CHANGE_EVENT | `1000` | 打印机状态变化 | AppCommonEvent.ts:209 |
| PRINTER_UPDATE_CAPABILITY_EVENT | `1001` | 打印机能力更新 | AppCommonEvent.ts:210 |
| START_JOB_MANAGER_ABILITY_EVENT | `1002` | 启动任务管理 | AppCommonEvent.ts:211 |
| TERMINATE_JOB_MANAGER_ABILITY_EVENT | `1003` | 终止任务管理 | AppCommonEvent.ts:212 |
| PRINTER_INVALID_EVENT | `1004` | 打印机无效 | AppCommonEvent.ts:213 |
| WLAN_INACTIVE_EVENT | `1005` | WiFi 未激活 | AppCommonEvent.ts:214 |
| WLAN_ACTIVE_EVENT | `1006` | WiFi 已激活 | AppCommonEvent.ts:215 |
| ADD_PRINTER_EVENT | `1007` | 添加打印机 | AppCommonEvent.ts:216 |

**证据**：`common/src/main/ets/model/Constants.ts:208-217`

---

## 首选项键

### 隐私声明

| 键名 | 值 | 说明 | 定义位置 |
|-----|-----|------|---------|
| KEY_PRIVACY_STATEMENT_PREFERENCES | `'AGREE_PRIVACY_STATEMENT'` | 隐私声明首选项 | PreferencesKey.ts:220 |

**证据**：`common/src/main/ets/model/Constants.ts:220`

---

## 权限配置

### 权限清单

| 权限名称 | 使用场景 | 定义位置 |
|---------|---------|---------|
| ohos.permission.MANAGE_PRINT_JOB | 打印任务管理 | entry/module.json5:67-77 |
| ohos.permission.GET_WIFI_INFO | 获取 WiFi 信息 | entry/module.json5:78-88 |
| ohos.permission.SET_WIFI_INFO | 设置 WiFi 信息 | entry/module.json5:89-98 |
| ohos.permission.PUBLISH_AGENT_REMINDER | 发布代理提醒 | entry/module.json5:99-108 |
| ohos.permission.INTERNET | 网络访问 | entry/module.json5:109-118 |
| ohos.permission.securityguard.REPORT_SECURITY_INFO | 上报安全信息 | entry/module.json5:120-129 |
| ohos.permission.GET_RUNNING_INFO | 获取运行信息 | entry/module.json5:131-140 |
| ohos.permission.FILE_ACCESS_MANAGER | 文件访问管理 | entry/module.json5:141-150 |
| ohos.permission.GET_BUNDLE_INFO_PRIVILEGED | 获取 Bundle 信息 | entry/module.json5:151-160 |

**证据**：`entry/src/main/module.json5:66-161`

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目整体介绍
- [目录结构](02_Directory_Structure.md) - 模块组织
- [内部 API](05_Internal_API.md) - 模块间接口
