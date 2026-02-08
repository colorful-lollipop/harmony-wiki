# 构建指南

## GN 构建系统概览

**配置入口**: `notification.gni`

### 路径变量

| 变量 | 值 | 说明 |
|------|-----|------|
| `component_name` | `distributed_notification_service` | 组件名 |
| `subsystem_name` | `notification` | 子系统名 |
| `services_path` | `${component_path}/services` | 服务路径 |
| `frameworks_path` | `${component_path}/frameworks` | 框架路径 |
| `interfaces_path` | `${component_path}/interfaces` | 接口路径 |

**证据**: `notification.gni:20-32`

---

## 核心 Targets

### 服务层

| Target | 类型 | 输出 | 职责 |
|--------|------|------|------|
| `libans` | ohos_shared_library | `libans.z.so` | **ANS核心服务** |
| `libreminder` | ohos_shared_library | `libreminder.z.so` | 提醒服务 |
| `libans_distributed` | ohos_shared_library | `libans_distributed.so` | 分布式支持 |
| `libdans` | ohos_shared_library | `libdans.so` | 分布式服务 |

**证据**: `services/ans/BUILD.gn:17-19`

### 框架层

| Target | 类型 | 输出 | 职责 |
|--------|------|------|------|
| `ans_innerkits` | ohos_shared_library | `libans_innerkits.so` | 客户端库 |
| `reminder_innerkits` | ohos_shared_library | `libreminder_innerkits.so` | 提醒客户端 |
| `notification_subscriber_ipc` | ohos_shared_library | `libnotification_subscriber_ipc.so` | 订阅IPC |

**证据**: `frameworks/ans/BUILD.gn:62-207`

### 语言绑定

| Target | 类型 | 输出 | 职责 |
|--------|------|------|------|
| `notification` | ohos_shared_library | `libnotification.so` | N-API绑定 |
| `cj_notification_manager_ffi` | ohos_shared_library | FFI库 | Cangjie FFI |
| `notification_manager_ani` | ohos_shared_library | ANI模块 | ArkTS绑定 |

### 接口层

| Target | 类型 | 输出 | 职责 |
|--------|------|------|------|
| `ohnotification` | ohos_shared_library | `libohnotification.so` | NDK接口 |
| `ans_sa_profile` | ohos_sa_profile | SA配置 | SA profile |

---

## SA 配置

### ANS Service (SA 3203)

```json
{
    "name": 3203,
    "libpath": "libans.z.so",
    "run-on-create": true,
    "depend": [3299],
    "extension": ["backup", "restore"]
}
```

**证据**: `sa_profile/3203.json`

### Reminder Service (SA 3204)

```json
{
    "name": 3204,
    "libpath": "libreminder.z.so",
    "run-on-create": false
}
```

**证据**: `services/reminder/sa_profile/3204.json`

---

## 特性开关

| 开关 | 默认值 | C++宏定义 |
|------|--------|-----------|
| `feature_badge_manager` | true | - |
| `feature_local_liveview` | true | - |
| `feature_disturb_manager` | true | - |
| `feature_distributed_db` | true | `DISTRIBUTED_NOTIFICATION_SUPPORTED` |
| `feature_priority_notification` | false | `ANS_FEATURE_PRIORITY_NOTIFICATION` |
| `feature_all_scenario_collaboration` | true | `ALL_SCENARIO_COLLABORATION` |
| `feature_support_geofence` | false | - |
| `notification_smart_reminder_supported` | true | `NOTIFICATION_SMART_REMINDER_SUPPORTED` |

**证据**: `notification.gni:54-80`

---

## 产物清单

| 产物 | 类型 | 安装路径 | 用途 |
|------|------|----------|------|
| `libans.z.so` | SA | system/lib | 通知核心服务 |
| `libreminder.z.so` | SA | system/lib | 提醒服务 |
| `libans_innerkits.so` | 共享库 | system/lib | 客户端库 |
| `libnotification.so` | NAPI | module/ | JS/TS接口 |
| `libohnotification.so` | NDK | system/lib | 原生开发 |
| `notification_*.abc` | 字节码 | framework/ | ETS框架 |
| `enable_notification_dialog.hap` | HAP | app/ | 权限对话框 |

---

## 依赖关系

```
libans.so (服务)
├── libans_innerkits.so
├── libans_base.so
└── [可选] libans_distributed.so

libnotification.so (NAPI)
├── libans_innerkits.so
└── libreminder_innerkits.so

libohnotification.so (NDK)
└── libans_innerkits.so
```

---

## 构建命令

```bash
# 构建主服务
gn gen out && ninja -C out libans

# 构建客户端库
ninja -C out libans_innerkits

# 构建NDK
ninja -C out ohnotification

# 构建完整组件
hb build -p //base/notification/distributed_notification_service
```
