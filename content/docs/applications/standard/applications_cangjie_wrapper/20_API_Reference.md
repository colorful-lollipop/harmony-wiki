# Cangjie API 参考

本文档提供 `applications_cangjie_wrapper` 公开的完整 Cangjie API 参考，包括函数签名、参数说明、返回值、错误码和使用示例。

---

## 公开 API 清单

### 核心函数

| 函数 | 签名 | 位置 | 说明 |
|------|------|------|------|
| `getValue` | `getValue<T>(context, name, defValue): String` | `settings.cj:54` | 基础查询 |
| `getValue` | `getValue<T,P>(context, name, defValue, domainName): String` | `settings.cj:94` | 带域查询 |

### 枚举类型

| 枚举 | 说明 | 位置 |
|------|------|------|
| `DomainName` | 域名称枚举 | `settings_common.cj:30` |
| `Date` | 日期时间设置项 | `settings_common.cj:84` |
| `Display` | 显示效果设置项 | `settings_common.cj:156` |

---

## 核心 API 详解

### getValue (基础版本)

```cangjie
/**
 * Get value from settingsdata.
 *
 * @param { UIAbilityContext } context - Indicates the Context or dataAbilityHelper used to access
 * the database.
 * @param { T } name - Indicates the name of the character string.
 * @param { String } defValue - Indicates the default value of the character string.
 * @returns { String } Returns settingsdata value.
 * @throws { BusinessException } 14800000 - Parameter error.
 */
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Applications.Settings.Core",
    throwexception: true,
    workerthread: true
]
public func getValue<T>(
    context: UIAbilityContext,
    name: T,
    defValue: String
): String where T <: ToString
```

**代码位置**: `ohos/settings/settings.cj:48-75`

#### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `context` | `UIAbilityContext` | ✅ | Ability 上下文，用于访问 Settings 数据库 |
| `name` | `T <: ToString` | ✅ | 设置项名称，通常为 Date/Display 枚举值 |
| `defValue` | `String` | ✅ | 默认值，查询失败时返回此值 |

#### 返回值

| 类型 | 说明 |
|------|------|
| `String` | 设置项的值，如果查询失败则返回 `defValue` |

#### 异常

| 错误码 | 说明 | 触发条件 |
|--------|------|----------|
| `14800000` | Parameter error. | context 为空或无效 |
| `14700104` | System internal error. | 系统内存不足或死锁 |

#### 使用示例

```cangjie
import ohos.app.ability.ui_ability.UIAbilityContext
import ohos.settings.{getValue, Date, Display}

// 查询时间格式 (12小时制或24小时制)
func getTimeFormat(context: UIAbilityContext): String {
    let format = getValue(context, Date.TimeFormat, "24")
    // 返回值: "12" 或 "24"
    return format
}

// 查询屏幕亮度 (0-255)
func getBrightness(context: UIAbilityContext): Int32 {
    let brightnessStr = getValue(context, Display.ScreenBrightnessStatus, "128")
    // 将字符串解析为整数
    match (Int32.parse(brightnessStr)) {
        case Some(value) => value
        case None => 128
    }
}

// 查询自动亮度开关
func isAutoBrightness(context: UIAbilityContext): Bool {
    let auto = getValue(context, Display.AutoScreenBrightness, "0")
    return auto == "1"
}
```

---

### getValue (带域版本)

```cangjie
/**
 * Get value from settingsdata.
 *
 * @param { UIAbilityContext } context - Indicates the Context or dataAbilityHelper used to access
 * the database.
 * @param { T } name - Indicates the name of the character string.
 * @param { String } defValue - Indicates the default value of the character string.
 * @param { P } domainName - Indicates the name of the domain name to set.
 * @returns { String } Returns settingsdata value.
 * @throws { BusinessException } 14800000 - Parameter error.
 */
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Applications.Settings.Core",
    throwexception: true,
    workerthread: true
]
public func getValue<T, P>(
    context: UIAbilityContext,
    name: T,
    defValue: String,
    domainName: P
): String where T <: ToString, P <: ToString
```

**代码位置**: `ohos/settings/settings.cj:88-117`

#### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `context` | `UIAbilityContext` | ✅ | Ability 上下文 |
| `name` | `T <: ToString` | ✅ | 设置项名称 |
| `defValue` | `String` | ✅ | 默认值 |
| `domainName` | `P <: ToString` | ✅ | 域名称，通常为 DomainName 枚举值 |

