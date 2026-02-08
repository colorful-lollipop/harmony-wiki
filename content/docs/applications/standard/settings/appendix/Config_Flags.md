# 关键宏定义与 Feature Flags

> Settings 应用的关键宏定义与配置标志

---

## 目的

本文档提供 Settings 应用的关键宏定义、配置标志和编译选项。

## 适用范围

- 目标读者：构建工程师、系统开发者
- 项目：@ohos/settings (Settings 3.1)

---

## 宏定义

### N-API/ANI/CJ FFI 参数宏

```cpp
// napi/settings/napi_settings.h
#define ARGS_ONE 1
#define ARGS_TWO 2
#define ARGS_THREE 3
#define ARGS_FOUR 4
#define ARGS_FIVE 5

#define PARAM0 0
#define PARAM1 1
#define PARAM2 2
#define PARAM3 3
#define PARAM4 4
```

**说明**：
- `ARGS_*`：参数数量
- `PARAM*`：参数索引
- 用于验证函数参数数量和索引

### 调用类型枚举

```cpp
// napi/settings/napi_settings.h
enum CallType {
    INVALID_CALL,
    STAGE_SYNC,
    STAGE_CALLBACK,
    STAGE_CALLBACK_SPECIFIC,
    STAGE_PROMISE,
    STAGE_PROMISE_SPECIFIC,
    FA_SYNC,
    FA_CALLBACK,
    FA_PROMISE
};
```

**说明**：
- `STAGE_*`：Stage 模型调用
- `FA_*`：Feature Ability 调用
- `*_SYNC`：同步调用
- `*_CALLBACK`：回调模式
- `*_PROMISE`：Promise 模式

---

## 常量定义

### 设置数据常量

```cpp
// napi/settings/napi_settings.cpp
const std::string SETTINGS_DATA_BASE_URI = "dataability:///com.ohos.settingsdata.DataAbility";
const std::string SETTINGS_DATA_FIELD_KEYWORD = "KEYWORD";
const std::string SETTINGS_DATA_FIELD_VALUE = "VALUE";
const std::string PERMISSION_EXCEPTION = "Permission denied";
const std::string DEFAULT_ANONYMOUS = "******";
```

**说明**：
- `SETTINGS_DATA_BASE_URI`：DataAbility 基础 URI
- `SETTINGS_DATA_FIELD_KEYWORD`：字段名（键名）
- `SETTINGS_DATA_FIELD_VALUE`：字段名（值）
- `PERMISSION_EXCEPTION`：权限异常消息
- `DEFAULT_ANONYMOUS`：敏感数据默认值（用于日志脱敏）

### 错误码常量

```cpp
// napi/settings/napi_settings.cpp
const int PERMISSION_EXCEPTION_CODE = 201;
const int QUERY_SUCCESS_CODE = 1;
const int STATUS_ERROR_CODE = -1;
const int PERMISSION_DENIED_CODE = -2;
const int USERID_HELPER_NUMBER = 100;
const int DATA_SHARE_DIED1 = 29189;
const int DATA_SHARE_DIED2 = 32;
```

**说明**：
- `PERMISSION_EXCEPTION_CODE`：权限异常（201）
- `QUERY_SUCCESS_CODE`：查询成功（1）
- `STATUS_ERROR_CODE`：状态错误（-1）
- `PERMISSION_DENIED_CODE`：权限拒绝（-2）
- `USERID_HELPER_NUMBER`：用户 ID 常量（100）
- `DATA_SHARE_DIED1`：DataShare 死亡错误码 1（29189）
- `DATA_SHARE_DIED2`：DataShare 死亡错误码 2（32）

### 智能场景错误码

```cpp
// napi/intelligentscene/common/intelligence_inner_errors.h
const int32_t ERROR_PERMISSION_DENIED = 201;          // No permission to call the interface.
const int32_t ERROR_SYSTEM_CAP_ERROR  = 801;          // The specified SystemCapability was not found.
const int32_t ERROR_INTERNAL_ERROR               = 35200001;    // Internal error.
const int32_t ERROR_IPC_ERROR                    = 1600002;    // Marshalling or unmarshalling error.
const int32_t ERROR_SERVICE_CONNECT_ERROR        = 1600003;    // Failed to connect to service.
```

**说明**：
- `ERROR_PERMISSION_DENIED`：权限被拒绝
- `ERROR_SYSTEM_CAP_ERROR`：系统能力未找到
- `ERROR_INTERNAL_ERROR`：内部错误
- `ERROR_IPC_ERROR`：IPC 序列化/反序列化错误
- `ERROR_SERVICE_CONNECT_ERROR`：连接服务失败

---

## 配置标志

### 构建选项

#### GN Targets

| Target | 关键配置 | 说明 |
|--------|----------|------|
| settings (ANI) | is_boot_abc = "True" | 系统启动时加载 |
| intelligentscene (ANI) | is_boot_abc = "True" | 系统启动时加载 |

#### 版本脚本

- `settings_ani.versionscript`：控制符号导出
- `intelligentscene_ani.versionscript`：控制符号导出

### 产品配置

| 产品 | 状态 | SDK 版本 |
|--------|--------|----------|
| phone | ✅ 激活 | 23 |
| wearable | ⚠️ 已注释 | 23 |

---

## 系统能力

```json
// bundle.json
{
  "syscap": [
    "SystemCapability.Applications.Settings.Core",
    "SystemCapability.Applications.IntelligentScene"
  ]
}
```

**说明**：
- `Settings.Core`：设置管理核心能力
- `IntelligentScene`：智能场景能力（免打扰模式等）

---

## 编译宏

### 平台检测

```typescript
// product/phone/src/main/ets/pages/settingList.ets
const deviceTypeInfo = deviceInfo.deviceType;
```

**说明**：
- 用于判断设备类型（default, tablet, wearable）
- 根据设备类型调整 UI 布局

---

## 依赖配置

### 外部依赖

从 bundle.json 中声明的依赖：

| 依赖 | 用途 | 最小版本 |
|--------|--------|----------|
| ability_runtime | 能力运行时 | - |
| ace_engine | ACE 引擎 | - |
| data_share | 数据共享框架 | - |
| os_account | 账户管理 | - |
| napi | N-API 支持 | - |
| ipc | IPC 框架 | - |
| hisysevent | 系统事件 | - |
| samgr | 系统能力管理器 | - |
| access_token | 访问令牌 | - |
| distributed_notification_service | 分布式通知 | - |

---

## 调试选项

### 日志控制

```cpp
// napi/settings/napi_settings_log.h
#define SETTING_LOG_DEBUG(msg) OH_LOG_DEBUG("Settings", msg)
#define SETTING_LOG_INFO(msg) OH_LOG_INFO("Settings", msg)
#define SETTING_LOG_WARN(msg) OH_LOG_WARN("Settings", msg)
#define SETTING_LOG_ERROR(msg) OH_LOG_ERROR("Settings", msg)
```

**说明**：
- 使用 OpenHarmony HiLog 系统
- 支持不同日志级别（DEBUG/INFO/WARN/ERROR）
- 标签："Settings"

---

## 相关跳转

- **[00_Overview.md](00_Overview.md)** - 项目概览
- **[06_GN_Targets.md](06_GN_Targets.md)** - GN Targets 详细文档
- **[08_Security_Audit.md](08_Security_Audit.md)** - 安全评审

---

**最后更新**：2026-02-06 00:11:23
