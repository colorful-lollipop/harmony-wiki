# 关键调用链

## 概述

本文档梳理 telephony_cangjie_wrapper 的关键调用路径，包括：
- 仓颉层 → FFI → 原生层的完整调用链
- 错误处理与异常传播路径

## 调用链索引

| 调用场景 | 入口 API | 章节 |
|---------|---------|------|
| 拨打电话 | `Call.makeCall()` | [§1](#1-makecall-拨打电话) |
| 带上下文拨号 | `Call.makeCall(context, phoneNumber)` | [§2](#2-makecallcontext-phoneNumber-带上下文拨号) |
| 获取通话状态 | `Call.getCallState()` | [§3](#3-getcallstate-获取通话状态) |
| 判断紧急号码 | `Call.isEmergencyPhoneNumber()` | [§4](#4-isemergencyphonenumber-判断紧急号码) |
| 格式化号码 | `Call.formatPhoneNumber()` | [§5](#5-formatphonenumber-格式化号码) |

---

## 1. makeCall - 拨打电话

### 调用链

```
仓颉应用
    │
    ▼ Call.makeCall("13800138000")
    
Call (call.cj:57)
    │
    ├─► LibC.mallocCString(phoneNumber)
    ├─► FfiOHOSTelephonyCallMakeCall(cString)
    │
    ▼
call_manager:cj_telephony_call_ffi (C++)
    │
    ▼
call_manager 核心服务
    │
    ▼
系统拨号界面 (跳转到 Contacts 应用)
```

### 代码路径

| 层级 | 文件 | 函数/操作 |
|------|------|----------|
| 仓颉 | `call.cj:57` | `makeCall(phoneNumber)` |
| FFI | `telephony_call_ffi.cj:36` | `FfiOHOSTelephonyCallMakeCall()` |
| 原生 | call_manager | `make_call()` |

### 错误处理

```cj
// call.cj:59-66
try (cNumber = LibC.mallocCString(phoneNumber).asResource()) {
    let errCode = FfiOHOSTelephonyCallMakeCall(cNumber.value)
    if (errCode != SUCCESS_CODE) {
        throw BusinessException(getErrorCode(errCode), getErrorMsg(errCode))
    }
}
```

---

## 2. makeCall(context, phoneNumber) - 带上下文拨号

### 调用链

```
仓颉应用
    │
    ▼ Call.makeCall(context, "13800138000")
    
Call (call.cj:84)
    │
    ├─► HashMap<String, WantValueType>()
    ├─► Want(bundleName: "com.ohos.contacts", ...)
    │
    ▼
ability_cangjie_wrapper
    │
    ▼ context.startAbility(Want)
    
系统拨号界面
```

### 特点

- **线程**：主线程执行
- **路径**：直接启动 Contacts 应用，不经过 call_manager
- **参数**：使用 Want 机制传递

### 代码路径

| 层级 | 文件 | 函数/操作 |
|------|------|----------|
| 仓颉 | `call.cj:84-91` | `makeCall(context, phoneNumber)` |
| FFI | N/A | 无 FFI 调用 |
| 原生 | ability_cangjie_wrapper | `startAbility()` |

---

## 3. getCallState - 获取通话状态

### 调用链

```
仓颉应用
    │
    ▼ Call.getCallState()
    
Call (call.cj:121)
    │
    ├─► FfiOHOSTelephonyCallGetCallState()
    │
    ▼
call_manager:cj_telephony_call_ffi (C++)
    │
    ▼
call_manager 核心服务
    │
    ▼ 返回 Int32 (-1~3)
    
CallState.parse(enumCode)
    │
    ▼ 返回 CallState 枚举
```

### 枚举映射

```cj
// number_format_options.cj:161-170
protected static func parse(val: Int32): CallState {
    match (val) {
        case -1 => CallStateUnknown
        case 0 => CallStateIdle
        case 1 => CallStateRinging
        case 2 => CallStateOffhook
        case 3 => CallStateAnswered
        case _ => throw BusinessException(8300001, "Parameter error.")
    }
}
```

### 代码路径

| 层级 | 文件 | 函数/操作 |
|------|------|----------|
| 仓颉 | `call.cj:121` | `getCallState()` |
| FFI | `telephony_call_ffi.cj:30` | `FfiOHOSTelephonyCallGetCallState()` |
| 仓颉 | `number_format_options.cj:161` | `CallState.parse()` |

---

## 4. isEmergencyPhoneNumber - 判断紧急号码

### 调用链

```
仓颉应用
    │
    ▼ Call.isEmergencyPhoneNumber("110", options)
    
Call (call.cj:162)
    │
    ├─► LibC.mallocCString(phoneNumber)
    ├─► FfiOHOSTelephonyCallIsEmergencyPhoneNumber(..., slotId, errCode)
    │
    ▼
call_manager:cj_telephony_call_ffi (C++)
    │
    ▼
紧急号码查询服务
    │
    ▼ 返回 Bool + errCode
```

### 特点

- **线程**：workerthread 执行
- **参数**：包含 slotId 用于多卡场景
- **错误**：通过 errCode 输出参数返回

### 代码路径

| 层级 | 文件 | 函数/操作 |
|------|------|----------|
| 仓颉 | `call.cj:162` | `isEmergencyPhoneNumber()` |
| FFI | `telephony_call_ffi.cj:34` | `FfiOHOSTelephonyCallIsEmergencyPhoneNumber()` |

---

## 5. formatPhoneNumber - 格式化号码

### 调用链

```
仓颉应用
    │
    ▼ Call.formatPhoneNumber("13800138000", options)
    
Call (call.cj:197)
    │
    ├─► LibC.mallocCString(phoneNumber)
    ├─► LibC.mallocCString(countryCode)
    ├─► FfiOHOSTelephonyCallFormatPhoneNumber(..., errCode)
    │
    ▼
call_manager:cj_telephony_call_ffi (C++)
    │
    ▼
号码格式化服务
    │
    ▼ 返回 CString + errCode
    
LibC.free(formatNumber)  // 释放返回值
```

### 特点

- **线程**：workerthread 执行
- **内存**：需要手动释放 FFI 返回的 CString
- **错误**：通过 errCode 输出参数返回

### 代码路径

| 层级 | 文件 | 函数/操作 |
|------|------|----------|
| 仓颉 | `call.cj:197` | `formatPhoneNumber()` |
| FFI | `telephony_call_ffi.cj:28` | `FfiOHOSTelephonyCallFormatPhoneNumber()` |

---

## 6. hasVoiceCapability - 判断语音能力

### 调用链

```
仓颉应用
    │
    ▼ Call.hasVoiceCapability()
    
Call (call.cj:139)
    │
    └─► FfiOHOSTelephonyCallHasVoiceCapability()
    
call_manager:cj_telephony_call_ffi (C++)
    │
    ▼
设备能力查询
    │
    ▼ 返回 Bool
```

### 代码路径

| 层级 | 文件 | 函数/操作 |
|------|------|----------|
| 仓颉 | `call.cj:139` | `hasVoiceCapability()` |
| FFI | `telephony_call_ffi.cj:26` | `FfiOHOSTelephonyCallHasVoiceCapability()` |

---

## 依赖关系总图

```
┌──────────────────────────────────────────────────────────────┐
│                      仓颉应用层                               │
│  kit.TelephonyKit / ohos.telephony.call                      │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼ (import)
┌──────────────────────────────────────────────────────────────┐
│                      仓颉实现层                               │
│  call.cj              - API 实现                             │
│  number_format_options.cj - 数据类型/错误码                   │
│  telephony_call_ffi.cj - FFI 声明                            │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼ (external_deps)
┌──────────────────────────────────────────────────────────────┐
│                      FFI 边界                                 │
│  call_manager:cj_telephony_call_ffi                          │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                      原生服务层                               │
│  telephony_call_manager                                      │
│    ├─ 通话状态管理                                           │
│    ├─ 拨号服务                                               │
│    ├─ 紧急号码查询                                           │
│    └─ 号码格式化                                             │
└──────────────────────────────────────────────────────────────┘
```

---

## 外部依赖调用关系

| 依赖模块 | 调用方式 | 调用场景 |
|---------|---------|---------|
| ability_cangjie_wrapper | cj_external_deps | makeCall(context, ...) |
| hiviewdfx_cangjie_wrapper | cj_external_deps | 所有 API (日志) |
| cangjie_ark_interop | cj_external_deps | BusinessException, FFI 类型 |
| call_manager | external_deps | 所有核心 API |