#### 使用示例

```cangjie
import ohos.settings.{getValue, DomainName}

// 在设备共享域查询设置
func getDeviceSetting(context: UIAbilityContext, key: String): String {
    let value = getValue(context, key, "", DomainName.DeviceShared)
    return value
}

// 在用户属性域查询设置
func getUserSetting(context: UIAbilityContext, key: String): String {
    let value = getValue(context, key, "", DomainName.UserProperty)
    return value
}
```

---

## 枚举定义

### DomainName 枚举

```cangjie
/**
 * Provide domain name for query.
 */
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Applications.Settings.Core"
]
public enum DomainName <: ToString {
    /**
     * Provide the domain name for device shared Key.
     */
    @!APILevel[
        since: "22",
        syscap: "SystemCapability.Applications.Settings.Core"
    ]
    DeviceShared
    |
    /**
     * Provide the domain name for user property.
     */
    @!APILevel[
        since: "22",
        syscap: "SystemCapability.Applications.Settings.Core"
    ]
    UserProperty
    |
    /**
     * Provide the domain name for user security property.
     * @!Hide[isChecked: true]
     */
    UserSecurity
    |...

    public override func toString(): String {
        match (this) {
            case DeviceShared => "global"
            case UserProperty => "system"
            case _ => throw BusinessException(14800000, "Parameter error.")
        }
    }
}
```

**代码位置**: `ohos/settings/settings_common.cj:26-75`

| 枚举值 | 说明 | 内部字符串 |
|--------|------|------------|
| `DeviceShared` | 设备级共享键，所有用户共享 | `"global"` |
| `UserProperty` | 用户级属性键，每个用户独立 | `"system"` |
| `UserSecurity` | 用户安全级键 (内部隐藏) | (不支持 toString) |

---

### Date 枚举

```cangjie
/**
 * Provides methods for setting time and date formats.
 */
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Applications.Settings.Core"
]
public enum Date <: ToString {
    DateFormat      // settings.date.date_format
    | TimeFormat    // settings.date.time_format
    | AutoGainTime  // settings.date.auto_gain_time
    | AutoGainTimeZone  // settings.date.auto_gain_time_zone
    | ...

    public override func toString(): String {
        match (this) {
            case DateFormat => "settings.date.date_format"
            case TimeFormat => "settings.date.time_format"
            case AutoGainTime => "settings.date.auto_gain_time"
            case AutoGainTimeZone => "settings.date.auto_gain_time_zone"
            case _ => throw BusinessException(14800000, "Parameter error.")
        }
    }
}
```

**代码位置**: `ohos/settings/settings_common.cj:77-146`

| 枚举值 | 说明 | 键名 | 典型值 |
|--------|------|------|--------|
| `DateFormat` | 日期显示格式 | `settings.date.date_format` | `"mm/dd/yyyy"` |
| `TimeFormat` | 时间显示格式 (12/24小时制) | `settings.date.time_format` | `"12"` / `"24"` |
| `AutoGainTime` | 是否自动从 NITZ 获取时间 | `settings.date.auto_gain_time` | `"true"` / `"false"` |
| `AutoGainTimeZone` | 是否自动从 NITZ 获取时区 | `settings.date.auto_gain_time_zone` | `"true"` / `"false"` |

#### Date 使用示例

```cangjie
import ohos.settings.{getValue, Date}

// 获取日期格式
let dateFormat = getValue(context, Date.DateFormat, "yyyy/mm/dd")
// 可能的值: "mm/dd/yyyy", "dd/mm/yyyy", "yyyy/mm/dd"

// 获取时间格式
let timeFormat = getValue(context, Date.TimeFormat, "24")
// 可能的值: "12", "24"

// 检查是否自动获取时间
let autoTime = getValue(context, Date.AutoGainTime, "false")
let isAutoTime = autoTime == "true"

// 检查是否自动获取时区
let autoTimeZone = getValue(context, Date.AutoGainTimeZone, "false")
let isAutoTimeZone = autoTimeZone == "true"
```

---

### Display 枚举

