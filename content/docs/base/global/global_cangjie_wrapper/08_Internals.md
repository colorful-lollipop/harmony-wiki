# 内部 API 文档

> global_cangjie_wrapper 模块接口、依赖方向、稳定性标注

## 模块内部结构

### i18n 模块内部

```
ohos/i18n/
├── calendar.cj          # Calendar 类 + FFI 绑定 (主入口)
├── i18n_common.cj       # 公共类型与辅助函数
└── system.cj           # System 类
```

#### 内部辅助函数

| 函数 | 文件:行号 | 用途 |
|------|----------|------|
| `getErrorMsg(code: Int32)` | `i18n_common.cj:29` | 错误码转消息 |
| `CDate` struct | `i18n_common.cj:176` | C 兼容日期结构 |

#### FFI 绑定函数

> 位置: `calendar.cj:24-58`

| FFI 函数 | 用途 |
|----------|------|
| `FfiOHOSGetCalendar` | 创建日历实例 |
| `FfiOHOSCalendarSetTime` | 设置时间（毫秒） |
| `FfiOHOSCalendarSetDate` | 设置日期 |
| `FfiOHOSCalendarSetTimeZone` | 设置时区 |
| `FfiOHOSCalendarGetTimeZone` | 获取时区 |
| `FfiOHOSCalendarGetTimeInMillis` | 获取时间（毫秒） |
| `FfiOHOSCalendarGet` | 获取字段值 |
| `FfiOHOSCalendarSet` | 设置字段值 |
| `FfiOHOSCalendarAdd` | 增加字段值 |
| `FfiOHOSCalendarGetFirstDayOfWeek` | 获取每周首日 |
| `FfiOHOSCalendarSetFirstDayOfWeek` | 设置每周首日 |
| `FfiOHOSCalendarGetMinimalDaysInFirstWeek` | 获取首周最少天数 |
| `FfiOHOSCalendarSetMinimalDaysInFirstWeek` | 设置首周最少天数 |
| `FfiOHOSCalendarGetDisplayName` | 获取显示名称 |
| `FfiOHOSCalendarGetField` | 获取字段常量 |
| `FfiOHOSCalendarRelease` | 释放日历资源 |

### resource_manager 模块内部

```
ohos/resource_manager/
├── resource_manager.cj           # ResourceManager 主类
├── resource_manager_ffi.cj     # FFI 绑定 (30+ 函数)
├── resource_manager_common.cj   # 公共类型与工具
└── resource_manager_errors.cj   # 错误码定义
```

#### 内部辅助函数

| 函数 | 文件:行号 | 用途 |
|------|----------|------|
| `getResourceManager(context)` | `resource_manager.cj:48` | 带缓存的工厂函数 |
| `formatString()` | `resource_manager_common.cj:496` | 字符串格式化 (%d, %f, %s) |
| `throwIfNotSuccess()` | `resource_manager_errors.cj:26` | 错误码检查 |
| `getErrorMsg()` | `resource_manager_errors.cj:55` | 错误码转消息 |

#### FFI 绑定函数

> 位置: `resource_manager_ffi.cj:24-114` (30+ 函数)

| FFI 函数 | 用途 |
|----------|------|
| `CJ_GetResourceManagerStageMode` | 获取 Stage 模式 ResourceManager |
| `CJ_GetFAResMgr` | 获取 FA 模式 ResourceManager |
| `CJ_GetSystemResMgr` | 获取系统 ResourceManager |
| `CJ_GetRawFd` | 获取原始文件描述符 |
| `CJ_GetColor` | 获取颜色值 |
| `CJ_GetBoolean` | 获取布尔值 |
| `CJ_GetNumber` | 获取数值 |
| `CJ_GetMediaContent` | 获取媒体内容 |
| `CJ_GetMediaContentBase64` | 获取 Base64 编码媒体 |
| `CJ_GetPluralStringValue` | 获取复数字符串 |
| `CJ_GetStringArrayValue` | 获取字符串数组 |
| `CJ_GetString` | 获取字符串 |
| `CJ_GetConfiguration` | 获取配置 |
| `CJ_GetDeviceCapability` | 获取设备能力 |
| `CJ_AddResource` | 添加资源路径 |
| `CJ_RemoveResource` | 移除资源路径 |
| `CJ_GetLocales` | 获取区域列表 |
| ... | |

### resource 模块内部

```
ohos/resource/
├── app_resource.cj         # AppResource 类
└── resource_common.cj      # 资源 FFI + 辅助函数
```

#### 内部辅助函数

| 函数 | 文件:行号 | 用途 |
|------|----------|------|
| `getResourceString()` | `resource_common.cj` | 获取字符串资源 |
| `getResourceMedia()` | `resource_common.cj` | 获取媒体资源 |
| `getResourceColor()` | `resource_common.cj` | 获取颜色资源 |
| `getResourceLength()` | `resource_common.cj` | 获取尺寸资源 |
| `parseResourceParams()` | `resource_common.cj:184` | 解析资源参数 |
| `__GenerateResource__()` | `resource_common.cj:202` | 隐藏资源生成器 |

### raw_file_descriptor 模块内部

```
ohos/raw_file_descriptor/
└── raw_file_descriptor.cj  # RawFileDescriptor 类
```

## 稳定性标注

### 公开接口（Beta）

