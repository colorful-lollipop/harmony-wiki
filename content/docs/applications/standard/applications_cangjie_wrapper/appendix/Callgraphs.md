# 附录：关键调用链

本文档提供 `applications_cangjie_wrapper` 中关键功能的完整调用链追踪，帮助理解代码执行路径和调试问题。

---

## 调用链索引

| 功能 | 调用链 | 页内链接 |
|------|--------|----------|
| 基础设置查询 | `getValue<T>(context, name, defValue)` | [基础查询链](#1-基础设置查询-getvaluet) |
| 带域设置查询 | `getValue<T,P>(context, name, defValue, domainName)` | [带域查询链](#2-带域设置查询-getvaluetp) |
| 枚举值转换 | `DomainName.toString()` / `Date.toString()` / `Display.toString()` | [枚举转换链](#3-枚举值转换链) |
| 错误处理 | 异常抛出与处理 | [错误处理链](#4-错误处理链) |

---

## 1. 基础设置查询 (getValue<T>)

### 调用时序

```
调用入口: getValue<T>(context, name, defValue)
    │
    ├──> [Cangjie] ohos/settings/settings.cj:54
    │       │
    │       ├──> 参数获取
    │       │       └──> getStageContext(context) [ohos/settings/settings.cj:55]
    │       │               └──> ability_cangjie_wrapper 调用
    │       │
    │       ├──> 参数校验
    │       │       └──> stageContext.isNull() 检查 [ohos/settings/settings.cj:56]
    │       │               └──> 为空 → throw BusinessException(14800000)
    │       │
    │       ├──> 字符串转换
    │       │       ├──> name.toString() → CString
    │       │       └──> defValue → CString
    │       │               └──> LibC.mallocCString() [cangjie_ark_interop]
    │       │
    │       ├──> FFI 调用
    │       │       └──> FfiSettingsGetValue(...) [ohos/settings/settings_ffi.cj:23]
    │       │               └──> 进入 Native 层
    │       │                       └──> settings:cj_settings_ffi (C++)
    │       │                               └──> SettingsProvider 查询
    │       │
    │       ├──> 结果处理
    │       │       ├──> result.isNull() → 抛异常 [ohos/settings/settings.cj:67]
    │       │       ├──> result.toString() → Cangjie String
    │       │       └──> LibC.free(result) [ohos/settings/settings.cj:71]
    │       │
    │       └──> 返回结果值
    │
    └──> 返回给调用者
```

### 代码路径

```cangjie
// ohos/settings/settings.cj:48-75
@!APILevel[...]
public func getValue<T>(context: UIAbilityContext, name: T, defValue: String): String where T <: ToString {
    // 1. 获取 StageContext
    let stageContext = getStageContext(context)              // [55]
    
    // 2. 验证上下文
    if (stageContext.isNull()) {                             // [56]
        throw BusinessException(14800000, "Parameter error.") // [57]
    }
    
    // 3. 准备 FFI 调用
    var ret: Int32 = 0
    var value: String = ""
    
    unsafe {
        // 4. 分配 C 字符串资源
        try (
            cName = LibC.mallocCString(name.toString()).asResource(),      // [63]
            cDefValue = LibC.mallocCString(defValue).asResource()          // [64]
        ) {
            // 5. FFI 调用
            let result = FfiSettingsGetValue(stageContext, cName.value, 
                                             cDefValue.value, CString(CPointer()), 
                                             inout ret)                     // [66]
            
            // 6. 处理结果
            if (result.isNull()) {                                        // [67]
                throw BusinessException(ret, getErrorMsg(ret))            // [68]
            }
            value = result.toString()                                     // [70]
            LibC.free(result)                                             // [71]
        }
    }
    value                                                                 // [74]
}
```

---

## 2. 带域设置查询 (getValue<T,P>)

### 调用时序

```
调用入口: getValue<T,P>(context, name, defValue, domainName)
    │
    ├──> [Cangjie] ohos/settings/settings.cj:94
    │       │
    │       ├──> 参数获取与校验
    │       │       ├──> getStageContext(context)
    │       │       └──> 空检查 → 可能抛 14800000
    │       │
    │       ├──> 域名转换
    │       │       └──> domainName.toString() → "global" / "system"
    │       │               └──> DomainName.toString() [settings_common.cj:68]
    │       │
    │       ├──> 字符串分配
    │       │       ├──> name.toString() → CString
    │       │       ├──> defValue → CString
    │       │       └──> domainName.toString() → CString
    │       │
    │       ├──> FFI 调用
    │       │       └──> FfiSettingsGetValue(context, name, defValue, 
    │       │                              domainName, ret)
    │       │
    │       └──> 结果处理（同基础查询）
    │
    └──> 返回结果值
```

### 关键区别

带域版本与基础版本的主要区别：

```cangjie
// 基础版本 (settings.cj:66)
let result = FfiSettingsGetValue(stageContext, cName.value, cDefValue.value, 
                                 CString(CPointer()),  // ← 空指针
                                 inout ret)

// 带域版本 (settings.cj:108)
let result = FfiSettingsGetValue(stageContext, cName.value, cDefValue.value,
                                 cDomainName.value,    // ← 域名字符串
                                 inout ret)
```

### 域名字符串映射

```cangjie
// ohos/settings/settings_common.cj:68-74
public override func toString(): String {
    match (this) {
        case DeviceShared => "global"   // 设备级共享
        case UserProperty => "system"   // 用户级属性
        case _ => throw BusinessException(14800000, "Parameter error.")
    }
}
```

---

## 3. 枚举值转换链

### Date 枚举转换

```
Date.TimeFormat
    │
    ├──> toString() [settings_common.cj:137]
    │       │
    │       ├──> match (this)
    │       │       case TimeFormat
    │       │
    │       └──> 返回 "settings.date.time_format"
    │
    └──> 作为 name 参数传入 getValue()
```

### Display 枚举转换

```
Display.ScreenBrightnessStatus
    │
    ├──> toString() [settings_common.cj:216]
    │       │
    │       ├──> match (this)
    │       │       case ScreenBrightnessStatus
    │       │
    │       └──> 返回 "settings.display.screen_brightness_status"
    │
    └──> 作为 name 参数传入 getValue()
```

### 枚举完整映射表

| 枚举类型 | 枚举值 | toString() 返回值 | 设置键 |
|----------|--------|-------------------|--------|
| **Date** | DateFormat | `"settings.date.date_format"` | 日期格式 |
| | TimeFormat | `"settings.date.time_format"` | 时间格式 |
| | AutoGainTime | `"settings.date.auto_gain_time"` | 自动获取时间 |
| | AutoGainTimeZone | `"settings.date.auto_gain_time_zone"` | 自动获取时区 |
| **Display** | FontScale | `"settings.display.font_scale"` | 字体缩放 |
| | ScreenBrightnessStatus | `"settings.display.screen_brightness_status"` | 屏幕亮度 |
| | AutoScreenBrightness | `"settings.display.auto_screen_brightness"` | 自动亮度 |
| | ScreenOffTimeout | `"settings.display.screen_off_timeout"` | 屏幕超时 |
| **DomainName** | DeviceShared | `"global"` | 设备共享域 |
| | UserProperty | `"system"` | 用户属性域 |

---

## 4. 错误处理链

### 异常抛出路径

```
错误发生
    │
    ├──> 参数错误 (context 为空)
    │       │
    │       ├──> [settings.cj:57]
    │       │       throw BusinessException(14800000, "Parameter error.")
    │       │
    │       └──> 调用者捕获
    │               catch (e: BusinessException)
    │
    ├──> FFI 调用失败
    │       │
    │       ├──> [settings.cj:66-67]
    │       │       FfiSettingsGetValue(...) → result = null
    │       │
    │       ├──> [settings.cj:67-68]
    │       │       throw BusinessException(ret, getErrorMsg(ret))
    │       │
    │       └──> 调用者捕获
    │
    └──> 枚举值无效
            │
            ├──> [settings_common.cj:72/143/222]
            │       case _ => throw BusinessException(14800000, "Parameter error.")
            │
            └──> 调用者捕获
```

### 错误码映射链

```
Native 错误码 (ret 值)
    │
    ├──> [settings.cj:68] BusinessException(ret, getErrorMsg(ret))
    │
    └──> getErrorMsg(code) [settings.cj:26-36]
            │
            ├──> getUniversalErrorMsg(code) [cangjie_ark_interop]
            │       └──> 返回通用错误信息
            │
            ├──> ERROR_CODE_MAP[code]
            │       └──> 14700104 → "System internal error..."
            │
            └──> 未知错误码
                    └──> "Unknown error code ${code}"
```

---

## 5. Mock 实现调用链

### Mock 路径 (Windows/macOS)

```
构建系统检测
    │
    ├──> [ohos/settings/BUILD.gn:20]
    │       if (is_mingw || is_mac)
    │
    └──> 使用 Mock 源文件
            │
            ├──> sources = ["../../mock/ohos.settings.cj"]
            │
            └──> 调用 mock/ohos.settings.cj 中的实现
                    │
                    ├──> getValue<T>() → 返回空字符串 String()
                    │
                    └──> getValue<T,P>() → 返回空字符串 String()
```

### Mock 与真实实现对比

| 方面 | 真实实现 | Mock 实现 |
|------|----------|-----------|
| 源文件 | `ohos/settings/*.cj` | `mock/ohos.settings.cj` |
| FFI 调用 | ✅ 真实调用 | ❌ 无 |
| 返回值 | 系统设置值 | 空字符串 "" |
| 异常 | 可能抛出 | 永不抛出 |
| 使用场景 | OpenHarmony 设备 | 交叉编译/开发环境 |

---

## 6. 日志记录链

### 日志配置

```
settings_log.cj 初始化
    │
    ├──> LOG_CORE = 0
    ├──> SETTINGS_DOMAIN_ID = 0x500
    └──> SETTINGS_LOG = HilogChannel(LOG_CORE, SETTINGS_DOMAIN_ID, "CJ-Settings")
               │
               └──> hiviewdfx_cangjie_wrapper:ohos.hilog
```

### 当前使用情况

当前源码已配置日志但未在关键路径打印日志，预留用于：
- 调试信息输出
- 错误追踪
- 性能分析

---

## 7. 完整端到端调用示例

### 场景：查询屏幕亮度

```cangjie
// 应用代码
import ohos.settings.{getValue, Display}

let brightness = getValue(context, Display.ScreenBrightnessStatus, "128")
```

### 完整调用链

```
1. 应用层
   Display.ScreenBrightnessStatus
       │
       └──> toString() → "settings.display.screen_brightness_status"

2. getValue 调用
   getValue(context, "settings.display.screen_brightness_status", "128")
       │
       ├──> getStageContext(context) → stageContext
       ├──> isNull() 检查 → 通过
       │
       ├──> CString 分配
       │       ├──> "settings.display.screen_brightness_status"
       │       └──> "128"
       │
       ├──> FfiSettingsGetValue(stageContext, 
       │                        "settings.display.screen_brightness_status",
       │                        "128", null, &ret)
       │       │
       │       └──> [Native] 查询 SettingsProvider
       │               └──> SELECT value FROM settings 
       │                      WHERE key='settings.display.screen_brightness_status'
       │
       ├──> 返回 "200" (示例值)
       │
       ├──> toString() → Cangjie String "200"
       ├──> LibC.free(result)
       │
       └──> 返回 "200"

3. 应用层接收
   brightness = "200"
```

---

## 调试技巧

### 关键断点位置

| 功能 | 文件 | 行号 | 说明 |
|------|------|------|------|
| 入口检查 | settings.cj | 55 | context 获取 |
| 参数校验 | settings.cj | 56-58 | context 空检查 |
| FFI 调用前 | settings.cj | 66 | 查看 CString 值 |
| FFI 调用后 | settings.cj | 67 | 检查 result 值 |
| 结果处理 | settings.cj | 70-71 | 字符串转换与释放 |

### 常见问题定位

| 问题现象 | 检查点 | 可能原因 |
|----------|--------|----------|
| 抛出 14800000 | settings.cj:57 | context 为空或无效 |
| 抛出 14700104 | settings.cj:68 | Native 层错误 |
| 返回空字符串 | settings.cj:70 | 使用了 Mock 实现 |
| 崩溃 | settings.cj:71 | result 释放问题 |

---

## 相关文档

- **[API 参考](./../20_API_Reference.md)** - API 详细文档
- **[架构说明](./../10_Architecture.md)** - 架构设计说明
- **[安全分析](./../40_Security_Analysis.md)** - 安全风险分析
