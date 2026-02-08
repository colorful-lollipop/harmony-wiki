# N-API 接口参考

> global_cangjie_wrapper 对外 Cangjie API 完整清单

## 概览

### 模块列表

| 模块 | 包路径 | 主要类 | API 数量 |
|------|--------|--------|----------|
| i18n | `@ohos.global.i18n` | Calendar, System | 17+ |
| resource | `@ohos.global.resource` | AppResource | 5+ |
| resource_manager | `@ohos.global.resource_manager` | ResourceManager, Configuration, DeviceCapability | 30+ |
| raw_file_descriptor | `@ohos.global.raw_file_descriptor` | RawFileDescriptor | 3 |

## i18n 模块

### Calendar 类

> 位置: `ohos/i18n/calendar.cj:99-340`

```cangjie
@!APILevel([
    since: "22",
    syscap: "SystemCapability.Global.I18n"
])
public class Calendar <: RemoteDataLite {
    // 工厂函数
    public static func getCalendar(locale: CString, calendarType: CalendarType): Calendar
    
    // 实例方法
    public func setTime(time: Float64): Unit
    public func setTimeZone(timeZone: CString): Unit
    public func getTimeZone(): CString
    public func getFirstDayOfWeek(): Int32
    public func setFirstDayOfWeek(firstDayOfWeek: Int32): Unit
    public func getMinimalDaysInFirstWeek(): Int32
    public func setMinimalDaysInFirstWeek(minimalDays: Int32): Unit
    public func get(field: CalendarField): Int32
    public func set(field: CalendarField, value: Int32): Unit
    public func getDisplayName(calendarType: CalendarType, shortStyle: Bool): CString
    public func add(field: CalendarField, amount: Int32): Unit
    public func getTimeInMillis(): Float64
}
```

#### Calendar API 清单

| API | 参数 | 返回值 | 描述 |
|-----|------|--------|------|
| `getCalendar()` | locale: CString, type: CalendarType | Calendar | 创建日历实例 |
| `setTime()` | time: Float64 | Unit | 设置时间（毫秒） |
| `setTimeZone()` | timeZone: CString | Unit | 设置时区 |
| `getTimeZone()` | - | CString | 获取时区 |
| `getFirstDayOfWeek()` | - | Int32 | 获取每周首日 (1-7) |
| `setFirstDayOfWeek()` | firstDayOfWeek: Int32 | Unit | 设置每周首日 |
| `getMinimalDaysInFirstWeek()` | - | Int32 | 获取首周最少天数 |
| `setMinimalDaysInFirstWeek()` | minimalDays: Int32 | Unit | 设置首周最少天数 |
| `get()` | field: CalendarField | Int32 | 获取字段值 |
| `set()` | field: CalendarField, value: Int32 | Unit | 设置字段值 |
| `getDisplayName()` | type: CalendarType, shortStyle: Bool | CString | 获取显示名称 |
| `add()` | field: CalendarField, amount: Int32 | Unit | 增加字段值 |
| `getTimeInMillis()` | - | Float64 | 获取时间（毫秒） |

#### CalendarType 枚举

> 位置: `ohos/i18n/i18n_common.cj:46`

| 值 | 描述 |
|-----|------|
| `Buddhist` | 佛历 |
| `Chinese` | 中国农历 |
| `Coptic` | 科普特历 |
| `Ethiopic` | 埃塞俄比亚历 |
| `Hebrew` | 希伯来历 |
| `Gregory` | 公历（格里高利历） |
| `Indian` | 印度历 |
| `IslamicCivil` | 伊斯兰民事历 |
| `IslamicTbla` | 伊斯兰表格历 |
| `IslamicUmalqura` | 伊斯兰乌姆库拉历 |
| `Japanese` | 日本历 |
| `Persian` | 波斯历 |

### System 类

> 位置: `ohos/i18n/system.cj:33-54`

```cangjie
@!APILevel([
    since: "22",
    syscap: "SystemCapability.Global.I18n"
])
public class System {
    public static func getAppPreferredLanguage(): CString
}
```

#### System API 清单

| API | 参数 | 返回值 | 描述 |
|-----|------|--------|------|
| `getAppPreferredLanguage()` | - | CString | 获取应用首选语言标签 (如 "zh-CN") |

## resource 模块

### AppResource 类

> 位置: `ohos/resource/app_resource.cj:31-108`

