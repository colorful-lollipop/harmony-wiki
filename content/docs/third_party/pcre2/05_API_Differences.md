# 05 - API/接口差异

## 概述

OpenHarmony 中的 PCRE2 保留了原生的 C API，没有新增或删除函数。主要的差异体现在**行为变更**（通过 Patch 实现）和**编译选项**上。

## API 兼容性

### 原生 API 保持

OH 中的 PCRE2 完全保留上游 API：

| API 类别 | 函数示例 | 状态 |
|---------|---------|------|
| **编译** | `pcre2_compile()`, `pcre2_compile_context_create()` | ✅ 完全兼容 |
| **匹配** | `pcre2_match()`, `pcre2_dfa_match()` | ✅ 完全兼容 |
| **替换** | `pcre2_substitute()` | ✅ 完全兼容 |
| **JIT** | `pcre2_jit_compile()`, `pcre2_jit_match()` | ✅ 完全兼容 |
| **辅助** | `pcre2_get_error_message()`, `pcre2_match_data_create()` | ✅ 完全兼容 |

### 头文件

```c
// 标准头文件，与上游完全一致
#include "pcre2.h"

// 版本信息（10.46）
#define PCRE2_MAJOR           10
#define PCRE2_MINOR           46
#define PCRE2_DATE            2025-08-27
```

## 行为差异

### 1. 换行符识别（核心差异）

这是 OH Patch 引入的唯一行为差异。

#### 差异对比

| 换行符 | ASCII | 原生 PCRE2 | OH 静态库<br/>(ArkCompiler) | OH 共享库<br/>(SELinux等) |
|-------|-------|-----------|---------------------------|------------------------|
| LF | `0x0A` | ✅ | ✅ | ✅ |
| CR | `0x0D` | ✅ | ✅ | ✅ |
| VT | `0x0B` | ✅ | ❌ | ✅ |
| FF | `0x0C` | ✅ | ❌ | ✅ |
| NEL | `0x85` | ✅ | ❌ | ✅ |
| LS | `0x2028` | ✅ | ✅ | ✅ |
| PS | `0x2029` | ✅ | ✅ | ✅ |

#### 影响范围

此差异仅在以下情况体现：

1. **使用 `PCRE2_NEWLINE_ANY` 标志**（或相应编译上下文设置）
2. **且使用静态库**（`libpcre2_static` 或 `libpcre2_static_16`）

**不影响**的情况：
- 使用 `PCRE2_NEWLINE_LF`、`PCRE2_NEWLINE_CR`、`PCRE2_NEWLINE_CRLF` 等固定换行符模式
- 使用共享库 `libpcre2`

#### 代码示例

```c
// 差异示例代码
#include "pcre2.h"
#include <stdio.h>

int main() {
    PCRE2_SPTR pattern = (PCRE2_SPTR)".$";
    PCRE2_SPTR subject = (PCRE2_SPTR)"ab\x0B";  // "ab" + VT
    
    int errorcode;
    PCRE2_SIZE erroroffset;
    
    // 使用 PCRE2_NEWLINE_ANY
    pcre2_code* re = pcre2_compile(
        pattern, PCRE2_ZERO_TERMINATED,
        PCRE2_NEWLINE_ANY | PCRE2_MULTILINE,  // 关键：ANY 模式
        &errorcode, &erroroffset, NULL
    );
    
    pcre2_match_data* match_data = pcre2_match_data_create_from_pattern(re, NULL);
    int rc = pcre2_match(re, subject, 3, 0, 0, match_data, NULL);
    
    // 结果差异：
    // - 原生 PCRE2: rc = 1 (匹配成功，VT 前匹配)
    // - OH 静态库: rc = PCRE2_ERROR_NOMATCH (VT 不被视为换行符)
    
    printf("Match result: %d\n", rc);
    return 0;
}
```

### 2. 编译选项差异

不同目标启用的功能有所不同：

| 功能 | 原生 PCRE2<br/>(默认) | OH 共享库<br/>(libpcre2) | OH 静态库<br/>(libpcre2_static) |
|-----|-------------------|------------------------|------------------------------|
| **Unicode** | 可选 | ❌ 禁用 | ✅ 启用 |
| **JIT** | 可选 | ✅ 启用 | ✅ 启用 |
| **8-bit** | ✅ | ✅ | ✅ |
| **16-bit** | 可选 | ❌ | ✅ |
| **32-bit** | 可选 | ❌ | ❌ |

#### 影响

**Unicode 支持**:
```c
// 在 OH 共享库中，以下调用可能失败
pcre2_compile(pattern, PCRE2_UTF, ...);  // 可能返回 PCRE2_ERROR_UTF_NOT_SUPPORTED

// 在 OH 静态库中，上述调用正常工作
```

**实际影响**: 
- SELinux 不使用 Unicode 特性，无影响
- ArkCompiler 使用静态库，功能完整

## 配置差异

### config.h 差异

OH 使用预生成的 `config.h.generic`，与原生 configure 生成的配置对比：

| 宏 | 原生 (典型) | OH config.h.generic | 说明 |
|---|------------|---------------------|-----|
| `SUPPORT_UNICODE` | 1 | 0 | 共享库未启用 |
| `SUPPORT_JIT` | 1 | 1 | 始终启用 |
| `SUPPORT_PCRE2_8` | 1 | 1 | 始终启用 |
| `SUPPORT_PCRE2_16` | 0/1 | 0 | 共享库未启用 |
| `SUPPORT_PCRE2_32` | 0/1 | 0 | 未启用 |
| `HAVE_MEMMOVE` | 1 | 0 | 运行时检测 |
| `PCRE2_CODE_UNIT_WIDTH` | - | 未定义 | 编译时指定 |

**注意**: OH 静态库通过 `cflags` 覆盖部分配置：
```gn
cflags = [
  "-DSUPPORT_UNICODE=1",  # 启用 Unicode
]
```

## 使用建议

### 对于 ArkCompiler 开发者

无需关心差异，ArkCompiler 使用静态库，功能完整且行为符合 ECMAScript 规范。

### 对于 SELinux 开发者

使用原生 PCRE2 行为，注意共享库不支持 Unicode。

```c
// 避免使用 Unicode 特性
pcre2_compile(pattern, 0, ...);  // 不使用 PCRE2_UTF

// 如果需要 Unicode 支持，考虑静态链接
```

### 对于仓颉开发者

使用原生 PCRE2 行为，当前共享库配置。

### 对于应用开发者

**ArkTS/ETS 应用**:
```typescript
// 标准 ECMAScript 正则表达式，无需特殊处理
let regex = /pattern/flags;
```

**仓颉应用**:
```cangjie
// 使用仓颉标准库的 regex 模块
import std.regex
```

## 升级注意事项

### 版本升级检查清单

- [ ] 检查 API 变更（通常向后兼容）
- [ ] 验证 Patch 能否正确应用
- [ ] 测试 ArkTS/ETS 正则表达式行为
- [ ] 测试 SELinux 策略编译
- [ ] 测试仓颉正则表达式功能

### ABI 兼容性

| 版本 | ABI 兼容 | 说明 |
|-----|---------|-----|
| 10.40 → 10.46 | ✅ | 向后兼容 |
| 10.46 → 未来 | ⚠️ | 需验证 |

---

> **下一步**: 了解安全分析和 CVE 信息，请参阅 [06_Security.md](./06_Security.md)。
