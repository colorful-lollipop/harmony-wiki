# API 参考

## 概览

 telephony_cangjie_wrapper 提供仓颉语言的电话呼叫管理 API，位于 `ohos.telephony.call` 包中。

**包路径**：`ohos.telephony.call`

**Kit 入口**：`kit.TelephonyKit` (重新导出)

## 命名空间与类

```cj
// 直接导入
import ohos.telephony.call.{Call, CallState, EmergencyNumberOptions, NumberFormatOptions}

// 或通过 Kit 导入
import kit.TelephonyKit.*
```

## Call 类

提供静态方法进行呼叫管理。

### makeCall - 拨打电话

**路径**：`ohos/telephony/call/call.cj:57`

```cj
public static func makeCall(phoneNumber: String): Unit
```

| 参数 | 类型 | 说明 |
|------|------|------|
| phoneNumber | String | 要拨打的电话号码 |

**异常**：
- `BusinessException(8300001)` - 无效参数
- `BusinessException(8300002)` - 服务连接失败
- `BusinessException(8300003)` - 系统内部错误
- `BusinessException(8300999)` - 未知错误

**线程**：`workerthread`（后台线程执行）

**系统权限**：`SystemCapability.Applications.Contacts`

---

**带上下文版本**：`ohos/telephony/call/call.cj:84`

```cj
public static func makeCall(context: UIAbilityContext, phoneNumber: String): Unit
```

| 参数 | 类型 | 说明 |
|------|------|------|
| context | UIAbilityContext | 调用方上下文 |
| phoneNumber | String | 要拨打的电话号码 |

**说明**：此版本直接启动联系人应用进行拨号。

---

### getCallState - 获取通话状态

**路径**：`ohos/telephony/call/call.cj:121`

```cj
public static func getCallState(): CallState
```

**返回值**：当前通话状态（CallState 枚举）

**异常**：
- `BusinessException(8300001)` - 参数错误

---

### hasCall - 判断是否有通话

**路径**：`ohos/telephony/call/call.cj:102`

```cj
public static func hasCall(): Bool
```

**返回值**：
- `true` - 至少有一个通话不在空闲状态
- `false` - 所有通话都处于空闲状态

---

### hasVoiceCapability - 判断语音能力

**路径**：`ohos/telephony/call/call.cj:139`

```cj
public static func hasVoiceCapability(): Bool
```

**返回值**：
- `true` - 设备支持语音通话（CS 或 IMS）
- `false` - 仅支持分组交换（不支持语音）

---

### isEmergencyPhoneNumber - 判断紧急号码

**路径**：`ohos/telephony/call/call.cj:162`

```cj
public static func isEmergencyPhoneNumber(
    phoneNumber: String, 
    options!: EmergencyNumberOptions = EmergencyNumberOptions(slotId: 0)
): Bool
```

| 参数 | 类型 | 说明 |
|------|------|------|
| phoneNumber | String | 要检查的电话号码 |
| options | EmergencyNumberOptions | 可选，SIM 卡槽信息 |

**返回值**：
- `true` - 是紧急号码
- `false` - 不是紧急号码

**异常**：
- `BusinessException(8300001)` - 无效参数
- `BusinessException(8300002)` - 服务连接失败
- `BusinessException(8300003)` - 系统内部错误

**线程**：`workerthread`

---

### formatPhoneNumber - 格式化号码

**路径**：`ohos/telephony/call/call.cj:197`