```cangjie
@!APILevel([
    since: "22",
    syscap: "SystemCapability.Global.I18n"
])
public class AppResource <: Length & ResourceColor & ResourceStr {
    public let bundleName: String
    public let moduleName: String
    public let id: Int64
    public let params: (String, String)?
    public let resType: ResourceType
}
```

#### AppResource API 清单

| 属性/方法 | 类型 | 描述 |
|-----------|------|------|
| `bundleName` | String | 资源所属包名 |
| `moduleName` | String | 资源所属模块名 |
| `id` | Int64 | 资源 ID |
| `params` | (String, String)? | 资源参数 |
| `resType` | ResourceType | 资源类型 |

#### 实现接口

| 接口 | 用途 |
|------|------|
| `Length` | 尺寸资源 |
| `ResourceColor` | 颜色资源 |
| `ResourceStr` | 字符串资源 |

## resource_manager 模块

### ResourceManager 类

> 位置: `ohos/resource_manager/resource_manager.cj:35-695`

```cangjie
@!APILevel([
    since: "22",
    syscap: "SystemCapability.Global.I18n"
])
public class ResourceManager {
    // 静态工厂
    public static func getResourceManager(bundleName: String, moduleName: String): ResourceManager
    
    // 字符串资源
    public func getString(resId: UInt32): CString
    public func getStringByName(resourceName: CString): CString
    public func getPluralStringValue(resId: UInt32, quantity: Int32): CString
    public func getPluralStringByName(resourceName: CString, quantity: Int32): CString
    
    // 颜色资源
    public func getColor(resId: UInt32): UInt32
    public func getColorByName(resourceName: CString): UInt32
    
    // 布尔值资源
    public func getBoolean(resId: UInt32): Bool
    public func getBooleanByName(resourceName: CString): Bool
    
    // 数值资源
    public func getNumber(resId: UInt32): Float64
    public func getNumberByName(resourceName: CString): Float64
    
    // 媒体资源
    public func getMediaContent(resId: UInt32): Uint8Array
    public func getMediaByName(resourceName: CString): Uint8Array
    public func getMediaContentBase64(resId: UInt32): CString
    public func getMediaBase64ByName(resourceName: CString): CString
    
    // 字符串数组
    public func getStringArrayValue(resId: UInt32): CStringArray
    public func getStringArrayByName(resourceName: CString): CStringArray
    
    // 原始文件
    public func getRawFd(rawFileName: CString): RawFileDescriptor
    public func closeRawFd(fd: RawFileDescriptor): Unit
    public func getRawFileContent(rawFileName: CString): Uint8Array
    public func getRawFileList(rawFileName: CString): CStringArray
    
    // 配置与能力
    public func getConfiguration(): Configuration
    public func getDeviceCapability(): DeviceCapability
    
    // 动态资源
    public func addResource(path: CString): Bool
    public func removeResource(path: CString): Bool
    
    // 区域
    public func getLocales(isIncludeSystemApps: Bool): CStringArray
}
```

#### ResourceManager API 清单（分类）

| 分类 | API | 描述 |
|------|-----|------|
| **工厂** | `getResourceManager()` | 创建 ResourceManager 实例 |
| **字符串** | `getString()`, `getStringByName()` | 获取字符串资源 |
| | `getPluralStringValue()`, `getPluralStringByName()` | 获取复数字符串 |
| **颜色** | `getColor()`, `getColorByName()` | 获取颜色值 |
| **布尔** | `getBoolean()`, `getBooleanByName()` | 获取布尔值 |
| **数值** | `getNumber()`, `getNumberByName()` | 获取数值 |
| **媒体** | `getMediaContent()`, `getMediaByName()` | 获取媒体内容 |
| | `getMediaContentBase64()`, `getMediaBase64ByName()` | 获取 Base64 编码媒体 |
| **字符串数组** | `getStringArrayValue()`, `getStringArrayByName()` | 获取字符串数组 |
| **原始文件** | `getRawFd()`, `closeRawFd()` | 获取/关闭原始文件描述符 |
| | `getRawFileContent()`, `getRawFileList()` | 获取原始文件内容 |
| **配置** | `getConfiguration()` | 获取设备配置 |
| | `getDeviceCapability()` | 获取设备能力 |
| **动态资源** | `addResource()`, `removeResource()` | 添加/移除资源路径 |
| **区域** | `getLocales()` | 获取支持区域列表 |

### Configuration 类

> 位置: `ohos/resource_manager/resource_manager_common.cj:34-119`

