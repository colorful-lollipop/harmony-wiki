# 03 - OH 构建系统适配

## 3.1 构建配置概览

### 核心构建文件

CMSIS 库在 OpenHarmony 中的构建配置非常简洁：

| 文件 | 路径 | 功能 |
|------|------|------|
| **cmsis.gni** | `//third_party/cmsis/cmsis.gni` | 定义包含路径变量 |

### 配置内容

```gn
# cmsis.gni
# Copyright (c) 2022-2022 Huawei Device Co., Ltd. All rights reserved.

CMSIS_INCLUDE_DIRS = [
  "//third_party/cmsis/CMSIS/Core/Include",
  "//third_party/cmsis/CMSIS/RTOS2/Include",
]
```

**关键说明**：
- 仅定义包含路径变量 `CMSIS_INCLUDE_DIRS`
- 无编译目标（CMSIS 是纯头文件库）
- 无特殊编译选项（defines、flags 等）

---

## 3.2 与其他库的对比

### 典型第三方库的 BUILD.gn 结构

```gn
# 普通第三方库（如 curl）
config("public") {
  include_dirs = [ "include" ]
  defines = [ "CURL_STATICLIB", "HAVE_CONFIG_H" ]
}

static_library("curl") {
  sources = [ ... ]
  configs += [ ":public" ]
  deps = [ ... ]
}
```

### CMSIS 的特殊性

```gn
# CMSIS（纯头文件库）
# 无 static_library 目标
# 无 sources
# 无 deps
# 仅提供 include_dirs

CMSIS_INCLUDE_DIRS = [ ... ]
```

### 对比表

| 特性 | 普通第三方库 | CMSIS |
|------|--------------|-------|
| **编译产物** | 静态库/动态库 | 无（头文件库） |
| **sources** | 需要 | 不需要 |
| **deps** | 可能有 | 无 |
| **defines** | 可能有 | 无 |
| **configs** | 可能有 | 无 |

---

## 3.3 在 OH 中引入 CMSIS

### 方式一：使用 .gni 文件（推荐）

```gn
# 模块的 BUILD.gn
import("//third_party/cmsis/cmsis.gni")

static_library("my_module") {
  sources = [ "my_source.c" ]
  
  # 添加 CMSIS 包含路径
  include_dirs = CMSIS_INCLUDE_DIRS
}
```

**优点**：
- 集中管理包含路径
- 路径变更只需修改一处

### 方式二：直接指定路径

```gn
# 模块的 BUILD.gn
static_library("my_module") {
  sources = [ "my_source.c" ]
  
  # 直接指定路径（不推荐）
  include_dirs = [
    "//third_party/cmsis/CMSIS/Core/Include",
    "//third_party/cmsis/CMSIS/RTOS2/Include",
  ]
}
```

**缺点**：
- 路径分散在各模块
- 维护困难

### 实际使用示例

#### 内核适配层 (`kernel/liteos_m/kal/cmsis/BUILD.gn`)

```gn
import("//kernel/liteos_m/liteos.gni")
import("$THIRDPARTY_CMSIS_DIR/cmsis.gni")  # 使用变量引入

module_switch = defined(LOSCFG_KAL_CMSIS)
module_name = get_path_info(rebase_path("."), "name")
kernel_module(module_name) {
  sources = [ "cmsis_liteos2.c" ]
  configs += [ "$LITEOSTOPDIR:warn_config" ]
}

config("public") {
  include_dirs = CMSIS_INCLUDE_DIRS + [ "." ]  # 使用 CMSIS_INCLUDE_DIRS
}
```

#### 设备板级支持 (`device/qemu/arm_mps3_an547/liteos_m/board/BUILD.gn`)

```gn
import("//kernel/liteos_m/liteos.gni")
import("//third_party/cmsis/cmsis.gni")  # 引入 CMSIS

module_name = "bsp_config"

kernel_module(module_name) {
  asmflags = board_asmflags
  sources = [
    "driver/arm_uart_drv.c",
    "startup.s",
    # ...
  ]
}

config("public") {
  include_dirs = [
    ".",
    "include",
    # ...
  ]
  # 注意：此 config 中未直接使用 CMSIS_INCLUDE_DIRS
  # 依赖通过其他方式传递
}
```

---

## 3.4 编译器适配

### CMSIS 支持的编译器

CMSIS 头文件通过条件编译支持多种编译器：

```c
// CMSIS/Core/Include/cmsis_compiler.h

#if defined(__ARMCC_VERSION) && (__ARMCC_VERSION >= 6010050)
  // ARM Compiler 6
  #include "cmsis_armclang.h"
#elif defined(__clang__)
  // Clang
  #include "cmsis_clang.h"
#elif defined(__GNUC__)
  // GCC
  #include "cmsis_gcc.h"
#elif defined(__ICCARM__)
  // IAR
  #include "cmsis_iccarm.h"
#endif
```

### OH 使用的编译器

| 编译器 | OH 版本支持 | CMSIS 支持 |
|--------|-------------|------------|
| **Clang** | 主要编译器 | ✅ `cmsis_clang_m.h` |
| **GCC** | 部分设备支持 | ✅ `cmsis_gcc_m.h` |

### OH 特有编译器配置

```gn
# 在 SoC 或芯片配置中定义
board_cflags = [
  "-mcpu=cortex-m4",        # 指定 CPU
  "-mthumb",                # Thumb 指令集
  "-mfpu=fpv4-sp-d16",      # FPU 配置
  "-mfloat-abi=softfp",     # 浮点 ABI
]
```

