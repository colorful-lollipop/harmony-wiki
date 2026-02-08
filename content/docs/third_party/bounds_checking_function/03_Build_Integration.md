# bounds_checking_function OpenHarmony 构建适配

## 1. BUILD.gn 结构总览

### 1.1 文件位置
```
third_party/bounds_checking_function/
├── BUILD.gn           # 主构建文件
├── libsec_src.gni     # 源文件列表
└── include/
    ├── securec.h      # 主头文件
    └── securectype.h  # 类型定义头文件
```

### 1.2 构建目标

BUILD.gn 定义了以下构建目标：

| 目标名 | 类型 | 说明 |
|--------|------|------|
| `libsec_static` | static_library | 静态库，用于独立组件 |
| `libsec_shared` | shared_library | 动态库，系统主要使用 |
| `libsec_public_config` | config | 公共头文件路径配置 |

---

## 2. 多系统类型支持

### 2.1 条件编译结构

```gn
if (defined(ohos_lite)) {
  # mini/small 系统构建逻辑
  if (current_toolchain == "//build/toolchain/linux:clang_x64" ||
      ohos_kernel_type != "liteos_m") {
    # 标准工具链构建
  } else if (ohos_kernel_type == "liteos_m") {
    # liteos_m 内核特殊处理
  }
} else {
  # standard 系统构建逻辑
}
```

### 2.2 Lite 系统适配 (ohos_lite)

#### 静态库目标
```gn
lite_library("libsec_static") {
  target_type = "static_library"
  sources = libsec_sources
  public_configs = [ ":libsec_public_config" ]
}
```

#### 动态库目标
```gn
lite_library("libsec_shared") {
  target_type = "shared_library"
  sources = libsec_sources
  public_configs = [ ":libsec_public_config" ]
}
```

#### LiteOS-M 特殊处理
```gn
else if (ohos_kernel_type == "liteos_m") {
  group("libsec_static") {
    # 空 group，使用 kernel/liteos_m/kal/libsec/BUILD.gn 编译
  }
}
```
**说明**: LiteOS-M 内核使用自己的安全函数实现，不依赖本库

### 2.3 标准系统适配

#### 静态库
```gn
ohos_static_library("libsec_static") {
  sources = libsec_sources
  public_configs = [ ":libsec_public_config" ]
  
  cflags = [
    "-D_INC_STRING_S",
    "-D_INC_WCHAR_S",
    "-D_SECIMP=//",
    "-D_STDIO_S_DEFINED",
    "-D_INC_STDIO_S",
    "-D_INC_STDLIB_S",
    "-D_INC_MEMORY_S",
  ]
  
  part_name = "bounds_checking_function"
  subsystem_name = "thirdparty"
}
```

#### 动态库
```gn
ohos_shared_library("libsec_shared") {
  sources = libsec_sources
  public_configs = [ ":libsec_public_config" ]
  
  branch_protector_ret = "pac_ret"  # PAC 保护
  
  cflags = [
    "-D_INC_STRING_S",
    "-D_INC_WCHAR_S",
    "-D_SECIMP=//",
    "-D_STDIO_S_DEFINED",
    "-D_INC_STDIO_S",
    "-D_INC_STDLIB_S",
    "-D_INC_MEMORY_S",
  ]
  
  innerapi_tags = [
    "chipsetsdk_sp",
    "platformsdk",
    "sasdk",
  ]
  
  part_name = "bounds_checking_function"
  subsystem_name = "thirdparty"
  
  install_images = [
    "system",
    "updater",
    "ramdisk",
  ]
}
```

---

## 3. 关键编译选项详解

### 3.1 宏定义 (cflags)

