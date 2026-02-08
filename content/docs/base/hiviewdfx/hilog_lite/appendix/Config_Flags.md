# Hilog Lite 配置项与 Feature Flags

本文档汇总 hilog_lite 组件的所有可配置项和 Feature Flags。

## 配置来源

| 配置文件 | 描述 |
|----------|------|
| `bundle.json` | 组件级别配置 |
| `frameworks/mini/BUILD.gn` | 轻量系统构建配置 |
| `frameworks/featured/BUILD.gn` | 小型系统构建配置 |
| `services/apphilogcat/BUILD.gn` | 应用日志服务配置 |

---

## 组件级别配置 (bundle.json)

### 适配系统

```json
{
  "adapted_system_type": [
    "mini",     // LiteOS-M
    "small"     // LiteOS-A
  ]
}
```

### 资源占用

```json
{
  "rom": "500KB",
  "ram": "~500KB"
}
```

### Feature Flags

| 特性名 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `hilog_lite_disable_privacy_feature` | bool | false | 禁用隐私保护功能 |
| `hilog_lite_disable_hilog_static` | bool | false | 禁用静态库构建 |
| `hilog_lite_disable_js_feature` | bool | false | 禁用 JS/ACE Lite 功能 |

> 证据来源: bundle.json:24-43

---

## 轻量系统配置 (frameworks/mini/BUILD.gn)

### 构建设置 (declare_args)

| 配置项 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `hilog_lite_file_size` | int | 8192 | 日志文件大小 (字节) |
| `hilog_lite_disable_cache` | bool | false | 禁用日志缓存 |
| `hilog_lite_limit_level_default` | int | 30 | 默认限流级别 |
| `hilog_lite_disable_print_limit` | bool | false | 禁用打印限流 |
| `hilog_lite_log_static_cache_size` | int | 1024 | 静态缓存大小 (字节) |
| `hilog_lite_hiview_hilog_file_buf_size` | int | 512 | 文件缓冲区大小 (字节) |
| `hilog_lite_disable_core_init` | bool | false | 禁用核心初始化 |
| `hilog_lite_customize_implementation` | bool | false | 启用自定义实现 |

> 证据来源: frameworks/mini/BUILD.gn:16-25

### 条件编译

| 条件 | 定义 | 效果 |
|------|------|------|
| `hilog_lite_disable_cache` | `DISABLE_HILOG_CACHE` | 禁用缓存 |
| `hilog_lite_disable_print_limit` | `DISABLE_HILOG_LITE_PRINT_LIMIT` | 禁用打印限流 |
| `hilog_lite_disable_core_init` | `DISABLE_HILOG_LITE_CORE_INIT` | 不自动初始化 |

### 编译时常量

| 常量 | 值来源 | 描述 |
|------|--------|------|
| `HIVIEW_LOG_FILE_SIZE` | `hilog_lite_file_size` | 日志文件大小 |
| `LOG_LIMIT_DEFAULT` | `hilog_lite_limit_level_default` | 默认限流级别 |
| `LOG_STATIC_CACHE_SIZE` | `hilog_lite_log_static_cache_size` | 静态缓存大小 |
| `HIVIEW_HILOG_FILE_BUF_SIZE` | `hilog_lite_hiview_hilog_file_buf_size` | 缓冲区大小 |

---

## 小型系统配置 (frameworks/featured/BUILD.gn)

### 构建设置 (declare_args)

| 配置项 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `hilog_lite_disable_privacy_feature` | bool | false | 禁用隐私保护 |
| `hilog_lite_disable_hilog_static` | bool | false | 禁用静态库 |

> 证据来源: frameworks/featured/BUILD.gn:16-19

### 条件编译

| 条件 | 定义 | 效果 |
|------|------|------|
| `hilog_lite_disable_privacy_feature` | `DISABLE_HILOG_PRIVACY` | 禁用隐私标识处理 |

---

## 应用日志服务配置 (services/apphilogcat/BUILD.gn)

### 构建设置 (declare_args)

| 配置项 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `hilog_lite_hilog_file_size` | int | 1024000 | 日志文件最大大小 |
| `hilog_lite_enable_apphilogcat_init_release` | bool | false | Release 版本启用 |
| `hilog_lite_enable_apphilogcat_init_debug` | bool | true | Debug 版本启用 |
| `hilog_lite_apphilogcat_log_level_release` | int | 5 | Release 日志级别 |
| `hilog_lite_apphilogcat_log_level_debug` | int | 3 | Debug 日志级别 |
| `hilog_lite_enable_hilogcat_build` | bool | true | 启用 hilogcat 构建 |
| `hilog_lite_apphilogcat_log_dir` | string | "/storage/data/log" | 日志存储目录 |