| 接口 | 位置 | 稳定性 | 依据 |
|------|------|--------|------|
| `Calendar` | `calendar.cj:99` | Beta | `@!APILevel(since: "22")` |
| `System` | `system.cj:33` | Beta | `@!APILevel(since: "22")` |
| `ResourceManager` | `resource_manager.cj:35` | Beta | `@!APILevel` |
| `AppResource` | `app_resource.cj:31` | Beta | `@!APILevel` |
| `RawFileDescriptor` | `raw_file_descriptor.cj:29` | Beta | `@!APILevel` |
| `Configuration` | `resource_manager_common.cj:34` | Beta | `@!APILevel` |
| `DeviceCapability` | `resource_manager_common.cj:122` | Beta | `@!APILevel` |

### 内部接口（不稳定）

| 接口 | 位置 | 用途 | 备注 |
|------|------|------|------|
| `__GenerateResource__()` | `resource_common.cj:202` | 资源生成器 | 前后双下划线，隐藏 API |
| `CDate` | `i18n_common.cj:176` | C 日期结构 | 内部类型 |
| 各类 `FFI*` / `CJ_*` 函数 | `*_ffi.cj` | FFI 绑定 | 仅模块内部使用 |

## 依赖方向

```
kit.LocalizationKit (聚合入口)
    │
    ├──► ohos.i18n ──────────────► [外部] i18n:cj_i18n_ffi
    │         │
    │         └──► [cj_external_deps]
    │               ├── cangjie_ark_interop:ohos.ffi
    │               ├── cangjie_ark_interop:ohos.business_exception
    │               ├── cangjie_ark_interop:ohos.labels
    │               └── hiviewdfx_cangjie_wrapper:ohos.hilog
    │
    ├──► ohos.resource_manager ──► [外部] resource_management:cj_resource_manager_ffi
    │         │
    │         ├──► ohos.raw_file_descriptor (cj_deps)
    │         ├──► ohos.resource (cj_deps)
    │         │
    │         └──► [cj_external_deps]
    │               ├── cangjie_ark_interop:ohos.ffi
    │               ├── cangjie_ark_interop:ohos.business_exception
    │               ├── cangjie_ark_interop:ohos.labels
    │               └── hiviewdfx_cangjie_wrapper:ohos.hilog
    │
    └──► ohos.resource ───────────► [外部] ace_engine:cj_frontend_ohos
              │
              └──► [cj_external_deps]
                    ├── arkui_cangjie_wrapper:ohos.base
                    ├── cangjie_ark_interop:ohos.encoding.json
                    ├── cangjie_ark_interop:ohos.ffi
                    ├── cangjie_ark_interop:ohos.labels
                    ├── cangjie_ark_interop:ohos.business_exception
                    └── hiviewdfx_cangjie_wrapper:ohos.hilog
```

**注意**: 无环依赖设计

## 生命周期管理

### RemoteDataLite 模式

`Calendar` 和 `ResourceManager` 都继承 `RemoteDataLite`：

```cangjie
public class Calendar <: RemoteDataLite {
    protected init(id: Int64) {  // Int64 是原生句柄
        super(id)
    }
    
    protected func releaseFFIData(myDataId: Int64): Unit {
        // 自动调用 FFI 释放原生资源
    }
}
```

### ResourceManager 单例模式

```cangjie
public class ResourceManager <: RemoteDataLite {
    private static let RES_MGR_MAP = HashMap<UIntNative, ResourceManager>()
    private static let APP_MUTEX = Mutex()
    
    protected static func getResourceManager(context: StageContext): ResourceManager {
        synchronized(APP_MUTEX) {
            // 双重检查锁模式
        }
    }
}
```

## 线程模型

### Worker Thread 支持

部分 API 标记为可在工作线程执行：

| API | 文件:行号 | 标注 |
|-----|----------|------|
| `getRawFd()` | `resource_manager.cj:76` | `@!APILevel[workerthread: true]` |
| `getRawFileContent()` | `resource_manager.cj:121` | `@!APILevel[workerthread: true]` |
| `getRawFileList()` | `resource_manager.cj:150` | `@!APILevel[workerthread: true]` |
| `getMediaContent()` | `resource_manager.cj:364` | `@!APILevel[workerthread: true]` |
| `getPluralStringValue()` | `resource_manager.cj:463` | `@!APILevel[workerthread: true]` |

### 日志输出

> 位置: `resource_manager_common.cj:25`

```cangjie
let RES_LOG = HilogChannel(0, 0xD001E00, "CJ-ResourceManager")
```

| 字段 | 值 |
|------|-----|
| Domain | `0xD001E00` |
| Tag | `CJ-ResourceManager` |

## 错误处理

### 错误码定义

> 位置: `resource_manager_errors.cj:24-63`

| 错误码 | 消息 | 用途 |
|--------|------|------|
| 9001001 | Invalid resource ID | 无效资源 ID |
| 9001002 | No matching resource found (ID) | 未找到匹配资源（按 ID） |
| 9001003 | Invalid resource name | 无效资源名称 |
| 9001004 | No matching resource found (name) | 未找到匹配资源（按名称） |
| 9001005 | Invalid relative path | 无效相对路径 |
| 9001006 | Resource referenced cyclically | 循环引用 |
| 9001007 | Failed to format resource (ID) | 格式化失败（按 ID） |
| 9001008 | Failed to format resource (name) | 格式化失败（按名称） |
| 9001009 | Failed to access system resource | 访问系统资源失败 |
| 9001010 | Invalid overlay path | 无效覆盖路径 |
| 890001 | Invalid parameter (i18n) | 无效参数 |

### 异常类型

- `BusinessException` - 来自 `cangjie_ark_interop:ohos.business_exception`

## 相关文档

- [API 参考](./03_NAPI_Reference.md)
- [架构设计](./02_Architecture.md)
- [构建系统](./05_Build_System.md)