| 宏 | 说明 | 影响 |
|----|------|------|
| `-D_INC_STRING_S` | 启用安全字符串函数声明 | 暴露 `strcpy_s`, `strcat_s` 等 |
| `-D_INC_WCHAR_S` | 启用宽字符安全函数声明 | 暴露 `wcscpy_s`, `wcscat_s` 等 |
| `-D_SECIMP=//` | 安全实现标记 | 影响函数导出属性 |
| `-D_STDIO_S_DEFINED` | 标准 I/O 安全函数定义 | 启用格式化 I/O 函数 |
| `-D_INC_STDIO_S` | 包含 stdio_s 头文件 | `sprintf_s`, `scanf_s` 等 |
| `-D_INC_STDLIB_S` | 包含 stdlib_s 头文件 | 安全标准库函数 |
| `-D_INC_MEMORY_S` | 包含 memory_s 头文件 | `memcpy_s`, `memset_s` 等 |

### 3.2 PAC 安全编译选项

```gn
branch_protector_ret = "pac_ret"
```

**技术细节**:
- **全称**: Pointer Authentication Code for Return addresses
- **架构**: ARMv8.3-A 及以上
- **原理**: 使用专用密钥对返回地址签名，返回前验证签名
- **防护**: ROP (Return-Oriented Programming) 攻击
- **兼容性**: 非 ARM64 架构自动忽略

**配置对比**:
| 选项值 | 说明 |
|--------|------|
| `"none"` | 禁用保护 |
| `"standard"` | 标准返回保护 |
| `"pac_ret"` | PAC 返回保护（最强） |

### 3.3 与上游的差异

#### 上游 libboundscheck Makefile
```makefile
# 上游使用简单 Makefile
CC = gcc
CFLAGS = -fstack-protector-strong -O2 -D_FORTIFY_SOURCE=2
```

#### OpenHarmony BUILD.gn
```gn
# OH 使用 GN 构建系统，配置更丰富
cflags = [
  # 功能宏定义
  "-D_INC_STRING_S",
  # ... 其他宏
  # 安全编译选项通过 branch_protector_ret 单独配置
]
branch_protector_ret = "pac_ret"
```

**关键差异**:
1. **构建系统** - 上游用 Makefile，OH 用 GN
2. **安全选项** - OH 额外使能 PAC 保护
3. **多目标** - OH 支持静态/动态库多目标
4. **多系统** - OH 支持 lite/standard 多系统类型

---

## 4. libsec_src.gni 源文件配置

### 4.1 包含目录
```gn
libsec_include_dirs = [ "//third_party/bounds_checking_function/include" ]
```

### 4.2 源文件列表（40 个 C 文件）

