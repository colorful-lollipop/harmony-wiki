# 关键配置与宏

> 本文档描述 Form Fwk 的关键配置开关、宏定义与 Feature Flags

## GN 配置变量

**配置文件**: `form_fwk.gni`

### 卡片尺寸配置

| 变量 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `form_fwk_form_dimension_2_3` | bool | false | 启用 2x3 卡片尺寸 |
| `form_fwk_form_dimension_3_3` | bool | false | 启用 3x3 卡片尺寸 |

### 功能开关

| 变量 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `form_fwk_watch_api_disable` | bool | false | 禁用 Watch API |
| `form_fwk_dynamic_support` | bool | false | 启用动态 SA 支持 |

### 系统集成开关

| 变量 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `device_usage_statistics` | bool | true | 设备使用统计 |
| `cite_memmgr` | bool | false | 内存管理引用 |
| `res_schedule_service` | bool | false | 资源调度服务 |
| `theme_mgr_enable` | bool | false | 主题管理器 |
| `form_runtime_power` | bool | true | 电源管理支持 |
| `hiappevent_global_part_enabled` | bool | false | HiAppEvent 全局 |
| `device_manager_enable` | bool | false | 设备管理器 |

## 编译宏定义

### 日志标签

```cpp
// 文件: interfaces/inner_api/include/fms_log_wrapper.h
#define FMS_LOG_TAG "FormManagerService"
```

### Feature Flags (BUILD.gn 中定义)

| 宏 | 定义位置 | 功能 |
|----|----------|------|
| `FORM_DIMENSION_2_3` | `frameworks/js/napi/BUILD.gn:252` | 2x3 尺寸支持 |
| `FORM_DIMENSION_3_3` | `frameworks/js/napi/BUILD.gn:255` | 3x3 尺寸支持 |
| `WATCH_API_DISABLE` | `form_fwk.gni:53` | 禁用 Watch API |
| `SUPPORT_POWER` | `BUILD.gn:360` | 电源管理支持 |
| `DEVICE_USAGE_STATISTICS_ENABLE` | `BUILD.gn:366` | 设备使用统计 |
| `RES_SCHEDULE_ENABLE` | `BUILD.gn:375` | 资源调度 |
| `FORM_EVENT_FOR_TEST` | `BUILD.gn:379` | 测试事件 |
| `THEME_MGR_ENABLE` | `BUILD.gn:356` | 主题管理 |
| `MEM_MGR_ENABLE` | `BUILD.gn:351` | 内存管理 |
| `NO_RUNTIME_EMULATOR` | `BUILD.gn:452` | 无运行时模拟器 |

## SA ID 常量

```cpp
// Form Manager Service SA ID
constexpr int32_t FORM_MGR_SERVICE_ID = 403;
```

## 权限常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `PERMISSION_INTERACT_ACROSS_LOCAL_ACCOUNTS` | - | 跨本地账户权限 |
| `ohos.permission.USE_DATA_FORM` | - | 使用卡片数据权限 |

## 错误码常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `ERR_FORM_INVALID_FORM_ID` | 16500001 | 无效 formId |
| `ERR_FORM_INVALID_PARAMETER` | 16500002 | 无效参数 |
| `ERR_FORM_PERMISSION_DENIED` | 16500050 | 权限拒绝 |

## CFI 安全配置

**配置文件**: `BUILD.gn`

```cpp
sanitize = {
    cfi = true;              // 启用 CFI
    cfi_cross_dso = true;    // 跨 DSO CFI 检查
    debug = false;           // 生产模式
}
```
