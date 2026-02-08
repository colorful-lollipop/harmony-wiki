# 附录: Feature Flags 配置

## 目的

本文档详细说明 Pasteboard 的 Feature Flags 配置选项。

## Feature Flags 列表

### pasteboard_dlp_part_enabled

**描述**: 启用 DLP (Data Loss Prevention) 数据防泄漏功能

**默认值**: `true`

**配置位置**: `pasteboard.gni:33`

**条件编译**:
```gn
if (pasteboard_dlp_part_enabled) {
  external_deps += [ "dlp_permission_service:libdlp_permission_sdk" ]
  defines += [ "WITH_DLP" ]
}
```

**代码影响**:
```cpp
// services/core/src/pasteboard_service.cpp:28
#ifdef WITH_DLP
#include "dlp_permission_kit.h"
#endif
```

**依赖组件**: `security_dlp_permission_service`

---

### pasteboard_device_info_manager_part_enabled

**描述**: 启用设备信息管理，用于分布式设备画像

**默认值**: `true`

**配置位置**: `pasteboard.gni:34`

**条件编译**:
```gn
if (pasteboard_device_info_manager_part_enabled) {
  external_deps += [
    "device_info_manager:distributed_device_profile_common",
    "device_info_manager:distributed_device_profile_sdk",
  ]
  defines += [ "PB_DEVICE_INFO_MANAGER_ENABLE" ]
}
```

**依赖组件**: `deviceprofile_device_info_manager`

---

### pasteboard_device_manager_part_enabled

**描述**: 启用分布式设备管理，支持跨设备剪贴板

**默认值**: `true`

**配置位置**: `pasteboard.gni:35`

**条件编译**:
```gn
if (pasteboard_device_manager_part_enabled) {
  external_deps += [ "device_manager:devicemanagersdk" ]
  defines += [ "PB_DEVICE_MANAGER_ENABLE" ]
}
```

**代码影响**:
```cpp
// framework/framework/device/dm_adapter.cpp
#ifdef PB_DEVICE_MANAGER_ENABLE
// 设备管理实现
#endif
```

**依赖组件**: `distributedhardware_device_manager`

---

### pasteboard_screenlock_mgr_part_enabled

**描述**: 启用屏幕锁管理，支持锁屏状态检测

**默认值**: `true`

**配置位置**: `pasteboard.gni:36`

**条件编译**:
```gn
if (pasteboard_screenlock_mgr_part_enabled) {
  external_deps += [ "screenlock_mgr:screenlock_client" ]
  defines += [ "PB_SCREENLOCK_MGR_ENABLE" ]
}
```

**代码影响**:
```cpp
// services/core/src/pasteboard_service.cpp:162
#ifdef PB_SCREENLOCK_MGR_ENABLE
auto screenLockManager = OHOS::ScreenLock::ScreenLockManager::GetInstance();
auto isScreenLocked = screenLockManager->IsScreenLocked();
#endif
```

**依赖组件**: `theme_screenlock_mgr`

---

### pasteboard_vixl_part_enabled

**描述**: 启用 VIXL 库（ARM 指令集模拟器），用于特定架构优化

**默认值**: `true` (如果 `third_party_vixl` 存在)

**配置位置**: `pasteboard.gni:37`

**条件编译**:
```gn
if (pasteboard_vixl_part_enabled) {
  external_deps += [ "vixl:libvixl" ]
  defines += [ "PB_VIXL_ENABLE" ]
}
```

**依赖组件**: `third_party_vixl`

---

### pasteboard_dataclassification_enabled

**描述**: 启用数据分类功能，支持数据安全等级标记

**默认值**: `true`

**配置位置**: `pasteboard.gni:38`

**条件编译**:
```gn
if (pasteboard_dataclassification_enabled) {
  external_deps += [ "dataclassification:data_transit_mgr" ]
  defines += [ "PB_DATACLASSIFICATION_ENABLE" ]
}
```

**代码影响**:
```cpp
// services/core/src/pasteboard_service.cpp:193
#ifdef PB_DATACLASSIFICATION_ENABLE
auto status = DATASL_OnStart();
#endif
```

**依赖组件**: `security_dataclassification`

---

## 使用示例

### 启用/禁用 Feature

```gn
# 在产品的 args.gni 中覆盖默认值
pasteboard_dlp_part_enabled = false
pasteboard_device_manager_part_enabled = false
```

### 最小化构建

```gn
# 仅保留核心功能
pasteboard_dlp_part_enabled = false
pasteboard_device_info_manager_part_enabled = false
pasteboard_device_manager_part_enabled = false
pasteboard_screenlock_mgr_part_enabled = false
pasteboard_dataclassification_enabled = false
```

## Feature 组合建议

### 标准系统 (Full Features)

```gn
pasteboard_dlp_part_enabled = true
pasteboard_device_info_manager_part_enabled = true
pasteboard_device_manager_part_enabled = true
pasteboard_screenlock_mgr_part_enabled = true
pasteboard_dataclassification_enabled = true
```

### 轻量系统 (Minimal)

```gn
pasteboard_dlp_part_enabled = false
pasteboard_device_info_manager_part_enabled = false
pasteboard_device_manager_part_enabled = false
pasteboard_screenlock_mgr_part_enabled = false
pasteboard_dataclassification_enabled = false
pasteboard_vixl_part_enabled = false
```

### 仅本地模式 (No Distributed)

```gn
pasteboard_dlp_part_enabled = true
pasteboard_device_info_manager_part_enabled = false
pasteboard_device_manager_part_enabled = false
pasteboard_screenlock_mgr_part_enabled = true
pasteboard_dataclassification_enabled = true
```

## 相关链接

- [GN 构建 → 05_GN_Targets.md](../05_GN_Targets.md)
- [安全评审 → 06_Security.md](../06_Security.md)
