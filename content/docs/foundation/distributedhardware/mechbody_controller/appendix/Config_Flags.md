# 配置开关

## 构建配置 (GN)

### 全局配置 (mechbody.gni)

**文件**：`mechbody.gni`

```gn
declare_args() {
  # L1 特性开关
  mechbody_controller_feature_L1 = false

  # 产品特性开关 (默认启用)
  mechbody_controller_feature_product = true

  # 扩展特性开关
  mechbody_controller_extended = false
}
```

### 条件编译

| Define | 条件 | 用途 | 文件位置 |
|--------|------|------|----------|
| `MECHBODY_CONTROLLER_L1` | `mechbody_controller_feature_L1 == true` | L1 变体编译 | napi/BUILD.gn:42 |
| `MECHBODY_CONTROLLER_EXTENDED` | `mechbody_controller_extended == true` | 扩展功能 | services/BUILD.gn:114 |
| `BINDER_IPC_32BIT` | `target_cpu == "arm"` | 32-bit ARM IPC | napi/BUILD.gn:45 |

### 使用示例

```bash
# 启用 L1 特性
hb build mechbody_controller --gn-args "mechbody_controller_feature_L1=true"

# 启用扩展特性
hb build mechbody_controller --gn-args "mechbody_controller_extended=true"

# 禁用产品特性 (仅测试)
hb build mechbody_controller --gn-args "mechbody_controller_feature_product=false"
```

---

## 编译器配置

### 编译选项 (services/BUILD.gn)

```gn
cflags = [
  "-frtti",           # 运行时类型信息
  "-fpie",            # 位置无关可执行
]

cflags_cc = [
  "-fstack-protector-strong",  # 栈保护
]

ldflags = [
  "-Wl,-z,relro",    # 只读重定位
  "-Wl,-z,now",      # 立即绑定
]
```

### Sanitizer 配置

```gn
sanitize = {
  boundary_sanitize = true    # 边界检查
  cfi = true                   # 控制流完整性
  cfi_cross_dso = true         # 跨 DSO CFI
  debug = false
  integer_overflow = true      # 整数溢出检查
  ubsan = true                 # 未定义行为检查
}
```

---

## 服务配置

### SA 配置 (sa_profile/8550.json)

**文件**：`sa_profile/8550.json`

```json
{
  "process": "mechbody",
  "systemability": [
    {
      "name": 8550,
      "libpath": "libmechbody_service.z.so",
      "run-on-create": false,
      "auto-restart": true,
      "distributed": false,
      "min_hdi_proxy_version": []
    }
  ]
}
```

| 字段 | 值 | 说明 |
|------|-----|------|
| **process** | `mechbody` | 运行进程名 |
| **libpath** | `libmechbody_service.z.so` | 库路径 |
| **run-on-create** | `false` | 懒加载 |
| **auto-restart** | `true` | 崩溃后自动重启 |
| **distributed** | `false` | 非分布式 |

---

### 初始化配置 (etc/init/mechbody.cfg)

**文件**：`etc/init/mechbody.cfg`