```cangjie
@!APILevel([
    since: "22",
    syscap: "SystemCapability.Global.I18n"
])
public class Configuration {
    public init()
    public let direction: Direction
    public let locale: String
    public let deviceType: DeviceType
    public let screenDensity: ScreenDensity
    public let colorMode: ColorMode
    public let mcc: Int32
    public let mnc: Int32
}
```

#### Configuration 属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `direction` | Direction | 屏幕方向 |
| `locale` | String | 区域设置 |
| `deviceType` | DeviceType | 设备类型 |
| `screenDensity` | ScreenDensity | 屏幕密度 |
| `colorMode` | ColorMode | 颜色模式 |
| `mcc` | Int32 | 移动国家代码 |
| `mnc` | Int32 | 移动网络代码 |

### DeviceCapability 类

> 位置: `ohos/resource_manager/resource_manager_common.cj:122-157`

```cangjie
@!APILevel([
    since: "22",
    syscap: "SystemCapability.Global.I18n"
])
public class DeviceCapability {
    public init()
    public let screenDensity: ScreenDensity
    public let deviceType: DeviceType
}
```

### 枚举类型

| 枚举 | 值 | 描述 |
|------|-----|------|
| **ColorMode** | `COLOR_MODE_NOT_SET`, `DARK`, `LIGHT` | 颜色模式 |
| **ScreenDensity** | `SCREEN_DENSITY_NOT_SET`, `SD_120`, `SD_160`, `SD_240`, `SD_320`, `SD_480` | 屏幕密度 |
| **DeviceType** | `DEVICE_NOT_SET`, `PHONE`, `TABLET`, `CAR`, `TV`, `WATCH`, `VEHICLE` | 设备类型 |
| **Direction** | `DIRECTION_NOT_SET`, `HORIZONTAL`, `VERTICAL` | 屏幕方向 |

## raw_file_descriptor 模块

### RawFileDescriptor 类

> 位置: `ohos/raw_file_descriptor/raw_file_descriptor.cj:29-69`

```cangjie
@!APILevel([
    since: "22",
    syscap: "SystemCapability.Global.I18n"
])
public class RawFileDescriptor {
    public init(fd: Int32, offset: Int64, length: Int64)
    public let fd: Int32
    public let offset: Int64
    public let length: Int64
}
```

#### RawFileDescriptor API 清单

| 属性/方法 | 类型 | 描述 |
|-----------|------|------|
| `fd` | Int32 | 文件描述符 |
| `offset` | Int64 | 文件偏移量 |
| `length` | Int64 | 数据长度 |
| `init()` | 构造函数 | 创建实例 |

## 错误码参考

> 位置: `ohos/resource_manager/resource_manager_errors.cj`

| 错误码 | 描述 |
|--------|------|
| `ERROR_OK` | 成功 |
| `ERROR_INTERNAL` | 内部错误 |
| `ERROR_RES_NUM` | 资源数量超出限制 |
| `ERROR_RES_TYPE` | 资源类型不匹配 |
| `ERROR_RES_NAME` | 资源名称无效 |
| `ERROR_RES_ID` | 资源 ID 无效 |
| `ERROR_RES_PATH` | 资源路径无效 |
| `ERROR_RES_FILE` | 文件操作错误 |

## 使用示例

### Calendar 使用示例

```cangjie
import { Calendar, CalendarType } from '@ohos.global.i18n'

// 创建日历
let cal = Calendar.getCalendar("zh-CN", CalendarType.Gregory)

// 设置时间
cal.setTime(Date.now())

// 获取时区
let tz = cal.getTimeZone()
console.log("Timezone:", tz)

// 获取时间
let millis = cal.getTimeInMillis()
```

### ResourceManager 使用示例

```cangjie
import { ResourceManager } from '@ohos.global.resource_manager'

// 获取 ResourceManager
let rm = ResourceManager.getResourceManager("com.example.app", "entry")

// 获取字符串
let str = rm.getString(0x1000000)

// 获取颜色
let color = rm.getColor(0x1000001)

// 获取媒体
let media = rm.getMediaContent(0x1000002)

// 打开原始文件
let fd = rm.getRawFd("test.raw")
// 使用 fd...
rm.closeRawFd(fd)
```

## 相关文档

- [概览](./01_Overview.md)
- [架构设计](./02_Architecture.md)
- [内部 API](./04_Inner_API.md)
