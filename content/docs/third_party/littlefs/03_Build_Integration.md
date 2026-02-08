# OH 构建适配

> littlefs 在 OpenHarmony 中采用 `.gni` 文件模式进行构建系统适配，通过源文件列表和配置参数外置的方式实现集成。

---

## 构建系统概述

### 构建方式特点

| 特点 | 说明 |
|------|------|
| **无 BUILD.gn** | 不提供独立的 BUILD.gn target |
| **.gni 文件模式** | 通过 `.gni` 文件暴露源文件列表和包含路径 |
| **外部构建** | 由外部模块（如内核文件系统模块）引用这些变量来构建 |
| **配置外置** | 编译选项由引用方控制 |

### 构建文件结构

```
third_party/littlefs/
├── littlefs.gni              # OH GN 构建系统适配文件（核心）
├── lfs.c                    # 核心实现（与上游一致）
├── lfs.h                    # 主头文件（与上游一致）
├── lfs_util.c               # 工具函数（与上游一致）
├── lfs_util.h               # 工具头文件（与上游一致）
└── bd/
    ├── lfs_rambd.c          # RAM 块设备（测试用）
    └── lfs_rambd.h
```

---

## littlefs.gni 详细说明

### 文件内容

```gni
# Copyright (c) 2022-2022 Huawei Device Co., Ltd. All rights reserved.

LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE = [
  "//third_party/littlefs/lfs.c",
  "//third_party/littlefs/lfs_util.c",
  "//third_party/littlefs/bd/lfs_rambd.c",
]

LITTLEFS_INCLUDE_DIRS = [
  "//third_party/littlefs",
  "//third_party/littlefs/bd",
]
```

### 变量说明

| 变量 | 类型 | 说明 |
|------|------|------|
| `LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE` | list | 内核模块源文件列表 |
| `LITTLEFS_INCLUDE_DIRS` | list | 包含目录列表 |

---

## LiteOS-M 构建配置

### BUILD.gn 文件路径

`/oh/kernel/liteos_m/components/fs/littlefs/BUILD.gn`

### 构建配置

```gn
# Copyright (c) 2013-2019 Huawei Technologies Co., Ltd.
# Copyright (c) 2020-2021 Huawei Device Co., Ltd.

import("//kernel/liteos_m/liteos.gni")
import("$THIRDPARTY_LITTLEFS_DIR/littlefs.gni")

module_switch = defined(LOSCFG_FS_LITTLEFS)
module_name = get_path_info(rebase_path("."), "name")

kernel_module(module_name) {
  configs += [ "$LITEOSTOPDIR:warn_config" ]
  sources = LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE + [ "lfs_adapter.c" ]
}

config("public") {
  include_dirs = LITTLEFS_INCLUDE_DIRS + [ "." ]
}
```

### 配置说明

| 配置项 | 说明 |
|--------|------|
| `module_switch` | 条件编译开关，由 `LOSCFG_FS_LITTLEFS` 控制 |
| `sources` | 源文件列表 = littlefs 核心文件 + OH 适配层 |
| `include_dirs` | 包含目录 = littlefs 头文件 + 适配层目录 |
| `configs` | 添加警告配置 |

---

## UniProton 构建配置

### BUILD.gn 文件路径

`/oh/kernel/uniproton/BUILD.gn`

### 构建配置

```gn
# 引用 littlefs 源文件
if (defined(OS_SUPPORT_FS)) {
  sources += KERNEL_FS_SOURCES + LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE
}
```

### 配置说明

| 配置项 | 说明 |
|--------|------|
| `OS_SUPPORT_FS` | 条件编译开关，控制是否启用文件系统 |
| `KERNEL_FS_SOURCES` | UniProton 内核文件系统源文件列表 |
| `LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE` | littlefs 源文件列表（从 littlefs.gni 导入）|

---

## 关键编译选项

### 条件编译开关

| 宏 | 说明 | 使用场景 |
|---|------|----------|
| `LOSCFG_FS_LITTLEFS` | LiteOS-M 文件系统开关 | LiteOS-M 内核配置 |
| `OS_SUPPORT_FS` | UniProton 文件系统开关 | UniProton 内核配置 |

### 功能裁剪宏（上游提供）

