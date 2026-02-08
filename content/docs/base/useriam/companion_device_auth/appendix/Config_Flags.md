# Config_Flags - 配置与宏定义

> 编译配置、Feature Flags与宏定义汇总

---

## 1. Feature Flags

### 1.1 主配置 (companion_device_auth.gni)

| Flag | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| companion_device_auth_enable_auth_state_maintain_simulation | bool | true | 启用认证状态模拟 |
| companion_device_auth_deploy_mode | string | "standard" | 部署模式 (standard/ap) |
| companion_device_auth_path | string | "//base/useriam/companion_device_auth" | 项目路径 |
| companion_device_auth_enable_extension | bool | false | 启用扩展 (TEE支持) |
| companion_device_auth_enable_coverage | bool | false | 启用覆盖率检测 |

### 1.2 可选依赖检测

| Flag | 检测条件 | 影响 |
|------|----------|------|
| companion_device_auth_has_account_os_account | 有os_account部件 | 使用真实用户ID管理 |
| companion_device_auth_has_soft_bus_channel | 有dsoftbus部件 | 启用SoftBus通道 |
| companion_device_auth_has_user_auth_framework | 有user_auth_framework部件 | 启用UserAuth框架适配 |

---

## 2. 编译宏定义

### 2.1 条件编译宏

| 宏 | 定义位置 | 触发条件 | 说明 |
|----|----------|----------|------|
| HAS_USER_AUTH_FRAMEWORK | services/BUILD.gni | has_user_auth_framework | 启用UserAuth框架适配代码 |
| HAS_SOFT_BUS_CHANNEL | services/BUILD.gni | has_soft_bus_channel | 启用SoftBus通道代码 |
| ENABLE_TEST | test配置 | 构建测试 | 启用测试代码 |

### 2.2 常量定义 (common_defines.h)

```cpp
// SA ID
constexpr int32_t COMPANION_DEVICE_AUTH_SA_ID = 945;

// 参数索引
constexpr size_t ARGS_ONE = 1;
constexpr size_t ARGS_TWO = 2;
constexpr size_t PARAM0 = 0;
constexpr size_t PARAM1 = 1;

// Token掩码
const uint64_t TOKEN_ID_LOW_MASK = 0xffffffff;

// JS错误码映射
const int32_t FRAMEWORKS_CHECK_PERMISSION_FAILED = 201;
const int32_t FRAMEWORKS_CHECK_SYSTEM_PERMISSION_FAILED = 202;
const int32_t FRAMEWORKS_GENERAL_ERROR = 32600001;
const int32_t FRAMEWORKS_NOT_FOUND = 32600002;
const int32_t FRAMEWORKS_INVALID_PARAMS = 32600003;

// 权限
const std::string USE_USER_IDM_PERMISSION = "ohos.permission.USE_USER_IDM";
```

---

## 3. Sanitizer配置

### 3.1 安全加固选项

```gn
companion_device_auth_sanitize = {
    integer_overflow = true      # 整数溢出检测
    ubsan = true                 # 未定义行为检测
    boundary_sanitize = true     # 边界检测
    cfi = true                   # 控制流完整性
    cfi_cross_dso = true         # 跨DSO CFI
    debug = false                # 调试模式
    blocklist = "${companion_device_auth_path}/cfi_blocklist.txt"
}
```

### 3.2 分支保护

```gn
branch_protector_ret = "pac_ret"  # ARM Pointer Authentication
```

---

## 4. 枚举类型

### 4.1 ResultCode (错误码)

```cpp
enum ResultCode : int32_t {
    SUCCESS = 0,
    FAIL = 1,
    GENERAL_ERROR = 2,
    CANCELED = 3,
    TIMEOUT = 4,
    TYPE_NOT_SUPPORT = 5,
    TRUST_LEVEL_NOT_SUPPORT = 6,
    BUSY = 7,
    INVALID_PARAMETERS = 8,
    LOCKED = 9,
    NOT_ENROLLED = 10,
    CANCELED_FROM_WIDGET = 11,
    HARDWARE_NOT_SUPPORTED = 12,
    PIN_EXPIRED = 13,
    COMPLEXITY_CHECK_FAILED = 14,
    AUTH_TOKEN_CHECK_FAILED = 15,
    AUTH_TOKEN_EXPIRED = 16,
    COMMUNICATION_ERROR = 17,
    NO_VALID_CREDENTIAL = 18,
    CHECK_PERMISSION_FAILED = 20001,
    CHECK_SYSTEM_PERMISSION_FAILED = 20002,
    INVALID_BUSINESS_ID = 20003,
    USER_ID_NOT_FOUND = 20004,
};
```

### 4.2 设备相关枚举

```cpp
enum class DeviceIdType : int32_t {
    UNKNOWN = 0,
    UNIFIED_DEVICE_ID = 1,
    VENDOR_BEGIN = 10000,
};

enum class SelectPurpose : int32_t {
    SELECT_ADD_DEVICE = 1,
    SELECT_AUTH_DEVICE = 2,
    CHECK_OPERATION_INTENT = 3,
    VENDOR_BEGIN = 10000,
};

enum class AuthType : int32_t {
    PIN = 1,
    FACE = 2,
    FINGERPRINT = 4,
    COMPANION_DEVICE = 64,
};

enum class BusinessId : int32_t {
    DEFAULT = 0,
    VENDOR_BEGIN = 10000,
};
```

---

## 5. 超时配置

### 5.1 XCollie超时

```cpp
// services/external_adapters/access_token/src/access_token_kit_adapter_impl.cpp
constexpr uint32_t API_CALL_TIMEOUT = 20;  // 20秒
```

### 5.2 操作超时

| 操作 | 超时时间 | 位置 |
|------|----------|------|
| 权限检查 | 20s | access_token_kit_adapter_impl.cpp |
| UserAuth操作 | 20s | user_auth_adapter_impl.cpp |
| DeviceManager操作 | 20s | device_manager_adapter_impl.cpp |

---

## 6. 资源限制

### 6.1 内存限制

从 `bundle.json`:
```json
{
    "rom": "3380KB",
    "ram": "7271KB"
}
```

### 6.2 字符串长度限制

```cpp
// frameworks/js/napi/src/companion_device_auth_napi_helper.cpp
constexpr size_t MAX_STRING_LENGTH = 65536;  // 64KB
```

---

## 7. 日志配置

### 7.1 日志Domain

| 组件 | Domain |
|------|--------|
| IPC | 0xD002421 |

### 7.2 日志Tag

| 文件 | Tag |
|------|-----|
| N-API | CDA_NAPI |
| Service | CDA_SA |
| Client | CDA_CLIENT |
| IPC | COMPANION_DEVICE_AUTH_IPC |

---

*文档生成时间: 2025-02-06*
