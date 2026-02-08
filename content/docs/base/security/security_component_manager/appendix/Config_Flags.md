# 附录：配置与开关 - Security Component Manager

> 目的：了解 GN 构建系统的关键宏定义与特性开关

---

## 适用范围

本文档适用于：
- 需要自定义构建配置的开发者
- 需要启用/禁用功能的开发者
- 需要调试编译问题的开发者

---

## 关键结论

1. **主配置文件**：`security_component.gni`（feature flags）
2. **Coverage 配置**：`config/BUILD.gn`（覆盖率统计）
3. **预处理器宏**：`HILOG_ENABLE`、`SEC_COMP_SERVICE_COMPILE_ENABLE`、`SECURITY_COMPONENT_ENHANCE_ENABLE`
4. **编译变体**：CFI enabled/disabled、TDD 测试模式

---

## GN 配置文件

### 1. security_component.gni - 主配置

**证据路径**：`security_component.gni:14-21`

```gn
sec_comp_dir = "//base/security/security_component_manager"

if (!defined(global_parts_info) ||
    defined(global_parts_info.security_security_component_enhance)) {
  security_component_enhance_enable = true
} else {
  security_component_enhance_enable = false
}
```

**配置项**：

| 变量 | 类型 | 默认值 | 用途 |
|------|------|---------|------|
| `sec_comp_dir` | string | `"//base/security/security_component_manager"` | 根目录路径 |
| `security_component_enhance_enable` | bool | 根据 `global_parts_info` 决定 | 启用/禁用增强框架 |

**使用示例**：

```bash
# 启用增强功能
./build.sh --product-name rk3568 --build-variant root \
  --gn-args security_security_component_enhance=true

# 禁用增强功能（如果产品配置未包含）
./build.sh --product-name rk3568 --build-variant root
```

---

### 2. config/BUILD.gn - Coverage 配置

**证据路径**：`config/BUILD.gn`

```gn
config("coverage_flags") {
  if (defined(use_clang_coverage) && use_clang_coverage) {
    cflags_cc += [
      "-fprofile-instr-generate",
      "-fcoverage-mapping",
      "-DTDD_COVERAGE",
    ]
    ldflags += [
      "-fprofile-instr-generate",
      "-coverage",
    ]
  }
}
```

**配置项**：

| 编译选项 | 类型 | 用途 |
|---------|------|------|
| `-fprofile-instr-generate` | cflags_cc | 生成覆盖率数据（编译时） |
| `-fcoverage-mapping` | cflags_cc | 生成覆盖率映射 |
| `-DTDD_COVERAGE` | cflags_cc | 启用测试覆盖率模式 |
| `-fprofile-instr-generate` | ldflags | 链接时生成覆盖率数据 |
| `-coverage` | ldflags | 链接覆盖率库 |

**使用示例**：

```bash
# 启用覆盖率统计
./build.sh --product-name rk3568 --build-variant root \
  --gn-args use_clang_coverage=true
```

---

## 预处理器宏

### 1. HILOG_ENABLE - 日志启用

**定义位置**：
- `frameworks/inner_api/security_component/BUILD.gn:77`
- `services/security_component_service/sa/BUILD.gn:157-160`
- `services/security_component_service/sa/BUILD.gn:236-239`

**用途**：启用 HiLog 日志输出

**使用**：
```cpp
#ifdef HILOG_ENABLE
#include "hilog/log.h"

#define SC_LOG_DEBUG(label, fmt, ...) \
    (label, LOG_DEBUG, "%{public}s: " fmt, ##__VA_ARGS__)
#define SC_LOG_ERROR(label, fmt, ...) \
    (label, LOG_ERROR, "%{public}s: " fmt, ##__VA_ARGS__)
#else
// 日志禁用时的空宏
#define SC_LOG_DEBUG(label, fmt, ...)
#define SC_LOG_ERROR(label, fmt, ...)
#endif
```

---

### 2. SEC_COMP_SERVICE_COMPILE_ENABLE - 服务端编译

**定义位置**：
- `services/security_component_service/sa/BUILD.gn:239`
- `services/security_component_service/sa/BUILD.gn:240`

**用途**：标记服务端代码编译

**使用**：
```cpp
#ifdef SEC_COMP_SERVICE_COMPILE_ENABLE
// 服务端代码
class SecCompService : public SystemAbility, public SecCompServiceStub {
    // 服务端实现
};
#else
// 客户端代码
class SecCompClient {
    // 客户端实现
};
#endif
```

---

### 3. SECURITY_COMPONENT_ENHANCE_ENABLE - 增强功能启用

**定义位置**：
- `services/security_component_service/sa/BUILD.gn:241`
- `services/security_component_service/sa/BUILD.gn:242`

**用途**：启用/禁用增强框架功能