| 宏 | 功能 | 默认值 | 推荐值 |
|---|------|--------|--------|
| `LFS_READONLY` | 只读模式 | 未定义 | 未定义（读写模式）|
| `LFS_NO_MALLOC` | 禁用动态内存分配 | 未定义 | 定义（嵌入式推荐）|
| `LFS_THREADSAFE` | 线程安全支持 | 未定义 | 根据需求定义 |
| `LFS_MULTIVERSION` | 支持多版本磁盘格式 | 未定义 | 未定义 |
| `LFS_MIGRATE` | 支持从旧版本迁移 | 未定义 | 未定义 |
| `LFS_NO_INTRINSICS` | 禁用编译器内置函数 | 未定义 | 未定义 |

### 日志控制宏（上游提供）

| 宏 | 功能 | 默认值 | 推荐值 |
|---|------|--------|--------|
| `LFS_YES_TRACE` | 启用详细跟踪日志 | 未定义 | 未定义（生产环境）|
| `LFS_NO_DEBUG` | 禁用调试日志 | 未定义 | 定义（生产环境）|
| `LFS_NO_WARN` | 禁用警告日志 | 未定义 | 未定义（保留警告）|
| `LFS_NO_ERROR` | 禁用错误日志 | 未定义 | 未定义（保留错误）|
| `LFS_NO_ASSERT` | 禁用断言 | 未定义 | 未定义（保留断言）|

### OH 配置参数（lfs_conf.h）

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `LITTLE_FS_STANDARD_NAME_LENGTH` | 50 | 标准文件名长度 |
| `LITTLE_FS_MAX_NAME_LEN` | 255 | 最大文件名长度 |
| `LITTLEFS_MAX_LFN_LEN` | 255 | 最大长文件名长度 |
| `MAX_DEF_BUF_NUM` | 21 | 默认缓冲区数量 |
| `MAX_WRITE_FILE_LEN` | 500 | 最大写文件长度 |
| `MAX_READ_FILE_LEN` | 500 | 最大读文件长度 |
| `LFS_MAX_OPEN_DIRS` | 10 | 最大打开目录数 |

---

## 编译选项配置示例

### 基础配置

```gn
import("//third_party/littlefs/littlefs.gni")

kernel_module("littlefs") {
  sources = LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE + [
    "lfs_adapter.c",
  ]

  include_dirs = LITTLEFS_INCLUDE_DIRS + [
    ".",
  ]

  configs = [
    ":littlefs_config",
  ]
}

config("littlefs_config") {
  defines = [
    "LFS_NO_MALLOC=1",      # 禁用动态内存分配
    "LFS_NO_DEBUG=1",       # 禁用调试日志
    "LFS_THREADSAFE=0",      # 禁用线程安全（单线程环境）|
  ]

  include_dirs = [
    ".",  # 包含 lfs_conf.h
  ]
}
```

### 线程安全配置

```gn
config("littlefs_config") {
  defines = [
    "LFS_THREADSAFE=1",      # 启用线程安全
    "LFS_NO_MALLOC=1",      # 禁用动态内存分配
    "LFS_NO_DEBUG=1",       # 禁用调试日志
  ]

  # 线程安全需要用户提供 lock/unlock 回调
  include_dirs = [
    ".",
  ]
}
```

### 只读模式配置

```gn
config("littlefs_config") {
  defines = [
    "LFS_READONLY=1",        # 只读模式
    "LFS_NO_MALLOC=1",      # 禁用动态内存分配
  ]

  include_dirs = [
    ".",
  ]
}
```

### 调试模式配置

```gn
config("littlefs_config") {
  defines = [
    "LFS_YES_TRACE=1",      # 启用详细跟踪日志
    "LFS_DEBUG=1",         # 启用调试日志
  ]

  include_dirs = [
    ".",
  ]
}
```

---

## 与上游构建系统的差异

### 上游构建系统（Makefile）

#### Makefile 结构

```makefile
CC ?= gcc
AR ?= ar

CFLAGS += -Wall -Wextra -Werror -pedantic -std=c99
OBJS = lfs.o lfs_util.o

liblfs.a: $(OBJS)
	$(AR) rcs $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

test:
	make -C scripts/test.py

.PHONY: clean test
clean:
	rm -f $(OBJS) liblfs.a
```

#### 构建目标