**说明**：编译器标志在**使用方**定义，不在 CMSIS 库中定义。

---

## 3.5 与上游构建系统的差异

### 上游 CMSIS 的构建方式

| 方式 | 说明 | OH 是否采用 |
|------|------|-------------|
| **CMSIS-Pack** | ARM 官方包管理格式 | ❌ 否 |
| **CMake** | 社区支持的 CMake 配置 | ❌ 否 |
| **直接包含** | 直接包含头文件 | ✅ 是 |

### 为什么 OH 不使用 CMSIS-Pack？

| 原因 | 说明 |
|------|------|
| **包管理系统差异** | OH 使用 GN 构建系统，非 CMSIS-Pack |
| **简化依赖** | CMSIS-Pack 引入额外复杂度 |
| **版本控制** | OH 通过 Git 子模块/代码仓管理版本 |
| **裁剪需求** | OH 只使用 Core 和 RTOS2，不需要完整 Pack |

### OH 构建优势

```
上游 CMSIS-Pack 方式：
CMSIS Pack Installer → 解包 → 选择组件 → 集成

OH GN 方式：
import("//third_party/cmsis/cmsis.gni") → 直接使用
```

**优势**：
- 更简单直接
- 与 OH 构建系统统一
- 易于自动化

---

## 3.6 构建验证

### 编译验证命令

```bash
# 1. 编译内核（包含 CMSIS 适配层）
hb build -T kernel -f

# 2. 编译特定芯片（如 hi3861v100）
hb build -p hispark_pegasus -f

# 3. 编译 XTS 测试（验证 CMSIS 功能）
hb build -T xts -f
```

### 验证要点

| 检查项 | 验证方法 |
|--------|----------|
| **头文件包含** | 编译无 "file not found" 错误 |
| **API 可用性** | `cmsis_liteos2.c` 编译通过 |
| **内联函数** | 链接无未定义符号错误 |
| **寄存器访问** | 芯片启动代码正常执行 |

---

## 3.7 常见问题

### Q1: 为什么 CMSIS 没有 BUILD.gn？

**A**: CMSIS 是纯头文件库，不需要编译。通过 `cmsis.gni` 提供包含路径即可。

### Q2: 如何添加 CMSIS-DSP 或 CMSIS-NN？

**A**: 这些是独立仓库，需要单独引入：
- 从 [ARM-software/CMSIS-DSP](https://github.com/ARM-software/CMSIS-DSP) 引入
- 创建对应的 `BUILD.gn` 进行编译配置
- 当前 OH 的 CMSIS 只包含 Core 和 RTOS2

### Q3: 不同芯片使用不同的 core_cm*.h？

**A**: 是的，根据芯片的 Cortex-M 版本选择：
- Cortex-M0/M0+: `core_cm0.h`, `core_cm0plus.h`
- Cortex-M3: `core_cm3.h`
- Cortex-M4: `core_cm4.h`
- Cortex-M7: `core_cm7.h`
- Cortex-M23: `core_cm23.h`
- Cortex-M33: `core_cm33.h`

选择由芯片配置文件中的 `-mcpu` 标志决定，编译器会自动包含正确的头文件。

---

## 3.8 配置参考

### 典型芯片配置示例

#### Cortex-M4 (Hi3861V100)

```gn
# device/soc/hisilicon/hi3861v100/BUILD.gn

config("chip_config") {
  cflags = [
    "-mcpu=cortex-m4",
    "-mthumb",
    "-mfpu=fpv4-sp-d16",
    "-mfloat-abi=softfp",
  ]
  
  include_dirs = [
    "//third_party/cmsis/CMSIS/Core/Include",
    "//third_party/cmsis/CMSIS/RTOS2/Include",
  ]
  
  defines = [
    "__CORTEX_M4",           # 通知 CMSIS 使用 CM4 配置
    "__FPU_PRESENT=1",       # 启用 FPU 支持
  ]
}
```

#### Cortex-M33 (WS63V100)

```gn
# device/soc/hisilicon/ws63v100/BUILD.gn

config("chip_config") {
  cflags = [
    "-mcpu=cortex-m33",
    "-mthumb",
    "-mfpu=fpv5-sp-d16",
    "-mfloat-abi=softfp",
    "-mcmse",                # ARMv8-M 安全扩展
  ]
  
  defines = [
    "__CORTEX_M33",
    "__FPU_PRESENT=1",
    "__ARMV8M_MAINLINE__=1",
  ]
}
```

---

## 3.9 总结

### 构建系统特点

| 特性 | 说明 |
|------|------|
| **简洁** | 仅一个 `cmsis.gni` 文件 |
| **无编译** | 纯头文件库，无编译目标 |
| **标准化** | 使用 `CMSIS_INCLUDE_DIRS` 变量统一引入 |
| **灵活** | 编译器配置在使用方定义 |

### 最佳实践

1. **引入方式**：使用 `import("//third_party/cmsis/cmsis.gni")`
2. **包含路径**：使用 `CMSIS_INCLUDE_DIRS` 变量
3. **编译器配置**：在芯片/SoC 层定义 `-mcpu` 等选项
4. **宏定义**：在需要时定义 `__CORTEX_Mx` 和 `__FPU_PRESENT`

---

*文档版本: 1.0*  
*最后更新: 2025-02-08*
