# 配置参数

## 编译开关

### hichecker_support_asan

| 属性 | 值 |
|------|-----|
| 文件 | `hichecker.gni:15` |
| 默认值 | `true` |
| 类型 | boolean |

**说明**: 控制是否启用 Address Sanitizer (ASan) 支持。

```gni
declare_args() {
  hichecker_support_asan = true
}
```

**影响**:
- `true`: 编译时链接 ASan 库，用于内存调试
- `false`: 关闭 ASan，正常编译

**建议**:
- 开发调试阶段: `true`
- 生产发布阶段: `false`

## 系统参数

### hiviewdfx.hichecker.{processName}

**用途**: 通过系统参数动态配置检测规则

**格式**: `hiviewdfx.hichecker.{进程名}`

**示例**:
```bash
# 设置应用进程启用 ArkUI 性能检测
param set hiviewdfx.hichecker.example_app 0x40000000000
```

**允许的规则值**:

| 规则 | 值 (十六进制) | 描述 |
|------|-------------|------|
| `RULE_CHECK_ARKUI_PERFORMANCE` | `0x40000000000` | ArkUI 性能检测 |

**证据**: `hichecker.cpp:42`

```cpp
constexpr uint64_t ALLOWED_RULE = Rule::RULE_CHECK_ARKUI_PERFORMANCE;
```

### 参数读取流程

```
GetParameter("hiviewdfx.hichecker.{进程名}")
         │
         ▼
    检查返回值长度
         │
         ▼
    解析为 uint64 (BASE 10)
         │
         ▼
    校验是否只包含 ARKUI_PERFORMANCE 位
         │
         ▼
    AddRule(rule & ALLOWED_RULE)
```

## 规则位掩码

### 规则定义

| 常量 | 位位置 | 值 | 用途 |
|------|-------|-----|------|
| `RULE_THREAD_CHECK_SLOW_PROCESS` | bit 0 | `0x1` | 线程耗时调用 |
| `RULE_CHECK_SLOW_EVENT` | bit 32 | `0x100000000` | 进程耗时事件 |
| `RULE_CHECK_ABILITY_CONNECTION_LEAK` | bit 33 | `0x200000000` | Ability 连接泄露 |
| `RULE_CHECK_ARKUI_PERFORMANCE` | bit 34 | `0x400000000` | ArkUI 性能 |
| `RULE_CAUTION_PRINT_LOG` | bit 62 | `0x4000000000000000` | 日志告警 |
| `RULE_CAUTION_TRIGGER_CRASH` | bit 63 | `0x8000000000000000` | 崩溃告警 |

### 预定义组合

**证据**: `hichecker.h:33-38`

```cpp
const uint64_t ALL_RULES = RULE_CAUTION_PRINT_LOG | RULE_CAUTION_TRIGGER_CRASH 
    | RULE_THREAD_CHECK_SLOW_PROCESS | RULE_CHECK_SLOW_EVENT 
    | RULE_CHECK_ABILITY_CONNECTION_LEAK | RULE_CHECK_ARKUI_PERFORMANCE;

const uint64_t ALL_PROCESS_RULES = RULE_CHECK_SLOW_EVENT | RULE_CHECK_ABILITY_CONNECTION_LEAK 
    | RULE_CHECK_ARKUI_PERFORMANCE;

const uint64_t ALL_THREAD_RULES = RULE_THREAD_CHECK_SLOW_PROCESS;

const uint64_t ALL_CAUTION_RULES = RULE_CAUTION_PRINT_LOG | RULE_CAUTION_TRIGGER_CRASH;
```

## 产物配置

### etc/param 配置

| 产物 | 源文件 | 安装路径 |
|------|--------|----------|
| hichecker.para | `interfaces/native/innerkits/hichecker.para` | `system/etc/param/` |
| hichecker.para.dac | `interfaces/native/innerkits/hichecker.para.dac` | `system/etc/param/` |

**证据**: `interfaces/native/innerkits/BUILD.gn:23-50`

```gni
ohos_prebuilt_etc("hichecker.para") {
  source = "hichecker.para"
  install_images = [ "system", "updater" ]
  module_install_dir = "etc/param"
  part_name = "hichecker"
}

ohos_prebuilt_etc("hichecker.para.dac") {
  source = "hichecker.para.dac"
  install_images = [ "system", "updater" ]
  module_install_dir = "etc/param"
  part_name = "hichecker"
}
```

## N-API 模块配置

### support_jsapi

**用途**: 控制 N-API 模块编译条件

**证据**: `interfaces/js/kits/napi/BUILD.gn:24`

```gni
ohos_shared_library("hicchecker") {
  if (support_jsapi) {
    // 只在 support_jsapi 为 true 时编译
  }
}
```

## 其他配置

### LOG_DOMAIN / LOG_TAG

| 配置 | 值 | 用途 |
|------|-----|------|
| LOG_DOMAIN | `0xD002D0B` | HiChecker 日志域 |
| LOG_TAG | `HICHECKER` / `HiChecker_NAPI` | 日志标签 |

**证据**:
- Native: `hichecker.cpp:38`
- N-API: `napi_hichecker.cpp:28`
