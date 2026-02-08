# 关键宏与特性开关

## 概述

本文档梳理 ability_runtime 中的关键编译配置开关（Feature Flags）和宏定义。

## 特性开关（Feature Flags）

### 全局配置（ability_runtime.gni）

**位置**：`ability_runtime.gni`

| 变量名 | 默认值 | 类型 | 说明 |
|--------|--------|------|------|
| `ability_runtime_auto_fill` | true | bool | 自动填充扩展支持 |
| `ability_runtime_child_process` | true | bool | 子进程支持 |
| `ability_runtime_graphics` | true | bool | 图形依赖特性 |
| `ability_runtime_power` | true | bool | 电源管理特性 |
| `ability_runtime_appspawn` | true | bool | 应用孵化支持 |
| `ability_runtime_auto_fill_ability` | - | string | 自动填充 Ability 名称 |
| `ability_runtime_smart_auto_fill_ability` | - | string | 智能自动填充 Ability |
| `ability_runtime_check_internet_permission` | false | bool | 网络权限检查 |
| `ability_runtime_forbid_start_enabled` | false | bool | 启动禁止功能 |
| `ability_runtime_app_no_response_dialog` | false | bool | ANR 对话框 |
| `ability_runtime_screenlock_enable` | true | bool | 锁屏集成 |
| `ability_runtime_udmf_enable` | true | bool | UDMF 数据管理 |
| `ability_runtime_hitrace_enable` | true | bool | HiTrace 追踪 |
| `ability_runtime_hiperf_enable` | true | bool | HiPerf 性能分析 |

### 可选特性

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `ability_runtime_action_extension` | true | 动作扩展 |
| `ability_runtime_photo_editor_extension` | true | 图片编辑扩展 |
| `ability_runtime_share_extension` | true | 分享扩展 |
| `ability_runtime_ui_service_extension` | true | UI 服务扩展 |
| `ability_runtime_media_library_enable` | true | 媒体库支持 |
| `ability_runtime_dsoftbus_enable` | false | 分布式软总线 |
| `ability_runtime_no_screen` | false | 无屏设备支持 |
| `ability_runtime_feature_coverage` | false | 代码覆盖率 |
| `ability_runtime_feature_sandboxmanager` | false | 沙箱管理器 |

### 条件编译开关

```gn
# 条件启用示例
if (defined(global_parts_info) &&
    defined(global_parts_info.hiviewdfx_hicollie)) {
  app_mgr_service_hicollie_enable = true
} else {
  app_mgr_service_hicollie_enable = false
}
```

## 编译宏定义

### 条件编译宏

| 宏名 | 定义位置 | 说明 |
|------|---------|------|
| `SUPPORT_GRAPHICS` | 条件编译 | 图形支持 |
| `BGTASKMGR_CONTINUOUS_TASK_ENABLE` | 条件编译 | 后台连续任务 |
| `OHOS_BUILD` | 全局定义 | OpenHarmony 构建 |

### 日志宏

**HiLog 标签定义**：
```cpp
constexpr OHOS::HiviewDFX::HiLogLabel LABEL = {
    LOG_CORE,          // 日志域
    0,                 // 模块 ID
    "AbilityManager"    // 标签名
};
```

**日志级别**：
```cpp
HILOG_DEBUG(LOG_CORE, "Debug message");
HILOG_INFO(LOG_CORE, "Info message");
HILOG_WARN(LOG_CORE, "Warn message");
HILOG_ERROR(LOG_CORE, "Error message");
HILOG_FATAL(LOG_CORE, "Fatal message");
```

### 错误码宏

| 宏名 | 值 | 说明 |
|------|------|------|
| `ERR_OK` | 0 | 成功 |
| `ERR_INVALID_VALUE` | 401 | 参数不合法 |
| `CHECK_PERMISSION_FAILED` | - | 权限校验失败 |
| `ERR_PERMISSION_DENIED` | - | 权限拒绝 |

## SA 配置

### SA ID 常量

| 服务 | SA ID | 定义位置 |
|------|--------|---------|
| AbilityManagerService | 3701 | `ability_manager_service.h` |
| AppManagerService | 1201 | `app_mgr_service.h` |
| QuickFixManager | 动态分配 | `quick_fix_manager_service.h` |
| UriPermissionManager | 动态分配 | `uri_permission_manager_service.h` |

### SA 注册宏

```cpp
// 延迟注册
DECLARE_DELAYED_SINGLETON(ServiceName)

// 系统能力声明
DECLEAR_SYSTEM_ABILITY(ServiceName)

// 注册
REGISTER_SYSTEM_ABILITY_BY_ID(ServiceName, SA_ID, true)
```

## 配置文件

### 服务配置文件

| 文件 | 路径 | 说明 |
|------|------|------|
| `abilitymgr_sa.cfg` | `services/abilitymgr/etc/` | AbilityManagerService 配置 |
| `appmgr_sa.cfg` | `services/appmgr/etc/` | AppManagerService 配置 |

### 配置格式

```json
{
  "services": [
    {
      "name": "AbilityManagerService",
      "path": "/system/lib64/libability_manager_service.so",
      "run-on-create": true,
      "permissions": [
        "ohos.permission.MANAGE_ABILITIES"
      ]
    }
  ]
}
```

## 运行时参数

### 能力常量

```cpp
// 生命周期状态
constexpr int32_t STATE_INITIAL = 0;
constexpr int32_t STATE_INACTIVE = 1;
constexpr int32_t STATE_ACTIVATED = 2;
constexpr int32_t STATE_BACKGROUND = 3;
constexpr int32_t STATE_TERMINATED = 4;

// 启动模式
constexpr int32_t SINGLETON = 0;
constexpr int32_t LAUNCH_NEW = 1;
constexpr int32_t RESET = 2;
```

### 窗口配置

```cpp
// 窗口显示模式
constexpr int32_t SHOW_MODE_MINIMIZED = 0;
constexpr int32_t SHOW_MODE_FULL_SCREEN = 1;
constexpr int32_t SHOW_MODE_SPLIT_PRIMARY = 2;
constexpr int32_t SHOW_MODE_SPLIT_SECONDARY = 3;
```

## 相关文档

- [GN Targets](06_GN_Targets.md)
- [编译产物说明](07_Build_Artifacts.md)
