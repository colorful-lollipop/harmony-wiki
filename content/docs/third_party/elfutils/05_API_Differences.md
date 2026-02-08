# API 差异分析

---

## 概述

**状态**: OpenHarmony 与上游 elfutils **无 API 差异**

**说明**: OH 主要是通过构建系统迁移来适配 elfutils，未对 API 进行修改或扩展。

---

## API 兼容性

### 库接口

| 库 | OH 版本 | 上游版本 | API 差异 |
|-----|---------|---------|---------|
| libelf | 0.193 | 0.193 | ❌ 无 |
| libdw | 0.193 | 0.193 | ❌ 无 |
| libdwfl | 0.193 | 0.193 | ❌ 无 |
| libebl | 0.193 | 0.193 | ❌ 无 |
| libdwelf | 0.193 | 0.193 | ❌ 无 |
| libdwfl_stacktrace | 0.193 | 0.193 | ❌ 无 |

### 头文件

所有公共头文件保持上游原始版本：

- `libelf/libelf.h` - ELF 操作接口
- `libdw/libdw.h` - DWARF 接口
- `libdw/dwarf.h` - DWARF 常量和类型
- `libdwfl/libdwfl.h` - 高级 DWARF 接口
- `libdwelf/libdwelf.h` - ELF 工具接口
- `libdwfl_stacktrace/libdwfl_stacktrace.h` - 堆栈跟踪接口（0.193 新增）

---

## 新增 API（0.193 上游）

### libdw 新增函数

#### dwarf_language()

**声明**：
```c
int dwarf_language (unsigned int lang, const char **namep);
```

**功能**：将 DWARF 语言编码转换为可读名称

**参数**：
- `lang`: DWARF 语言编码（如 `DW_LANG_C`, `DW_LANG_C_plus_plus`）
- `namep`: 返回语言名称字符串

**返回值**：
- 0 成功
- -1 失败

**OH 状态**：✅ 已包含

#### dwarf_language_lower_bound()

**声明**：
```c
unsigned int dwarf_language_lower_bound (bool *standardp);
```

**功能**：返回最小的 DWARF 语言编码值

**OH 状态**：✅ 已包含

### libdwfl_stacktrace 新增库（实验性）

#### Dwflst_Process_Tracker

**声明**：
```c
typedef struct Dwflst_Process_Tracker Dwflst_Process_Tracker;
```

**功能**：跟踪和缓存跨多个 libdwfl 会话的 Elf 结构

#### Dwflst_Process_Tracker 初始化

```c
Dwflst_Process_Tracker *
dwflst_process_tracker_init (void);
```

**功能**：初始化新的进程跟踪器

**OH 状态**：✅ 已包含，但暂无 OH 模块使用

---

## 行为变更的 API

### 无变更

**说明**: OH 未修改任何 API 的行为。

### 配置差异

| 配置项 | 上游默认 | OH 配置 | 影响 |
|--------|---------|---------|------|
| HAVE_CONFIG_H | configure 生成 | 预生成 | 仅构建系统差异，不影响 API |
| USE_ZLIB | 可选 | ✅ 启用 | ELF section 压缩支持 |
| USE_LZMA | 可选 | ❌ 禁用 | 不影响 API |
| USE_BZLIB | 可选 | ❌ 禁用 | 不影响 API |
| USE_ZSTD | 可选 | ❌ 禁用 | 不影响 API |

---

## 废弃或禁用的功能

### 工具链（未编译）

**上游工具**（OH 不编译）：

| 工具 | 功能 | OH 状态 |
|------|------|--------|
| eu-readelf | 显示 ELF 信息 | ❌ 不编译 |
| eu-nm | 列出符号 | ❌ 不编译 |
| eu-objdump | 显示对象文件信息 | ❌ 不编译 |
| eu-addr2line | 地址转换 | ❌ 不编译 |
| eu-strip | 剥离调试符号 | ❌ 不编译 |
| eu-unstrip | 恢复调试符号 | ❌ 不编译 |
| eu-stack | 堆栈跟踪 | ❌ 不编译 |
| eu-elflint | ELF 校验 | ❌ 不编译 |
| eu-elfcompress | ELF 压缩 | ❌ 不编译 |