```gn
libsec_sources = [
  # 内存操作 (5 个)
  "//third_party/bounds_checking_function/src/memcpy_s.c",
  "//third_party/bounds_checking_function/src/memmove_s.c",
  "//third_party/bounds_checking_function/src/memset_s.c",
  "//third_party/bounds_checking_function/src/wmemcpy_s.c",
  "//third_party/bounds_checking_function/src/wmemmove_s.c",
  
  # 字符串操作 (5 个)
  "//third_party/bounds_checking_function/src/strcpy_s.c",
  "//third_party/bounds_checking_function/src/strncpy_s.c",
  "//third_party/bounds_checking_function/src/strcat_s.c",
  "//third_party/bounds_checking_function/src/strncat_s.c",
  "//third_party/bounds_checking_function/src/strtok_s.c",
  
  # 宽字符字符串 (5 个)
  "//third_party/bounds_checking_function/src/wcscpy_s.c",
  "//third_party/bounds_checking_function/src/wcsncpy_s.c",
  "//third_party/bounds_checking_function/src/wcscat_s.c",
  "//third_party/bounds_checking_function/src/wcsncat_s.c",
  "//third_party/bounds_checking_function/src/wcstok_s.c",
  
  # 格式化输出 (8 个)
  "//third_party/bounds_checking_function/src/sprintf_s.c",
  "//third_party/bounds_checking_function/src/snprintf_s.c",
  "//third_party/bounds_checking_function/src/vsprintf_s.c",
  "//third_party/bounds_checking_function/src/vsnprintf_s.c",
  "//third_party/bounds_checking_function/src/swprintf_s.c",
  "//third_party/bounds_checking_function/src/vswprintf_s.c",
  "//third_party/bounds_checking_function/src/secureprintoutput_a.c",
  "//third_party/bounds_checking_function/src/secureprintoutput_w.c",
  
  # 格式化输入 (10 个)
  "//third_party/bounds_checking_function/src/sscanf_s.c",
  "//third_party/bounds_checking_function/src/scanf_s.c",
  "//third_party/bounds_checking_function/src/vscanf_s.c",
  "//third_party/bounds_checking_function/src/vsscanf_s.c",
  "//third_party/bounds_checking_function/src/fscanf_s.c",
  "//third_party/bounds_checking_function/src/vfscanf_s.c",
  "//third_party/bounds_checking_function/src/wscanf_s.c",
  "//third_party/bounds_checking_function/src/vwscanf_s.c",
  "//third_party/bounds_checking_function/src/fwscanf_s.c",
  "//third_party/bounds_checking_function/src/vfwscanf_s.c",
  "//third_party/bounds_checking_function/src/swscanf_s.c",
  "//third_party/bounds_checking_function/src/vswscanf_s.c",
  
  # 输入 (1 个)
  "//third_party/bounds_checking_function/src/gets_s.c",
  
  # 工具 (4 个)
  "//third_party/bounds_checking_function/src/securecutil.c",
  "//third_party/bounds_checking_function/src/secureinput_a.c",
  "//third_party/bounds_checking_function/src/secureinput_w.c",
]
```

---

## 5. Inner API 与 SDK 分层

### 5.1 innerapi_tags 配置

```gn
innerapi_tags = [
  "chipsetsdk_sp",    # 芯片平台 SDK (Special)
  "platformsdk",      # 平台 SDK
  "sasdk",            # SA (System Ability) SDK
]
```

### 5.2 层级说明

```
┌─────────────────────────────────────────────┐
│  chipsetsdk_sp (芯片厂商 SDK)               │
│  - 芯片厂商定制开发                          │
│  - 访问底层硬件接口                          │
│  - 最高权限级别                              │
├─────────────────────────────────────────────┤
│  platformsdk (平台 SDK)                     │
│  - 系统服务开发                              │
│  - 框架层开发                                │
│  - 中等权限级别                              │
├─────────────────────────────────────────────┤
│  sasdk (SA SDK)                             │
│  - System Ability 开发                       │
│  - 应用层服务开发                            │
│  - 标准权限级别                              │
└─────────────────────────────────────────────┘
```

### 5.3 依赖方式

不同层级的模块使用不同的依赖方式：

```gn
# chipsetsdk_sp 层级
deps = [ "//third_party/bounds_checking_function:libsec_shared" ]

# platformsdk 层级  
external_deps = [ "bounds_checking_function:libsec_shared" ]

# sa sdk 层级
external_deps = [ "bounds_checking_function:libsec_shared" ]
```

---

## 6. 安装镜像配置

### 6.1 install_images

```gn
install_images = [
  "system",      # 主系统镜像
  "updater",     # 升级程序镜像
  "ramdisk",     # 内存盘镜像
]
```

### 6.2 生命周期覆盖

```
启动阶段          镜像              bounds_checking_function 作用
─────────────────────────────────────────────────────────────────
Bootloader       -                 (不涉及)
Kernel           -                 (内核使用 kal/libsec)
Early Userspace  ramdisk           基础系统工具使用
Updater          updater           系统升级程序使用
Main System      system            完整系统功能使用
```

**重要性**: 需要在所有启动阶段可用，说明这是系统最基础的库之一

---

## 7. 特殊使用场景

### 7.1 直接源码引用

部分项目（如 smartperf_host）直接引用源码而非库：