```cj
public static func formatPhoneNumber(
    phoneNumber: String,
    options!: NumberFormatOptions = NumberFormatOptions()
): String
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| phoneNumber | String | - | 要格式化的电话号码 |
| options.countryCode | String | "CN" | 国家代码 |

**返回值**：格式化后的号码（标准数字字串格式），无效输入返回空字符串

**异常**：
- `BusinessException(8300001)` - 无效参数
- `BusinessException(8300002)` - 服务连接失败
- `BusinessException(8300003)` - 系统内部错误

**线程**：`workerthread`

---

### formatPhoneNumberToE164 - 格式化 E.164

**路径**：`ohos/telephony/call/call.cj:238`

```cj
public static func formatPhoneNumberToE164(phoneNumber: String, countryCode: String): String
```

| 参数 | 类型 | 说明 |
|------|------|------|
| phoneNumber | String | 要格式化的电话号码 |
| countryCode | String | ISO 3166-1 双字母国家代码 |

**返回值**：E.164 格式号码，无效输入返回空字符串

**异常**：
- `BusinessException(8300001)` - 无效参数
- `BusinessException(8300002)` - 服务连接失败
- `BusinessException(8300003)` - 系统内部错误

**线程**：`workerthread`

---

## CallState 枚举

**路径**：`ohos/telephony/call/number_format_options.cj:110`

```cj
public enum CallState {
    CallStateUnknown   // -1: 无效状态
    | CallStateIdle    //  0: 空闲，无通话
    | CallStateRinging //  1: 来电响铃或等待
    | CallStateOffhook //  2: 拨号/接通/保持中
    | CallStateAnswered//  3: 已接听
}
```

### 枚举值解析

| 值 | 常量 | 说明 |
|----|------|------|
| -1 | CallStateUnknown | 获取状态失败 |
| 0 | CallStateIdle | 无进行中的通话 |
| 1 | CallStateRinging | 有来电响铃或等待中 |
| 2 | CallStateOffhook | 至少一个通话在拨号/接通/保持状态 |
| 3 | CallStateAnswered | 通话已接听 |

---

## EmergencyNumberOptions 类

**路径**：`ohos/telephony/call/number_format_options.cj:75`

```cj
public class EmergencyNumberOptions {
    public var slotId: Int32  // SIM 卡槽索引 (0+)
    
    public init(slotId!: Int32 = 0)
}
```

| 属性 | 类型 | 说明 |
|------|------|------|
| slotId | Int32 | SIM 卡槽索引，范围 0 到设备支持的最大索引 |

---

## NumberFormatOptions 类

**路径**：`ohos/telephony/call/number_format_options.cj:180`

```cj
public class NumberFormatOptions {
    public var countryCode: String  // 国家代码
    
    public init(countryCode!: String = "CN")
}
```

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| countryCode | String | "CN" | ISO 3166-1 双字母国家代码 |

---

## 错误码参考

**路径**：`ohos/telephony/call/number_format_options.cj:29`

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 8300001 | Invalid parameter value | 无效参数值 |
| 8300002 | Operation failed. Cannot connect to service | 操作失败，无法连接服务 |
| 8300003 | System internal error | 系统内部错误 |
| 8300004 | Do not have sim card | 无 SIM 卡 |
| 8300005 | Airplane mode is on | 飞行模式已开启 |
| 8300006 | Network not in service | 网络无服务 |
| 8300999 | Unknown error code | 未知错误码 |

---

## API 清单表

| API | 命名空间 | 参数 | 返回值 | 同步/异步 | 线程 |
|-----|---------|------|--------|----------|------|
| makeCall | ohos.telephony.call | (phoneNumber: String) | Unit | 异步 | workerthread |
| makeCall (context) | ohos.telephony.call | (context, phoneNumber) | Unit | 异步 | 主线程 |
| getCallState | ohos.telephony.call | () | CallState | 同步 | 主线程 |
| hasCall | ohos.telephony.call | () | Bool | 同步 | 主线程 |
| hasVoiceCapability | ohos.telephony.call | () | Bool | 同步 | 主线程 |
| isEmergencyPhoneNumber | ohos.telephony.call | (phoneNumber, options?) | Bool | 异步 | workerthread |
| formatPhoneNumber | ohos.telephony.call | (phoneNumber, options?) | String | 异步 | workerthread |
| formatPhoneNumberToE164 | ohos.telephony.call | (phoneNumber, countryCode) | String | 异步 | workerthread |
