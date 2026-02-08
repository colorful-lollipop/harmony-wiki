# 关键宏与 Feature Flags

## 目的

本文档说明 DSLM 模块的关键宏定义和编译特性开关（Feature Flags）。

## 适用范围

- ✅ 关键宏定义
- ✅ Feature Flags 说明
- ✅ 编译选项

## 关键宏定义

### 1. 版本宏

**定义位置**：`services/dslm/dslm_core_defines.h:34-36`

```c
#define VERSION_MAJOR 3U
#define VERSION_MINOR 0U
#define VERSION_PATCH 0U

static inline uint32_t GetCurrentVersion(void)
{
    return (VERSION_MAJOR << 16U) + (VERSION_MINOR << 8U) + VERSION_PATCH;
}
```

**说明**：当前 DSLM 版本为 3.0.0

---

### 2. 设备标识符宏

**定义位置**：`interfaces/inner_api/include/device_security_defines.h:25`

```c
#define DEVICE_ID_MAX_LEN 64

typedef struct DeviceIdentify {
    uint32_t length;
    uint8_t identity[DEVICE_ID_MAX_LEN];
} DeviceIdentify;
```

**说明**：设备标识符最大长度为 64 字节

---

### 3. 状态机宏

**定义位置**：`baselib/utils/include/utils_state_machine.h`

```c
#define EVENT_QUERY_TIMEOUT  0
#define EVENT_QUERY_RESPONSE  1
#define EVENT_CREDENTIAL    2
#define EVENT_DEVICE_ONLINE  3
#define EVENT_DEVICE_OFFLINE 4
#define EVENT_TIMEOUT     5
```

**说明**：DSL 设备状态机的事件定义

---

### 4. 消息相关宏

**定义位置**：`baselib/msglib/include/messenger.h` (TLV 结构体)

```c
typedef struct MessengerConfig {
    const char *pkgName;
#ifdef L2_STANDARD
    const char *primarySockName;
    const char *secondarySockName;
#else
    const char *primarySessName;
    const char *secondarySessName;
#endif
    DeviceMessageReceiver messageReceiver;
    DeviceStatusReceiver statusReceiver;
    MessageSendResultNotifier sendResultNotifier;
    uint32_t threadCnt;
} MessengerConfig;
```

**说明**：Standard 系统使用 Socket 名称，Lite 系统使用会话名称

---

### 5. 错误码宏

**定义位置**：`interfaces/inner_api/include/device_security_defines.h:40-92`

**关键错误码**：
```c
#define DEFAULT_OPTION NULL

enum {
    SUCCESS = 0,
    ERR_INVALID_PARA = 1,
    ERR_INVALID_LEN_PARA = 2,
    ERR_NO_MEMORY = 3,
    ERR_NO_CHALLENGE = 5,
    ERR_NO_CRED = 6,
    ERR_SA_BUSY = 7,
    ERR_TIMEOUT = 8,
    ERR_PERMISSION_DENIAL = 30,
    // ... 更多错误码
};
```

---

### 6. 发送次数与超时宏

**定义位置**：`services/dslm/BUILD.gn:64-67`

```gn
defines = [
    "MAX_SEND_TIMES=5",
    "SEND_MSG_TIMEOUT_LEN=40000",  # 40 秒（生产版本）
]
```

**测试版本**：
```gn
defines = [
    "MAX_SEND_TIMES=5",
    "SEND_MSG_TIMEOUT_LEN=500",  # 0.5 秒（测试版本）
]
```

**说明**：设备凭据请求最多重试 5 次，超时 40 秒（生产）或 0.5 秒（测试）

---

### 7. 自动卸载超时宏

**定义位置**：`services/sa/standard/dslm_service.cpp:33`

```cpp
constexpr uint32_t UNLOAD_TIMEOUT = 10000;  // 10 秒
```

**说明**：SA 10 秒无请求后自动卸载

---

### 8. IPC 命令宏

**定义位置**：`common/include/dslm_service_ipc_interface_code.h`

```cpp
enum class DeviceSecurityLevelInterfaceCode {
    CMD_GET_DEVICE_SECURITY_LEVEL = 1,
};

enum class DeviceSecurityLevelCallbackInterfaceCode {
    CMD_SET_DEVICE_SECURITY_LEVEL = 1,
};
```

---

### 9. SA ID 宏

**定义位置**：`common/include/idevice_security_level.h`

```cpp
constexpr int32_t DEVICE_SECURITY_LEVEL_MANAGER_SA_ID = 3511;
```

---

### 10. 凭据类型宏

**定义位置**：`oem_property/include/dslm_cred.h`

```c
#define CRED_TYPE_STANDARD  1
#define CRED_TYPE_SMALL     2
#define CRED_TYPE_MINI      3
```

---

### 11. 签名算法宏

**定义位置**：`oem_property/common/dslm_credential_utils.c`

```c
#define TYPE_ECDSA_SHA_256  0
#define TYPE_ECDSA_SHA_384  1
```

**说明**：支持 ECDSA with SHA256 和 SHA384 两种签名算法

---

### 12. 安全等级宏