```cangjie
/**
 * Provides methods for setting the display effect, including the font size, 
 * screen brightness, screen rotation, animation factor, and display color.
 */
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Applications.Settings.Core"
]
public enum Display <: ToString {
    FontScale                   // settings.display.font_scale
    | ScreenBrightnessStatus    // settings.display.screen_brightness_status
    | AutoScreenBrightness      // settings.display.auto_screen_brightness
    | AutoScreenBrightnessMode  // @!Hide
    | ManualScreenBrightnessMode // @!Hide
    | ScreenOffTimeout          // settings.display.screen_off_timeout
    | ...

    public override func toString(): String {
        match (this) {
            case FontScale => "settings.display.font_scale"
            case ScreenBrightnessStatus => "settings.display.screen_brightness_status"
            case AutoScreenBrightness => "settings.display.auto_screen_brightness"
            case ScreenOffTimeout => "settings.display.screen_off_timeout"
            case _ => throw BusinessException(14800000, "Parameter error.")
        }
    }
}
```

**代码位置**: `ohos/settings/settings_common.cj:148-225`

| 枚举值 | 可见性 | 说明 | 键名 | 典型值 |
|--------|--------|------|------|--------|
| `FontScale` | 公开 | 字体缩放因子 | `settings.display.font_scale` | `"1.0"`, `"1.25"` |
| `ScreenBrightnessStatus` | 公开 | 屏幕亮度 (0-255) | `settings.display.screen_brightness_status` | `"0"`~`"255"` |
| `AutoScreenBrightness` | 公开 | 自动亮度开关 | `settings.display.auto_screen_brightness` | `"0"` / `"1"` |
| `AutoScreenBrightnessMode` | 隐藏 | 自动亮度模式值 | (内部使用) | (内部) |
| `ManualScreenBrightnessMode` | 隐藏 | 手动亮度模式值 | (内部使用) | (内部) |
| `ScreenOffTimeout` | 公开 | 屏幕超时时间 (毫秒) | `settings.display.screen_off_timeout` | `"60000"` |

#### Display 使用示例

```cangjie
import ohos.settings.{getValue, Display}

// 获取字体缩放比例
let fontScale = getValue(context, Display.FontScale, "1.0")
let scale = Float64.parse(fontScale) ?? 1.0

// 获取屏幕亮度
let brightness = getValue(context, Display.ScreenBrightnessStatus, "128")
let brightnessValue = Int32.parse(brightness) ?? 128

// 检查是否开启自动亮度
let autoBrightness = getValue(context, Display.AutoScreenBrightness, "0")
let isAutoBrightness = autoBrightness == "1"

// 获取屏幕超时时间 (毫秒)
let timeout = getValue(context, Display.ScreenOffTimeout, "60000")
let timeoutMs = Int64.parse(timeout) ?? 60000
```

---

## API 注解元数据

### @!APILevel 注解

所有公开 API 使用 `@!APILevel` 注解标记元数据：

| 属性 | 说明 | 示例值 |
|------|------|--------|
| `since` | API 引入版本 | `"22"` |
| `syscap` | 所需系统能力 | `"SystemCapability.Applications.Settings.Core"` |
| `throwexception` | 是否抛出异常 | `true` |
| `workerthread` | 是否支持 Worker 线程 | `true` |

**代码证据**: `ohos/settings/settings.cj:48-53`

### @!Hide 注解

用于隐藏内部 API：

```cangjie
@!Hide[isChecked: true]
UserSecurity  // 隐藏枚举值
```

**代码证据**: `ohos/settings/settings_common.cj:56`

---

## 错误码定义

### 错误码列表

| 错误码 | 名称 | 说明 | 触发场景 |
|--------|------|------|----------|
| `14800000` | `PARAM_ERROR` | Parameter error. | context 为空、无效枚举匹配 |
| `14700104` | `SYSTEM_ERROR` | System internal error such as out memory or deadlock. | 系统内存不足、死锁 |

### 错误处理示例

```cangjie
import ohos.business_exception.BusinessException
import ohos.settings.{getValue, Date}

func getTimeFormatSafe(context: UIAbilityContext): String {
    try {
        return getValue(context, Date.TimeFormat, "24")
    } catch (e: BusinessException) {
        // 记录错误日志
        println("获取时间格式失败: [${e.code}] ${e.message}")
        
        // 根据错误码处理
        return when (e.code) {
            14800000 -> {
                println("参数错误，请检查 context 是否有效")
                "24"  // 使用硬编码默认值
            }
            14700104 -> {
                println("系统错误，请稍后重试")
                "24"
            }
            else -> "24"
        }
    }
}
```