**原因**：
- OH 仅需要库功能
- 不包含命令行工具
- 减少攻击面

### debuginfod 守护进程（未编译）

**功能**：HTTP 服务器，提供 debuginfo 文件下载

**OH 状态**：❌ 不编译

**原因**：
- OH 不需要网络 debuginfo 服务
- debuginfo 通常离线管理

---

## 条件编译差异

### 平台宏使用

**OH 使用标准 Linux 宏**：

```c
#if defined(__linux__)
  // Linux 特定代码
#endif

#if defined(__aarch64__)
  // ARM64 特定代码
#endif
```

**无 OH 特有宏**：
- ❌ 无 `OHOS` 宏
- ❌ 无 `OPENHARMONY` 宏
- ❌ 无 OH 特定的条件编译

### 架构支持

**OH 主要支持的架构**：

| 架构 | 宏 | 后端文件 |
|------|-----|---------|
| aarch64 | `__aarch64__` | backends/aarch64_*.c |
| arm | `__arm__` | backends/arm_*.c |
| riscv64 | `__riscv` && __riscv_xlen == 64 | backends/riscv64_*.c |
| x86_64 | `__x86_64__` | backends/x86_64_*.c |

**其他支持架构**：
- alpha, bpf, csky, ia64, m68k, ppc, ppc64, s390, s390x, sh, sparc, sparc64, hexagon, mips

---

## 实验性 API

### libdwfl_stacktrace

**状态**：实验性（Experimental）

**上游说明**（libdwfl_stacktrace.h）：
```c
/*
 * XXX: This is an experimental initial version of API, and is
 * liable to change in future releases of elfutils, especially as
 * we figure out how to generalize work to other sample data
 * formats in addition to perf_events.
 */
```

**OH 使用情况**：
- ✅ 已编译到 `libdw_static`
- ❌ 无 OH 模块使用

**未来考虑**：
- 如果 OH 需要性能分析工具，可能使用此库
- 需要关注上游 API 变化

---

## 数据类型差异

### 无变更

所有数据类型与上游一致：

- `Elf` - ELF 文件句柄
- `Dwarf` - DWARF 调试信息句柄
- `Dwfl` - DWARF 文件句柄
- `Dwarf_Die` - DWARF 信息条目
- `Elf_Scn` - ELF 节
- `GElf_Ehdr` - 通用 ELF 头

---

## 错误码差异

### 无变更

错误码与上游一致：

- **libelf 错误**：`ELF_E_*` 宏
- **libdw 错误**：`DWARF_E_*` 宏
- **libdwfl 错误**：`DWFL_E_*` 宏

---

## 总结

### API 兼容性

| 方面 | 状态 |
|------|------|
| **公共 API** | ✅ 与上游完全一致 |
| **头文件** | ✅ 原始版本 |
| **行为** | ✅ 无变更 |
| **数据类型** | ✅ 无变更 |
| **错误码** | ✅ 无变更 |

### 实际差异

| 类型 | 描述 |
|------|------|
| **编译产物** | 仅静态库，无共享库和工具 |
| **功能范围** | 精简（不包含 debuginfod 和命令行工具） |
| **配置** | 预生成 config.h，非 configure 生成 |
| **使用** | 仅用于宿主机编译（libabigail） |

### 使用建议

1. **代码移植**：可以直接使用上游示例代码
2. **文档参考**：上游文档适用于 OH 版本
3. **API 学习**：参考上游教程和示例

---

## 相关文档

- [库概览](./01_Overview.md) - 库的功能介绍
- [Patch 分析](./02_Patches.md) - 源代码修改
- [使用情况](./04_Usage_in_OH.md) - 依赖关系
- [完整评估](_work/ASSESSMENT.md) - 技术评估