```gn
# developtools/smartperf_host/.../BUILD.gn
ohos_source_set("libsec_static") {
  sources = [
    "${THIRD_PARTY}/bounds_checking_function/src/fscanf_s.c",
    "${THIRD_PARTY}/bounds_checking_function/src/memcpy_s.c",
    # ... 所有源文件
  ]
  include_dirs = [ "${THIRD_PARTY}/bounds_checking_function/include" ]
}
```

**使用场景**:
- 需要静态链接所有依赖的工具程序
- 跨平台工具（如 host 端工具）
- 最小化依赖的独立程序

### 7.2 头文件直接引用

```gn
include_dirs = [ "//third_party/bounds_checking_function/include" ]
```

部分项目仅引用头文件，链接时通过其他方式获得实现：
- 系统启动早期代码
- 与 libc 紧密集成的模块

---

## 8. 构建依赖关系

### 8.1 bundle.json 配置

```json
{
  "build": {
    "sub_component": [
      "//third_party/bounds_checking_function:libsec_shared"
    ],
    "inner_kits": [
      {
        "name": "//third_party/bounds_checking_function:libsec_shared",
        "header": {
          "header_files": ["securec.h", "securectype.h"],
          "header_base": "//third_party/bounds_checking_function/include"
        }
      },
      {
        "name": "//third_party/bounds_checking_function:libsec_static",
        "header": {
          "header_files": ["securec.h", "securectype.h"],
          "header_base": "//third_party/bounds_checking_function/include"
        }
      }
    ]
  }
}
```

### 8.2 构建时序

```
构建阶段 1: bounds_checking_function
  ├── libsec_shared (动态库)
  └── libsec_static (静态库)
  
构建阶段 2: 依赖模块（并行）
  ├── c_utils
  ├── arkcompiler/runtime_core
  ├── developtools/hdc
  ├── ... (1941+ 个模块)
```

---

## 9. 与标准 C 库的共存

### 9.1 命名空间隔离

安全函数使用 `_s` 后缀，与标准 C 函数区分：
- `strcpy` (标准) vs `strcpy_s` (安全)
- `memcpy` (标准) vs `memcpy_s` (安全)

### 9.2 头文件隔离

```c
// 标准 C 库
#include <string.h>

// 安全 C 库
#include "securec.h"
```

### 9.3 混合使用策略

建议的编码规范：
```c
// ✅ 推荐：始终使用安全函数
#include "securec.h"
errno_t ret = strcpy_s(dest, sizeof(dest), src);
if (ret != EOK) {
    // 错误处理
}

// ❌ 避免：使用不安全函数
#include <string.h>
strcpy(dest, src);  // 危险！可能导致缓冲区溢出
```

---

## 10. 总结

### 10.1 OH 构建适配要点

1. **多系统支持** - 通过 `ohos_lite` 条件同时支持 mini/small/standard 系统
2. **安全增强** - 使能 PAC 保护提升运行时安全性
3. **SDK 分层** - 通过 `innerapi_tags` 支持三层 SDK 架构
4. **全生命周期** - 安装到 system/updater/ramdisk 三个镜像
5. **灵活引用** - 支持库依赖和直接源码引用两种方式

### 10.2 与上游的主要差异

| 方面 | 上游 libboundscheck | OpenHarmony |
|------|---------------------|-------------|
| 构建系统 | Makefile | GN |
| 多系统支持 | 单一 Linux | mini/small/standard |
| 安全编译 | 基础栈保护 | PAC + 栈保护 |
| 库类型 | 动态库为主 | 静态+动态 |
| SDK 分层 | 无 | 三层架构 |

### 10.3 使用建议

- **应用开发者**: 通过 `external_deps` 引用 `bounds_checking_function:libsec_shared`
- **系统服务开发**: 推荐使用安全函数替代所有标准 C 字符串/内存函数
- **芯片厂商**: 注意 `chipsetsdk_sp` 标签的权限级别
