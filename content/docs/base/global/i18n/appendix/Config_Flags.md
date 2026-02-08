# 关键配置选项

## GN 编译开关

### i18n.gni 全局变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `i18n_support_ui` | `true` | 是否支持 UI 相关功能 (图形化区域选择器等) |
| `i18n_support_app_preferred_language` | `true` | 是否支持应用设置首选语言 |
| `i18n_ext_part_exists` | `false` | 扩展部分是否存在 (用于条件编译) |

### 条件编译

```gn
# frameworks/intl/BUILD.gn

# UI 支持
if (i18n_support_ui) {
  defines += [ "SUPPORT_GRAPHICS" ]
  deps += [ "ability_base:want" ]
}

# 多用户支持 (仅 PC 平台)
if (target_platform == "pc") {
  defines += [ "SUPPORT_MULTI_USER" ]
}

# ASAN 支持
if (is_asan) {
  defines += [ "SUPPORT_ASAN" ]
}

# 应用首选语言
if (i18n_support_app_preferred_language) {
  defines += [ "SUPPORT_APP_PREFERRED_LANGUAGE" ]
  external_deps += [
    "ability_runtime:app_context",
    "bundle_framework:appexecfwk_base",
    ...
  ]
}
```

## Feature Flags

### SystemCapability

- `SystemCapability.Global.I18n`: 核心 i18n 功能

### 权限

- `ohos.permission.UPDATE_CONFIGURATION`: 修改系统区域配置 (仅系统应用)

## 构建产物配置

### 产物类型

| 产物 | 构建类型 | 依赖条件 |
|------|----------|----------|
| `libintl_util.so` | `ohos_shared_library` | 总是构建 |
| `libpreferred_language.so` | `ohos_shared_library` | 总是构建 |
| `libi18n.so` | `ohos_shared_library` | `interfaces/js/kits/BUILD.gn` |
| `libintl.so` | `ohos_shared_library` | `interfaces/js/innerkits/intl/BUILD.gn` |

## 调试配置

### 日志标签

- `I18N`: i18n 模块通用日志

### 日志级别

- `HILOG_ERROR_I18N`: 错误日志
- `HILOG_WARN_I18N`: 警告日志
- `HILOG_INFO_I18N`: 信息日志
- `HILOG_DEBUG_I18N`: 调试日志 (仅 debug 版本)