---

## 类型约束

### 泛型约束

```cangjie
// 基础版本: T 必须实现 ToString
generic T where T <: ToString

// 带域版本: T 和 P 都必须实现 ToString  
generic T, P where T <: ToString, P <: ToString
```

所有枚举 (`DomainName`, `Date`, `Display`) 都实现了 `ToString` 接口，因此可以直接作为参数传入。

### 自定义类型的支持

任何实现了 `ToString` 接口的类型都可以作为 `name` 或 `domainName` 参数：

```cangjie
// 自定义设置键类型
class CustomSetting <: ToString {
    let key: String
    
    init(key: String) {
        this.key = key
    }
    
    public override func toString(): String {
        return key
    }
}

// 使用自定义类型
let custom = CustomSetting("custom.setting.key")
let value = getValue(context, custom, "default")
```

---

## 完整使用示例

### 场景：应用初始化时读取系统设置

```cangjie
package my.app

import ohos.app.ability.ui_ability.UIAbilityContext
import ohos.business_exception.BusinessException
import ohos.settings.{getValue, Date, Display, DomainName}

// 应用设置配置类
public class AppSettings {
    // 时间格式
    public var timeFormat: String = "24"
    // 日期格式
    public var dateFormat: String = "yyyy/mm/dd"
    // 屏幕亮度 (0-255)
    public var screenBrightness: Int32 = 128
    // 是否自动亮度
    public var autoBrightness: Bool = false
    // 字体缩放
    public var fontScale: Float64 = 1.0
    // 屏幕超时 (毫秒)
    public var screenOffTimeout: Int64 = 60000
    
    // 从系统加载设置
    public func loadFromSystem(context: UIAbilityContext): Unit {
        try {
            // 时间设置
            this.timeFormat = getValue(context, Date.TimeFormat, "24")
            this.dateFormat = getValue(context, Date.DateFormat, "yyyy/mm/dd")
            
            // 显示设置
            let brightnessStr = getValue(context, Display.ScreenBrightnessStatus, "128")
            this.screenBrightness = Int32.parse(brightnessStr) ?? 128
            
            let autoStr = getValue(context, Display.AutoScreenBrightness, "0")
            this.autoBrightness = (autoStr == "1")
            
            let scaleStr = getValue(context, Display.FontScale, "1.0")
            this.fontScale = Float64.parse(scaleStr) ?? 1.0
            
            let timeoutStr = getValue(context, Display.ScreenOffTimeout, "60000")
            this.screenOffTimeout = Int64.parse(timeoutStr) ?? 60000
            
        } catch (e: BusinessException) {
            // 使用默认设置
            println("加载系统设置失败: ${e.message}")
        }
    }
}

// 使用示例
main(context: UIAbilityContext) {
    let settings = AppSettings()
    settings.loadFromSystem(context)
    
    println("当前时间格式: ${settings.timeFormat}小时制")
    println("当前亮度: ${settings.screenBrightness}")
    println("自动亮度: ${settings.autoBrightness}")
}
```

---

## 注意事项

### 1. 上下文有效性

必须确保传入的 `UIAbilityContext` 有效，否则抛出 `14800000` 错误：

```cangjie
// ✅ 正确：从 Ability 获取上下文
let context = this.getUIAbilityContext()

// ❌ 错误：传递 null 或无效上下文
let context: UIAbilityContext = /* 可能为 null */
```

### 2. 字符串解析

所有返回值都是字符串，需要根据具体设置项解析为合适类型：

```cangjie
// ✅ 正确处理解析失败
let value = getValue(context, Display.ScreenBrightnessStatus, "128")
let brightness = match (Int32.parse(value)) {
    case Some(v) => v
    case None => 128  // 使用默认值
}
```

### 3. Worker 线程

虽然标记支持 `workerthread`，但设置查询通常很快，主线程调用也可接受：

```cangjie
// 主线程调用（适合单次查询）
let format = getValue(context, Date.TimeFormat, "24")

// Worker 线程调用（适合批量查询）
// 使用 @Concurrent 标记的函数
```

---

## 下一步阅读

- **[架构说明](./10_Architecture.md)** - 理解 FFI 绑定和线程模型
- **[构建系统](./30_GN_Build.md)** - 了解如何依赖本模块
- **[安全分析](./40_Security_Analysis.md)** - 了解安全使用注意事项
