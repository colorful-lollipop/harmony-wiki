# 02 - Patch 详细分析

## 概览

| Patch 文件 | 修改类型 | 修改文件数 | 修改行数 | OH 特有 |
|-----------|----------|-----------|----------|---------|
| `add-linux-makefile-for-Format7zR.patch` | 新增 Makefile | 3 | +556 | 是 |

**Patch 统计**: 共 1 个 Patch，全部为构建系统适配，无功能代码修改。

---

## Patch 清单表

| Patch 文件 | 修改文件 | 修改类型 | 修改目的 | 关联的 OH 需求 |
|-----------|----------|----------|----------|----------------|
| add-linux-makefile-for-Format7zR.patch | CPP/7zip/7zip_gcc_r.mak | 新增 | GCC 编译支持 | 在 OH 构建环境编译 7z 工具 |
| add-linux-makefile-for-Format7zR.patch | CPP/7zip/Bundles/Format7zR/Arc_gcc.mak | 新增 | 对象文件列表 | 支持 Format7zR 模块编译 |
| add-linux-makefile-for-Format7zR.patch | CPP/7zip/Bundles/Format7zR/makefile.gcc | 新增 | 入口 Makefile | 提供编译入口 |

---

## Patch 详细分析

### Patch: `add-linux-makefile-for-Format7zR.patch`

#### 基本信息

| 属性 | 内容 |
|------|------|
| **Patch 文件** | add-linux-makefile-for-Format7zR.patch |
| **作者** | OSOSOS |
| **日期** | 2025-09-25 |
| **提交信息** | add linux makefile for Format7zR |
| **影响范围** | CPP/7zip/ 目录 |
| **风险等级** | 低 |

#### 修改文件详解

##### 1. CPP/7zip/7zip_gcc_r.mak (新增)

**文件作用**: 核心 GCC Makefile 配置文件

**文件大小**: 477 行

**关键配置**:

```makefile
# 编译器选项
FLAGS_BASE = -mbranch-protection=standard  -march=armv8.5-a
FLAGS_BASE = -mbranch-protection=standard
FLAGS_BASE =

CFLAGS_BASE = -O2 $(CFLAGS_BASE_LIST) $(CFLAGS_WARN_WALL) $(CFLAGS_WARN) \
 $(CFLAGS_DEBUG) -D_REENTRANT -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE \
 -fPIC

FLAGS_FLTO = -ffunction-sections
FLAGS_FLTO = -flto
FLAGS_FLTO = $(FLAGS_BASE)
```

**分析**:
- 使用 `-O2` 优化级别
- 支持大文件 (`_FILE_OFFSET_BITS=64`)
- 位置无关代码 (`-fPIC`)
- 包含分支保护选项的配置（注释状态）

##### 2. CPP/7zip/Bundles/Format7zR/Arc_gcc.mak (新增)

**文件作用**: Format7zR 模块的对象文件列表

**文件大小**: 58 行

**关键内容**:

```makefile
include ../../LzmaDec_gcc.mak

# 单线程模式支持
ifdef ST_MODE
LOCAL_FLAGS_ST = -DZ7_ST
endif

# C 语言源文件对象列表 (共 31 个对象文件)
COMMON_C_OBJS = \
  $O/7zAlloc.o \
  $O/7zArcIn.o \
  $O/7zBuf2.o \
  ...
  $O/LzFindMt.o \
  $O/MtDec.o \
  $O/Threads.o \
```

**包含的源文件**:
- 7z 格式处理: `7zAlloc`, `7zArcIn`, `7zBuf`, `7zCrc`, `7zDec`, `7zFile`
- LZMA 核心: `LzmaDec`, `LzmaEnc`, `LzmaLib`
- LZMA2 支持: `Lzma2Dec`, `Lzma2Enc`
- XZ 支持: `Xz`, `XzCrc64`, `XzDec`, `XzEnc`, `XzIn`
- 多线程: `LzFindMt`, `MtCoder`, `MtDec`, `Threads`
- 其他: `Bcj2`, `Bra`, `Delta`, `Ppmd7`, `Sha256`, `Sort`

##### 3. CPP/7zip/Bundles/Format7zR/makefile.gcc (新增)

**文件作用**: Format7zR 的入口 Makefile

**文件大小**: 21 行

**关键内容**:

