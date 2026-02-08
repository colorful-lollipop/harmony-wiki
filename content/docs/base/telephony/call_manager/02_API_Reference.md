# API 参考

**目的**: 详细描述 N-API 绑定、JS API 接口、参数校验和错误码

---

## N-API 入口

### 注册点

| 项目 | 值 | 证据 |
|------|-----|------|
| 注册文件 | `frameworks/js/napi/src/native_module.cpp` | 行 38 |
| 注册函数 | `NapiCallManager::RegisterCallManagerFunc` | 行 33 |
| 模块名 | `telephony.call` | 行 34 |
| N-API 版本 | 1 | 行 30 |

### N-API 结构

```cpp
// native_module.cpp:27-38
extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module module = {
        .nm_version = 1,
        .nm_flags = 0,
        .nm_filename = nullptr,
        .nm_register_func = NapiCallManager::RegisterCallManagerFunc,
        .nm_modname = "telephony.call",
        .nm_priv = nullptr,
    };
    napi_module_register(&module);
}
```

---

## JS API 清单

### 命名空间

所有 API 位于 `call` 命名空间下：

```typescript
import call from "@ohos.telephony.call";
```

### API 列表

#### 通话操作类

| API | 功能 | 权限 | 同步/异步 |
|-----|------|------|-----------|
| `dial()` | 拨打电话（已废弃） | PLACE_CALL | Async |
| `dialCall()` | 拨打电话 | PLACE_CALL | Async |
| `makeCall()` | 打开拨号盘 | 无 | Async |
| `answerCall()` | 接听来电 | ANSWER_CALL | Async |
| `hangUpCall()` | 挂断通话 | ANSWER_CALL | Async |
| `rejectCall()` | 拒绝来电 | ANSWER_CALL | Async |

**证据**: `interfaces/kits/js/@ohos.telephony.call.d.ts:110, 182, 494, 570, 627`

#### 通话控制类

| API | 功能 | 权限 | 同步/异步 |
|-----|------|------|-----------|
| `holdCall()` | 保持通话 | ANSWER_CALL | Async |
| `unHoldCall()` | 取消保持 | ANSWER_CALL | Async |
| `switchCall()` | 切换通话 | ANSWER_CALL | Async |
| `combineConference()` | 合并通话 | 无 | Async |

**证据**: `@ohos.telephony.call.d.ts:722-853`

#### 状态查询类

| API | 功能 | 权限 | 同步/异步 |
|-----|------|------|-----------|
| `hasCall()` | 是否有通话 | 无 | Async |
| `hasCallSync()` | 是否有通话（同步） | 无 | Sync |
| `getCallState()` | 获取通话状态 | 无 | Async |
| `getCallStateSync()` | 获取通话状态（同步） | 无 | Sync |

**证据**: `@ohos.telephony.call.d.ts:214-282`

#### 通话会议类

| API | 功能 | 权限 | 同步/异步 |
|-----|------|------|-----------|
| `getMainCallId()` | 获取主通话 ID | 无 | Async |
| `getSubCallIdList()` | 获取子通话 ID 列表 | 无 | Async |
| `getCallIdListForConference()` | 获取会议通话 ID | 无 | Async |

**证据**: `@ohos.telephony.call.d.ts:856-956`

#### 通话设置类

| API | 功能 | 权限 | 同步/异步 |
|-----|------|------|-----------|
| `getCallWaitingStatus()` | 获取呼叫等待状态 | GET_TELEPHONY_STATE | Async |
| `setCallWaiting()` | 设置呼叫等待 | ANSWER_CALL | Async |
| `getCallRestriction()` | 获取呼叫限制 | GET_TELEPHONY_STATE | Async |
| `setCallRestriction()` | 设置呼叫限制 | ANSWER_CALL | Async |

#### 号码处理类

| API | 功能 | 权限 | 同步/异步 |
|-----|------|------|-----------|
| `isEmergencyPhoneNumber()` | 是否紧急号码 | 无 | Async |
| `formatPhoneNumber()` | 格式化电话号码 | 无 | Async |
| `formatPhoneNumberToE164()` | 转换为 E.164 格式 | 无 | Async |

**证据**: `@ohos.telephony.call.d.ts:336-475`

#### 其他操作

| API | 功能 | 权限 | 同步/异步 |
|-----|------|------|-----------|
| `muteRinger()` | 静音来电铃声 | SET_TELEPHONY_STATE | Async |
| `hasVoiceCapability()` | 是否支持语音通话 | 无 | Sync |

---

## 权限要求

### 权限清单

| 权限 | 用途 | 保护的操作 |
|------|------|-----------|
| `ohos.permission.PLACE_CALL` | 拨号权限 | `dialCall`, `dial` |
| `ohos.permission.ANSWER_CALL` | 接听/挂断权限 | `answerCall`, `hangUpCall`, `rejectCall`, `holdCall`, `unHoldCall`, `switchCall`, `setCallWaiting`, `setCallRestriction` |
| `ohos.permission.GET_TELEPHONY_STATE` | 查询通话状态 | `getCallWaitingStatus`, `getCallRestriction` |
| `ohos.permission.SET_TELEPHONY_STATE` | 设置通话状态 | `muteRinger` |

**证据**: `services/call_manager_service/src/call_manager_service.cpp:62-68`

### 权限校验位置

权限校验在 `CallManagerService` 中执行：

```cpp
// call_manager_service.cpp:321-325
if (!TelephonyPermission::CheckPermission(OHOS_PERMISSION_PLACE_CALL)) {
    // 返回权限错误
    TELEPHONY_LOGE("PLACE_CALL permission check failed!");
    return TELEPHONY_ERR_PERMISSION_ERR;
}
```

---

## 错误码说明

### 系统错误码

| 错误码 | 说明 | 证据 |
|--------|------|------|
| 201 | Permission denied | `@ohos.telephony.call.d.ts:97` |
| 202 | Non-system applications use system APIs | `@ohos.telephony.call.d.ts:98` |
| 401 | Parameter error | `@ohos.telephony.call.d.ts:99` |

### 业务错误码

| 错误码 | 说明 | 触发条件 |
|--------|------|----------|
| 8300001 | Invalid parameter value | 参数校验失败 |
| 8300002 | Operation failed | 无法连接到服务 |
| 8300003 | System internal error | 系统内部错误 |
| 8300005 | Airplane mode is on | 飞行模式开启 |
| 8300006 | Network not in service | 网络不可用 |
| 8300007 | Conference call limit会议通话 exceeded | 数超限 |
| 8300999 | Unknown error | 未知错误 |

**证据**: `@ohos.telephony.call.d.ts:100-106`

---

## 参数校验

### 电话号码校验

- 号码格式校验（通过 `libphonenumber`）
- 空值检查
- 长度检查

### callId 校验

- 必须是正整数
- 对应的通话必须存在

### slotId 校验

- 范围: 0 到设备支持的最大卡槽数
- 默认值: 0（卡槽1）

---

## 相关文档

- [架构说明](01_Architecture.md)
- [构建系统](03_Build_System.md)
- [安全评审](04_Security_Review.md)