**使用**：
```cpp
#ifdef SECURITY_COMPONENT_ENHANCE_ENABLE
// 增强功能代码
int32_t SecCompEnhanceAdapter::SetEnhanceCfg(uint8_t* cfg, uint32_t cfgLen) {
    if (!isEnhanceInputHandlerInit) {
        InitEnhanceHandler(SEC_COMP_ENHANCE_INPUT_INTERFACE);
    }
    if (inputHandler != nullptr) {
        return inputHandler->SetEnhanceCfg(cfg, cfgLen);
    }
    return SC_ENHANCE_ERROR_NOT_EXIST_ENHANCE;
}
#else
// 增强功能禁用时的空实现
int32_t SecCompEnhanceAdapter::SetEnhanceCfg(uint8_t* cfg, uint32_t cfgLen) {
    return SC_ENHANCE_ERROR_NOT_EXIST_ENHANCE;
}
#endif
```

---

### 4. SECURITY_COMPONENT_ENHANCE_DISABLE - 增强功能禁用（测试用）

**定义位置**：
- `frameworks/BUILD.gn:37-39`

**用途**：测试编译时禁用增强功能（无 CFI 变体）

**使用**：
```gn
# frameworks/BUILD.gn
ohos_source_set("security_component_no_cfi_enhance_adapter_src_set") {
  cflags_cc = [
    "-DSECURITY_COMPONENT_ENHANCE_DISABLE",  # 禁用增强
  ]
}
```

---

### 5. TDD_COVERAGE - 测试覆盖率

**定义位置**：
- `frameworks/inner_api/security_component/BUILD.gn:83`
- `services/security_component_service/sa/BUILD.gn:260-262`

**用途**：标记测试编译，启用覆盖率统计

**使用**：
```cpp
#ifdef TDD_COVERAGE
// 测试覆盖率代码
extern "C" void __gcov_flush() {
    // 刷新覆盖率数据
}
#endif
```

---

## 编译变体

### 1. CFI Enabled（生产）

**目标**：
- `libsecurity_component_sdk`（CFI enabled）
- `security_component_common`（CFI enabled）
- `security_component_service`（CFI enabled）

**配置**：
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
branch_protector_ret = "pac_ret"
```

**证据路径**：`services/security_component_service/sa/BUILD.gn:135-139`

### 2. CFI Disabled（测试）

**目标**：
- `security_component_no_cfi_framework_src_set`
- `security_component_no_cfi_enhance_adapter_src_set`
- `security_component_no_cfi_enhance_sdk_src_set`
- `sec_comp_no_cfi_service_proxy`
- `sec_comp_no_cfi_service_stub`
- `sec_comp_service_stub_no_cfi`

**配置**：
```gn
# 无 sanitize 配置，移除 CFI
ohos_source_set("security_component_no_cfi_framework_src_set") {
  sources = [
    # 相同源文件
  ]
  # 无 sanitize 块
}
```

**用途**：避免 CFI 导致的测试链接问题

---

## GN 变量与宏速查表

| 变量/宏 | 类型 | 默认值 | 定义位置 | 用途 |
|----------|------|---------|----------|
| `sec_comp_dir` | string | `"//base/security/security_component_manager"` | `security_component.gni:14` | 根目录 |
| `security_component_enhance_enable` | bool | 动态 | `security_component.gni:20` | 增强功能开关 |
| `HILOG_ENABLE` | define | 启用 | 多个 BUILD.gn | 日志开关 |
| `SEC_COMP_SERVICE_COMPILE_ENABLE` | define | 启用 | `services/.../BUILD.gn:239` | 服务端编译标记 |
| `SECURITY_COMPONENT_ENHANCE_ENABLE` | define | 动态 | `services/.../BUILD.gn:241` | 增强功能开关 |
| `SECURITY_COMPONENT_ENHANCE_DISABLE` | define | 测试 | `frameworks/BUILD.gn:38` | 增强功能禁用（测试） |
| `TDD_COVERAGE` | define | 动态 | `frameworks/.../BUILD.gn:83` | 测试覆盖率 |
| `use_clang_coverage` | bool | false | `config/BUILD.gn` | 覆盖率编译开关 |
| `is_standard_system` | bool | true | `BUILD.gn:17` | 标准系统标志 |
| `use_cfi` | bool | true | 多个 BUILD.gn | CFI 开关 |

---

## 构建示例

### 1. 标准生产构建

```bash
# 标准构建（默认配置）
./build.sh --product-name rk3568 --build-variant root --build-target security_component_build_module
```

### 2. 启用增强功能的构建

```bash
# 启用增强框架
./build.sh --product-name rk3568 --build-variant root \
  --build-target security_component_build_module \
  --gn-args security_security_component_enhance=true
```

### 3. 启用覆盖率的测试构建

```bash
# 单元测试 + 覆盖率
./build.sh --product-name rk3568 --build-variant root \
  --build-target security_component_build_module_test \
  --gn-args use_clang_coverage=true
```

### 4. Fuzz 测试构建（无 CFI）

```bash
# Fuzz 测试
./build.sh --product-name rk3568 --build-variant root \
  --build-target security_component_build_fuzz_test \
  --gn-args use_cfi=false use_thin_lto=false
```

---

## 相关跳转

- [GN Targets](./05_GN_Targets.md) - 查看详细的 Target 定义
- [编译产物](./06_Build_Artifacts.md) - 查看构建输出

---

**返回 [主页](./README.md) | [导航](./SUMMARY.md)