```makefile
PROG = 7z
DEF_FILE = ../../Archive/Archive2.def

# IS_X64 = 1
# USE_ASM = 1
# ST_MODE = 1

include Arc_gcc.mak

LOCAL_FLAGS_SYS =

LOCAL_FLAGS = \
  -DZ7_EXTERNAL_CODECS \
  $(LOCAL_FLAGS_SYS) \
  $(LOCAL_FLAGS_ST) \

OBJS = \
  $(ARC_OBJS) \

include ../../7zip_gcc_r.mak
```

**分析**:
- 目标程序名: `7z`
- 支持外部编解码器 (`Z7_EXTERNAL_CODECS`)
- 可配置单线程模式 (`ST_MODE`)
- 可启用汇编优化 (`USE_ASM`)

#### 原始问题

**上游限制**:
- LZMA SDK 上游主要提供 Windows MSVC 的 makefile (`makefile`)
- Linux GCC 支持不完整，只有基础模块的 makefile
- Format7zR 模块缺少 Linux GCC 编译支持

**OH 需求**:
- 需要在 Linux 环境（包括 OH 构建系统）编译 7z 工具
- 需要完整的 GCC 编译配置
- 需要支持 ARM64 等架构的交叉编译

#### 修改内容摘要

| 项目 | 原始状态 | Patch 后状态 |
|------|----------|--------------|
| Format7zR GCC 支持 | ❌ 不存在 | ✅ 完整支持 |
| 编译配置 | Windows MSVC | GCC/Linux |
| 架构支持 | x86/x64 | 多架构 (ARM/ARM64/RISC-V) |
| 优化选项 | 基础 | -O2, -fPIC, LTO 可选 |

#### OH 价值

1. **构建系统兼容**: 使 7z 工具能在 OH GN 构建系统中编译
2. **跨平台支持**: 支持 ARM64、RISC-V 等 OH 目标架构
3. **开发工具链**: 为主机端开发提供 7z 压缩/解压能力

#### 与上游的差异

**上游**:
- `CPP/7zip/Bundles/Format7zR/makefile` - Windows MSVC 版本
- 无 Linux GCC 支持

**OH**:
- `CPP/7zip/Bundles/Format7zR/makefile.gcc` - Linux GCC 版本 (Patch 新增)
- `CPP/7zip/Bundles/Format7zR/Arc_gcc.mak` - 对象文件列表 (Patch 新增)
- `CPP/7zip/7zip_gcc_r.mak` - 核心配置 (Patch 新增)

#### 回归风险

| 风险项 | 等级 | 说明 |
|--------|------|------|
| 上游更新 | 低 | 上游很少修改 Makefile |
| 功能影响 | 无 | 仅新增文件，不修改现有代码 |
| 编译失败 | 低 | 独立文件，不影响其他模块 |
| 维护成本 | 低 | Makefile 改动频率低 |

**升级建议**:
- 此 Patch 为 **OH 特有**，升级上游版本时需要保留
- 建议将此 Patch 贡献给上游，增强上游对 GCC/Linux 的支持
- 升级时检查上游是否有新增源文件需要添加到 `Arc_gcc.mak`

---

## Patch 分类总结

### 按修改类型分类

| 类型 | 数量 | 说明 |
|------|------|------|
| 构建适配 | 1 | Makefile 添加 |
| Bugfix | 0 | - |
| 功能增强 | 0 | - |
| 安全修复 | 0 | - |
| OH 特有适配 | 1 | GCC/Linux 支持 |

### 按文件类型分类

| 类型 | 数量 | 文件 |
|------|------|------|
| Makefile | 3 | `7zip_gcc_r.mak`, `Arc_gcc.mak`, `makefile.gcc` |
| C/C++ 代码 | 0 | - |
| 头文件 | 0 | - |
| 配置文件 | 0 | - |

---

## 升级建议

### 短期 (当前版本维护)

1. **保持现状**: Patch 工作正常，无需修改
2. **文档化**: 已在本文档中详细记录 Patch 用途

### 中期 (上游版本升级)

1. **检查上游变更**:
   - 查看新版本是否有新增源文件
   - 检查是否需要更新 `Arc_gcc.mak` 中的对象文件列表
   
2. **测试验证**:
   - 验证 Format7zR 编译是否正常
   - 验证生成的 7z 工具功能正常

### 长期 (社区贡献)

1. **推向上游**:
   - 整理 Patch 提交给 7-Zip 项目
   - 说明 Linux GCC 支持的价值
   
2. **协作维护**:
   - 与上游协作维护 GCC Makefile
   - 减少 OH 特有 Patch 数量

---

## 附录：Patch 文件全文

Patch 文件位于: `//third_party/lzma/add-linux-makefile-for-Format7zR.patch`

完整内容详见原文件。

---

*最后更新: 2025-02-08*
