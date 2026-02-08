# 02_NAPI_Reference - N-API接口文档

> JS/TS API清单、参数说明、错误码与调用链

---

## 1. 模块信息

### 1.1 模块注册

从 `frameworks/js/napi/src/companion_device_auth_entry.cpp:607-617`:

```cpp
extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module module = {
        .nm_version = 1,
        .nm_flags = 0,
        .nm_filename = nullptr,
        .nm_register_func = ModuleInit,
        .nm_modname = "userIAM.companionDeviceAuth",
        .nm_priv = nullptr,
        .reserved = {}
    };
    napi_module_register(&module);
}
```

**模块名称**: `userIAM.companionDeviceAuth`

**导入方式**:
```typescript
import companionDeviceAuth from '@ohos.userIAM.companionDeviceAuth';
```

### 1.2 导出对象

从 `companion_device_auth_entry.cpp:570-604`:

```cpp
napi_value ModuleInit(napi_env env, napi_value exports)
{
    napi_value val = CompanionDeviceAuthInit(env, exports);  // 导出函数
    return EnumExport(env, val);                              // 导出枚举
}
```

---

## 2. API清单

### 2.1 模块级函数

| API | 类型 | 说明 | 位置 |
|-----|------|------|------|
| `getStatusMonitor(localUserId)` | 同步 | 获取StatusMonitor实例 | entry.cpp:411 |
| `registerDeviceSelectCallback(callback)` | 同步 | 注册设备选择回调 | entry.cpp:462 |
| `unregisterDeviceSelectCallback()` | 同步 | 注销设备选择回调 | entry.cpp:483 |
| `updateEnabledBusinessIds(templateId, businessIds)` | Promise | 更新启用业务ID | entry.cpp:504 |

### 2.2 StatusMonitor类方法

| API | 类型 | 说明 | 位置 |
|-----|------|------|------|
| `getTemplateStatus()` | Promise | 获取模板状态列表 | entry.cpp:106 |
| `onTemplateChange(callback)` | 同步 | 注册模板变化监听 | entry.cpp:155 |
| `offTemplateChange(callback?)` | 同步 | 注销模板变化监听 | entry.cpp:191 |
| `onContinuousAuthChange(param, callback)` | 同步 | 注册持续认证监听 | entry.cpp:226 |
| `offContinuousAuthChange(callback?)` | 同步 | 注销持续认证监听 | entry.cpp:261 |
| `onAvailableDeviceChange(callback)` | 同步 | 注册可用设备监听 | entry.cpp:296 |
| `offAvailableDeviceChange(callback?)` | 同步 | 注销可用设备监听 | entry.cpp:331 |

### 2.3 枚举类型

从 `entry.cpp:528-568`:

**BusinessId** - 业务标识
```typescript
enum BusinessId {
    DEFAULT = 0,
    VENDOR_BEGIN = 10000,
}
```

**DeviceIdType** - 设备ID类型
```typescript
enum DeviceIdType {
    UNIFIED_DEVICE_ID = 1,
    VENDOR_BEGIN = 10000,
}
```

**SelectPurpose** - 设备选择目的
```typescript
enum SelectPurpose {
    SELECT_ADD_DEVICE = 1,
    SELECT_AUTH_DEVICE = 2,
    CHECK_OPERATION_INTENT = 3,
    VENDOR_BEGIN = 10000,
}
```

---

## 3. API详细说明

### 3.1 getStatusMonitor

**功能**: 获取指定用户的StatusMonitor实例

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| localUserId | number | 是 | 本地用户ID |

**返回值**: StatusMonitor实例

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 201 | 权限检查失败 (非USE_USER_IDM权限) |
| 202 | 非系统应用 |
| 32600001 | 一般错误 |
| 32600002 | 用户不存在 |

**实现位置**: `frameworks/js/napi/src/companion_device_auth_entry.cpp:411-460`

**调用链**:
```
JS:getStatusMonitor()
  → NAPI:GetStatusMonitor()
    → CheckUseUserIdmPermission()
    → CheckCallerIsSystemApp()
    → StatusMonitor::SetLocalUserId()
      → Client:GetStatusMonitor()
        → IPC:GetTemplateStatus()
          → SA:CheckPermission()
```

### 3.2 registerDeviceSelectCallback

**功能**: 注册设备选择回调

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| callback | DeviceSelectCallback | 是 | 设备选择回调函数 |

**实现位置**: `frameworks/js/napi/src/companion_device_auth_entry.cpp:462-481`

### 3.3 updateEnabledBusinessIds

**功能**: 更新指定模板的启用业务ID列表

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| templateId | Uint8Array | 是 | 模板ID (8字节) |
| enabledBusinessIds | number[] | 是 | 业务ID数组 |

**返回值**: Promise<void>

**实现位置**: `frameworks/js/napi/src/companion_device_auth_entry.cpp:504-526`

### 3.4 StatusMonitor.getTemplateStatus

**功能**: 获取已添加的伴随设备模板状态

**返回值**: Promise<TemplateStatus[]>

**实现位置**: `frameworks/js/napi/src/companion_device_auth_entry.cpp:106-153`

---

## 4. 错误码映射

从 `frameworks/js/napi/src/companion_device_auth_napi_helper.cpp:45-53`:

| 内部错误码 | JS错误码 | 说明 |
|------------|----------|------|
| CHECK_PERMISSION_FAILED (20001) | 201 | 权限检查失败 |
| CHECK_SYSTEM_PERMISSION_FAILED (20002) | 202 | 非系统应用 |
| USER_ID_NOT_FOUND (20004) | 32600002 | 用户不存在 |
| NOT_ENROLLED (10) | 32600002 | 模板不存在 |
| INVALID_BUSINESS_ID (20003) | 32600003 | 参数无效 |
| GENERAL_ERROR (2) | 32600001 | 一般错误 |

---

## 5. 权限要求

所有API都需要以下权限:

1. **ohos.permission.USE_USER_IDM** - 使用用户身份管理
2. **系统应用** - 必须是系统应用(TokenIdKit::IsSystemAppByFullTokenID)

**权限检查代码**: `frameworks/js/napi/src/companion_device_auth_entry.cpp:35-60`

---

*文档生成时间: 2025-02-06*