**定义位置**：`services/dslm/dslm_core_defines.h`

```c
#define CRED_MAX_LEVEL_TYPE_STANDARD  5
#define CRED_MAX_LEVEL_TYPE_SMALL     3
#define CRED_MAX_LEVEL_TYPE_MINI      1
```

**说明**：不同系统形态的最大安全等级不同（Standard=SL5, Small=SL3, Mini=SL1）

## Feature Flags（特性开关）

### 1. device_security_level_feature_cred_level

**定义位置**：`oem_property/ohos/standard/BUILD.gn`

**默认值**：1

**说明**：设备凭据的安全等级（1-5，对应 SL1-SL5）

**编译配置**：
```gn
declare_args() {
  device_security_level_feature_cred_level = 1
}
```

**使用**：
```c
// 在 oem_property/ohos/common/dslm_ohos_verify.c 中
static const uint32_t CRED_MAX_LEVEL = device_security_level_feature_cred_level;
```

---

### 2. device_security_level_feature_plugin_path

**定义位置**：`services/sa/BUILD.gn`

**默认值**：""（空字符串）

**说明**：动态加载的插件 SO 路径。如果为空，使用内置 OEM 实现。

**编译配置**：
```gn
declare_args() {
  device_security_level_feature_plugin_path = ""
}
```

**使用**：
```cpp
// 在 services/sa/standard/dslm_service.cpp:182-189
#ifdef PLUGIN_SO_PATH
    handle_ = dlopen(PLUGIN_SO_PATH, RTLD_NOW);
#endif
```

---

### 3. device_security_level_feature_secondary_session_name

**定义位置**：`services/msg/BUILD.gn`

**默认值**：""（空字符串）

**说明**：DSoftBus 次级会话名称（用于特定场景）

**编译配置**：
```gn
declare_args() {
  device_security_level_feature_secondary_session_name = ""
}
```

---

### 4. device_security_level_feature_coverage

**定义位置**：`common/BUILD.gn`

**默认值**：false

**说明**：代码覆盖率特性（用于测试）

**编译配置**：
```gn
declare_args() {
  device_security_level_feature_coverage = false
}
```

## 编译选项

### 安全加固选项（Standard 版本）

**定义位置**：`interfaces/inner_api/BUILD.gn:102-109`, `services/sa/BUILD.gn:152-159`

```gn
sanitize = {
  integer_overflow = true    # 整数溢出检测
  ubsan = true              # 未定义行为检测
  boundary_sanitize = true   # 边界检查
  cfi = true                # 控制流完整性
  cfi_cross_dso = true      # 跨 DSO CFI
  blocklist = "../../cfi_blocklist.txt"
}
branch_protector_ret = "pac_ret"  # 返回地址保护
```

**说明**：Standard 版本启用全面的安全加固

---

### 系统形态选择

**定义位置**：`interfaces/inner_api/BUILD.gn:22-29`

```gn
if (os_level == "standard") {
    deps = [ ":dslm_sdk" ]
} else if (os_level == "small") {
    deps = [ ":dslm_sdk_small" ]
} else if (os_level == "mini") {
    deps = [ ":dslm_sdk_mini" ]
}
```

**说明**：根据 `os_level` 编译不同的 SDK 版本

---

### SELinux 配置

**定义位置**：`profile/dslm_service.cfg`

```json
"secon": "u:r:dslm_service:s0"
```

**说明**：DSL 服务的 SELinux Context 为 `u:r:dslm_service:s0`

---

### UID/GID 配置

**定义位置**：`profile/dslm_service.cfg`

```json
"uid": "3046",
"gid": "3046"
```

**说明**：DSL 服务的用户 ID 和组 ID 为 3046

---

### APL 配置

**定义位置**：`profile/dslm_service.cfg`

```json
"apl": "system_basic"
```

**说明**：DSL 服务的 APL（Ability Privilege Level）为 system_basic

---

### 权限配置

**定义位置**：`profile/dslm_service.cfg`

```json
"permission": [
    "ohos.permission.ACCESS_IDS",
    "ohos.permission.sec.ACCESS_UDID",
    "ohos.permission.ACCESS_SERVICE_DM",
    "ohos.permission.DISTRIBUTED_DATASYNC"
],
"permission_acls": [
    "ohos.permission.ACCESS_IDS"
]
```

**说明**：DSL 服务需要 4 个权限，其中 ACCESS_IDS 为 ACL

## 关键结论

1. **版本宏**：VERSION_MAJOR/MINOR/PATCH，当前为 3.0.0
2. **设备标识符**：最大长度 64 字节（DEVICE_ID_MAX_LEN）
3. **Feature Flags**：3 个可配置特性（cred_level、plugin_path、secondary_session_name）
4. **安全加固**：Standard 版本启用 CFI、UBSan、Integer Overflow Sanitize
5. **系统标识**：SA ID 3511、UID/GID 3046/3046、SELinux u:r:dslm_service:s0

## 相关跳转

- [06_GN_Targets.md](../06_GN_Targets.md) - GN Targets 详解
- [02_Directory_Structure.md](../02_Directory_Structure.md) - 目录结构