| 目标 | 命令 | 说明 |
|------|------|------|
| **默认** | `make` | 构建 liblfs.a 静态库 |
| **测试** | `make test` | 运行测试套件 |
| **代码分析** | `make code` | 代码覆盖率分析 |
| **基准测试** | `make bench` | 性能基准测试 |
| **清理** | `make clean` | 清理构建产物 |

#### 编译选项

```makefile
CFLAGS += -Wall -Wextra -Werror -pedantic -std=c99
CFLAGS += -DLFS_NO_MALLOC=1
CFLAGS += -DLFS_NO_DEBUG=1
```

### OH 构建系统（GN）

#### GN 结构

```gn
import("//third_party/littlefs/littlefs.gni")

kernel_module("littlefs") {
  sources = LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE + [
    "lfs_adapter.c",
  ]
  include_dirs = LITTLEFS_INCLUDE_DIRS + [ "." ]
  configs = [ ":littlefs_config" ]
}

config("littlefs_config") {
  defines = [ "LFS_NO_MALLOC=1", "LFS_NO_DEBUG=1" ]
}
```

#### 构建目标

| 目标 | 说明 |
|------|------|
| **kernel_module** | 内核模块（链接到内核镜像）|
| **source_set** | 源文件集合（可复用）|

#### 编译选项

```gn
defines = [
  "LFS_NO_MALLOC=1",
  "LFS_NO_DEBUG=1",
  "LFS_THREADSAFE=0",
]
```

### 差异对比表

| 方面 | 上游（Makefile） | OH 适配（GN） | 说明 |
|---|---|---|---|
| **构建文件** | Makefile | littlefs.gni + 引用方 BUILD.gn | OH 采用 GN 构建系统 |
| **目标类型** | 静态库（liblfs.a）| 内核模块（kernel_module）| OH 直接集成到内核 |
| **编译选项** | 在 Makefile 中定义 | 在引用方 BUILD.gn 中定义 | 编译选项外置 |
| **适配策略** | 独立构建 | 被集成到内核/fs 模块中构建 | OH 采用模块化集成 |
| **测试** | `make test` | 使用 OH 测试框架 | 测试方式不同 |
| **清理** | `make clean` | `gn clean` | 清理命令不同 |

---

## 配置机制说明

### lfs_config 结构体

```c
struct lfs_config {
    // 块设备操作
    int (*read)(const struct lfs_config *c, lfs_block_t block,
               lfs_off_t off, void *buffer, lfs_size_t size);
    int (*prog)(const struct lfs_config *c, lfs_block_t block,
               lfs_off_t off, const void *buffer, lfs_size_t size);
    int (*erase)(const struct lfs_config *c, lfs_block_t block);
    int (*sync)(const struct lfs_config *c);

    // 块设备配置
    lfs_size_t read_size;
    lfs_size_t prog_size;
    lfs_size_t block_size;
    lfs_block_t block_count;
    lfs_size_t cache_size;
    lfs_size_t lookahead_size;
    uint32_t block_cycles;

    // 上下文和缓冲区
    void *context;
    void *read_buffer;
    void *prog_buffer;
    void *lookahead_buffer;

    // 可选操作
    int (*lock)(const struct lfs_config *c);
    int (*unlock)(const struct lfs_config *c);
};
```

### OH 适配层配置

#### LiteOS-M 适配层（lfs_adapter.c）

```c
static struct lfs_config g_lfs_cfg = {
    .context = NULL,  // 分区索引
    .read = littlefs_block_read,
    .prog = littlefs_block_write,
    .erase = littlefs_block_erase,
    .sync = littlefs_block_sync,
    .read_size = LITTLEFS_READ_SIZE,
    .prog_size = LITTLEFS_PROG_SIZE,
    .block_size = LITTLEFS_BLOCK_SIZE,
    .block_count = LITTLEFS_BLOCK_COUNT,
    .cache_size = LITTLEFS_CACHE_SIZE,
    .lookahead_size = LITTLEFS_LOOKAHEAD_SIZE,
    .block_cycles = LITTLEFS_BLOCK_CYCLES,
};
```

#### HAL 层配置（littlefs_hal.c）

```c
const struct lfs_config littlefsConfig = {
    .context = NULL,
    .read = littlefs_hal_read,
    .prog = littlefs_hal_prog,
    .erase = littlefs_hal_erase,
    .sync = littlefs_hal_sync,
    .read_size = 256,
    .prog_size = 256,
    .block_size = 4096,
    .block_count = 512,
    .cache_size = 256,
    .lookahead_size = 256,
    .block_cycles = 500,
};
```