> 证据来源: services/apphilogcat/BUILD.gn:15-24

### 状态定义

```gn
apphilogcat_on = 1
apphilogcat_off = 0
```

### 条件编译

| 条件 | 宏定义 | 效果 |
|------|--------|------|
| `hilog_lite_enable_apphilogcat_init_release` | `APPHILOGCAT_STATUS_RELEASE` | Release 服务状态 |
| `hilog_lite_enable_apphilogcat_init_debug` | `APPHILOGCAT_STATUS_DEBUG` | Debug 服务状态 |
| N/A | `CONFIG_LOG_LEVEL_RELEASE` | Release 日志级别 |
| N/A | `CONFIG_LOG_LEVEL_DEBUG` | Debug 日志级别 |
| N/A | `HILOG_DIR` | 日志目录路径 |

---

## 日志级别映射

### 轻量系统级别

| 级别名 | 值 | 描述 |
|--------|-----|------|
| `HILOG_LV_INVALID` | 0 | 无效 |
| `HILOG_LV_DEBUG` | 1 | 调试 |
| `HILOG_LV_INFO` | 2 | 信息 |
| `HILOG_LV_WARN` | 3 | 警告 |
| `HILOG_LV_ERROR` | 4 | 错误 |
| `HILOG_LV_FATAL` | 5 | 致命 |
| `HILOG_LV_MAX` | 6 | 最大值 |

### 小型系统级别

| 级别名 | 值 | 描述 |
|--------|-----|------|
| `LOG_DEBUG` | 3 | 调试 |
| `LOG_INFO` | 4 | 信息 |
| `LOG_WARN` | 5 | 警告 |
| `LOG_ERROR` | 6 | 错误 |
| `LOG_FATAL` | 7 | 致命 |

### JS 级别常量

| 常量名 | 值 | 对应 |
|--------|-----|------|
| `LogLevel.DEBUG` | 3 | LOG_DEBUG |
| `LogLevel.INFO` | 4 | LOG_INFO |
| `LogLevel.WARN` | 5 | LOG_WARN |
| `LogLevel.ERROR` | 6 | LOG_ERROR |
| `LogLevel.FATAL` | 7 | LOG_FATAL |

---

## 模块 ID 映射

### 预定义模块

| ID | 模块名 | 用途 |
|----|--------|------|
| 0 | HIVIEW | DFX 子系统 |
| 1 | SAMGR | 系统能力管理器 |
| 2 | UPDATE | 更新模块 |
| 3 | ACE | 轻量级应用框架 |
| 4 | GRAPHIC | 图形子系统 |
| 5 | APP | 第三方应用 |
| 6 | AAFWK | 原子能力框架 |
| 7 | MEDIA | 多媒体 |
| 8 | DMS | 分布式调度 |
| 9 | SEN | 传感器 |
| 10 | SCY | 安全 |
| 11 | XTS | 兼容性测试 |
| 12 | SOFTBUS | 软总线 |
| 13 | POWERMGR | 电源管理 |
| 14 | UIKIT | UI 框架 |
| 15 | GLOBAL | 全局管理 |
| 16 | DATAMGR | 数据管理 |
| 17 | INIT | 初始化 |
| 32+ | OEM_CUSTOMIZE | OEM 自定义 |

> 最大支持 64 个模块 (HILOG_MODULE_MAX = 64)

---

## 配置组合示例

### 最小化构建

```gn
# 禁用不需要的功能
hilog_lite_disable_privacy_feature = true
hilog_lite_disable_hilog_static = false
hilog_lite_disable_js_feature = true
hilog_lite_disable_cache = true
hilog_lite_disable_print_limit = true
```

### 安全敏感场景

```gn
# 启用所有安全特性
hilog_lite_disable_privacy_feature = false  # 确保隐私功能启用
hilog_lite_disable_print_limit = false     # 确保限流启用
hilog_lite_limit_level_default = 4         # 提高默认级别
```

### 调试场景

```gn
# 启用调试功能
hilog_lite_enable_apphilogcat_init_debug = true
hilog_lite_apphilogcat_log_level_debug = 3
hilog_lite_disable_print_limit = true
```

---

## 相关文档

- [概览](01_Overview.md)
- [GN Targets](05_GN_Targets.md)
- [编译产物](06_Build_Outputs.md)
- [Native API](03_Native_API.md)