```json
{
  "name": "mechbody_controller",
  "uid": "mechbody",
  "gid": "mechbody",
  "permission": [
    "ohos.permission.ACCESS_BLUETOOTH",
    "ohos.permission.MANAGE_BLUETOOTH",
    "ohos.permission.MANAGE_LOCAL_ACCOUNTS",
    "ohos.permission.GET_BLUETOOTH_LOCAL_MAC",
    "ohos.permission.INJECT_INPUT_EVENT",
    "ohos.permission.CAMERA"
  ],
  "selinux": {
    "context": "u:r:mechbody:s0"
  }
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **name** | `mechbody_controller` | 服务名称 |
| **uid** | `mechbody` | 用户 ID |
| **gid** | `mechbody` | 组 ID |
| **selinux.context** | `u:r:mechbody:s0` | SELinux 上下文 |

---

## Bundle 配置 (bundle.json)

**文件**：`bundle.json`

```json
{
  "name": "@ohos/mechbody_controller",
  "description": "mechbody controller service",
  "version": "1.0",
  "component": {
    "name": "mechbody_controller",
    "subsystem": "distributedhardware",
    "syscap": [
      "SystemCapability.Mechanic.Core"
    ],
    "features": [
      "mechbody_controller_feature_L1",
      "mechbody_controller_feature_product",
      "mechbody_controller_extended"
    ],
    "adapted_system_type": [
      "standard"
    ],
    "build": {
      "sub_component": [
        "//foundation/distributedhardware/mechbody_controller/services:mechbody_service",
        "//foundation/distributedhardware/mechbody_controller/interface/ets/mech_manager:mechanic_manager_group_ani",
        "//foundation/distributedhardware/mechbody_controller/interface/napi/mech_manager:mechanicmanager_napi",
        "//foundation/distributedhardware/mechbody_controller/etc/init:etc",
        "//foundation/distributedhardware/mechbody_controller/sa_profile:mechbody_sa_profile"
      ],
      "inner_kits": [
        {
          "type": "xxxx",
          "name": "xxxx"
        }
      ],
      "test": []
    }
  }
}
```

---

## 运行时配置

### 蓝牙配置

**BLE UUID** (`ble_send_manager.cpp:63-65`)

```cpp
static constexpr char BLE_UUID_SERVICE[] = "0000FEED-0000-1000-8000-00805F9B34FB";
static constexpr char BLE_UUID_CHARACTERISTIC[] = "0000FEEE-0000-1000-8000-00805F9B34FB";
static constexpr char BLE_UUID_DESCRIPTOR[] = "00002902-0000-1000-8000-00805F9B34FB";
```

### IPC 命令码

**文件**：`services/include/mechbody_controller_ipc_interface_code.h`

```cpp
enum class IMechBodyControllerCode : uint32_t {
  MECHBODY_RESERVED = 0,
  ATTACH_STATE_CHANGE_LISTEN_ON = 1,
  ATTACH_STATE_CHANGE_LISTEN_OFF = 2,
  GET_ATTACHED_DEVICES = 3,
  SET_USER_OPERATION = 4,
  SET_CAMERA_TRACKING_ENABLED = 5,
  GET_CAMERA_TRACKING_ENABLED = 6,
  TRACKING_EVENT_LISTEN_ON = 7,
  GET_CAMERA_TRACKING_LAYOUT = 8,
  SET_CAMERA_TRACKING_LAYOUT = 9,
  GET_ROTATION_LIMITS = 10,
  ROTATE_BY_DEGREE = 11,
  REGISTER_CMD_CHANNEL = 12,
  // 回调命令
  ATTACH_STATE_CHANGE_CALLBACK = 24,
  TRACKING_EVENT_CALLBACK = 25,
  ROTATION_AXES_STATUS_CHANGE_CALLBACK = 26,
  ROTATE_CALLBACK = 27,
  SEARCH_TARGET_CALLBACK = 28,
};
```

---

## 调试配置

### 日志级别

**日志宏** (`services/include/mechbody_controller_log.h`)

| 级别 | 宏 | 说明 |
|------|-----|------|
| **DEBUG** | `HILOGD` | 调试信息 |
| **INFO** | `HILOGI` | 普通信息 |
| **WARN** | `HILOGW` | 警告 |
| **ERROR** | `HILOGE` | 错误 |
| **FATAL** | `HILOGF` | 致命错误 |

### 启用调试日志

```cpp
// 在代码中启用调试日志
#define DEBUG_MECHBODY 1

#ifdef DEBUG_MECHBODY
#define HILOGD(fmt, ...) printf(fmt, ##__VA_ARGS__)
#else
#define HILOGD(fmt, ...)
#endif
```

---

## Feature Flag 总结

| Flag | 默认值 | 类型 | 用途 |
|------|--------|------|------|
| `mechbody_controller_feature_L1` | `false` | GN | L1 设备特性 |
| `mechbody_controller_feature_product` | `true` | GN | 产品特性 |
| `mechbody_controller_extended` | `false` | GN | 扩展特性 |
| `SA 8550 run-on-create` | `false` | JSON | SA 懒加载 |
| `SA 8550 auto-restart` | `true` | JSON | 自动重启 |
| `boundary_sanitize` | `true` | BUILD | 边界检查 |
| `cfi` | `true` | BUILD | 控制流完整性 |
| `ubsan` | `true` | BUILD | 未定义行为 |