---

## 常见配置场景

### 场景 1: 小容量 Flash 设备

**配置**: 1MB Flash，4KB 块大小

```gn
config("littlefs_config") {
  defines = [
    "LFS_NO_MALLOC=1",      # 禁用动态内存分配
    "LFS_NO_DEBUG=1",       # 禁用调试日志
    "LFS_BLOCK_COUNT=256",   # 256 个 4KB 块 = 1MB
    "LFS_CACHE_SIZE=16",     # 16 字节缓存
    "LFS_LOOKAHEAD_SIZE=16", # 16 字节前瞻
  ]
}
```

### 场景 2: 大容量 Flash 设备

**配置**: 16MB Flash，4KB 块大小

```gn
config("littlefs_config") {
  defines = [
    "LFS_NO_MALLOC=1",      # 禁用动态内存分配
    "LFS_NO_DEBUG=1",       # 禁用调试日志
    "LFS_BLOCK_COUNT=4096",  # 4096 个 4KB 块 = 16MB
    "LFS_CACHE_SIZE=512",   # 512 字节缓存
    "LFS_LOOKAHEAD_SIZE=512", # 512 字节前瞻
    "LFS_BLOCK_CYCLES=1000", # 提高擦除周期阈值
  ]
}
```

### 场景 3: 线程安全环境

**配置**: 多线程环境

```gn
config("littlefs_config") {
  defines = [
    "LFS_NO_MALLOC=1",      # 禁用动态内存分配
    "LFS_NO_DEBUG=1",       # 禁用调试日志
    "LFS_THREADSAFE=1",     # 启用线程安全
  ]

  # 线程安全需要用户提供 lock/unlock 回调
  include_dirs = [
    ".",
  ]
}
```

### 场景 4: 调试模式

**配置**: 开发调试

```gn
config("littlefs_config") {
  defines = [
    "LFS_YES_TRACE=1",      # 启用详细跟踪日志
    "LFS_DEBUG=1",         # 启用调试日志
  ]

  include_dirs = [
    ".",
  ]
}
```

### 场景 5: 只读模式

**配置**: ROM 文件系统

```gn
config("littlefs_config") {
  defines = [
    "LFS_READONLY=1",       # 只读模式
    "LFS_NO_MALLOC=1",     # 禁用动态内存分配
  ]

  include_dirs = [
    ".",
  ]
}
```

---

## 构建系统适配总结

### 适配特点

| 特点 | 说明 |
|------|------|
| **最小侵入** | 不修改上游代码，保持代码纯净性 |
| **配置外置** | 编译选项由引用方控制，灵活性高 |
| **模块化集成** | 通过内核模块（kernel_module）集成到内核 |
| **适配层隔离** | VFS 适配层和 HAL 层与核心库分离 |

### 优势

- ✅ **易于升级**: 无需应用补丁，可直接同步上游更新
- ✅ **灵活性高**: 不同平台可根据需求定制编译选项
- ✅ **可维护性强**: 代码与上游一致，便于追踪问题和提交补丁
- ✅ **职责分离**: 适配层与核心库分离，职责明确

### 注意事项

1. **条件编译开关**: 确保 `LOSCFG_FS_LITTLEFS` 或 `OS_SUPPORT_FS` 正确配置
2. **配置参数**: 根据实际 Flash 容量和性能需求调整 `lfs_config` 参数
3. **线程安全**: 多线程环境需要启用 `LFS_THREADSAFE` 并提供 lock/unlock 回调
4. **内存限制**: 嵌入式环境建议启用 `LFS_NO_MALLOC` 并提供静态缓冲区

---

## 参考资料

### OH 文档

- [GN 构建系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/quick-start/ide-building.md)
- [LiteOS-M 内核文档](https://gitee.com/openharmony/kernel_liteos_m/blob/master/README.md)

### 上游文档

- [littlefs 官方文档](https://github.com/littlefs-project/littlefs/blob/master/README.md)
- [DESIGN.md - 设计原理](https://github.com/littlefs-project/littlefs/blob/master/DESIGN.md)

---

**最后更新时间**: 2026-02-08
**上游版本**: v2.11.2
**OH 版本**: 3.1
