# littlefs 原始库简介

> littlefs 是一个专为微控制器设计的小型故障安全文件系统，提供断电恢复、动态磨损均衡和内存边界限制等核心特性。

---

## 基本信息

| 项目 | 值 |
|------|-----|
| **库名称** | littlefs |
| **上游版本** | v2.11.2 (LFS_VERSION: 0x0002000b) |
| **磁盘版本** | lfs2.1 (LFS_DISK_VERSION: 0x00020001) |
| **许可证** | BSD 3-Clause License |
| **开发语言** | C99 |
| **上游地址** | https://github.com/littlefs-project/littlefs |

---

## 原始功能描述

littlefs 是一个专为嵌入式微控制器设计的小型故障安全文件系统，具有以下核心特性：

1. **断电恢复（Power-loss resilience）**
   - 所有文件操作都有强 copy-on-write 保证
   - 断电后会回退到最后已知良好状态
   - 所有 POSIX 操作（remove、rename等）都是原子性的

2. **动态磨损均衡（Dynamic wear leveling）**
   - 专为 Flash 存储设计
   - 动态块磨损均衡，延长 Flash 寿命
   - 可检测坏块并绕过

3. **内存边界限制（Bounded RAM/ROM）**
   - RAM 使用量严格受限，不随文件系统增长而变化
   - 无无界递归，动态内存限制在可配置的静态缓冲区

一句话描述：littlefs 是一个专为嵌入式设备设计的、具有断电恢复能力和磨损均衡功能的轻量级文件系统。

---

## 上游资源

### 官方仓库

- **GitHub**: https://github.com/littlefs-project/littlefs
- **最新版本**: v2.11.2 (commit: adad0fb)
- **最新发布**: 2024年

### 官方文档

| 文档 | 说明 | 链接 |
|------|------|------|
| **README.md** | 快速开始和示例 | [链接](https://github.com/littlefs-project/littlefs/blob/master/README.md) |
| **DESIGN.md** | 设计原理详细说明 | [链接](https://github.com/littlefs-project/littlefs/blob/master/DESIGN.md) |
| **SPEC.md** | 磁盘格式规范 | [链接](https://github.com/littlefs-project/littlefs/blob/master/SPEC.md) |
| **LICENSE.md** | BSD-3-Clause 许可证 | [链接](https://github.com/littlefs-project/littlefs/blob/master/LICENSE.md) |

### 生态系统

| 项目 | 描述 | 链接 |
|------|------|------|
| **littlefs-fuse** | Linux FUSE 封装 | [geky/littlefs-fuse](https://github.com/geky/littlefs-fuse) |
| **littlefs-python** | Python 封装 | [PyPI](https://pypi.org/project/littlefs-python/) |
| **littlefs2-rust** | Rust 封装 | [crates.io](https://crates.io/crates/littlefs2) |
| **mklittlefs** | 镜像创建工具（ESP8266/RP2040） | [earlephilhower/mklittlefs](https://github.com/earlephilhower/mklittlefs) |
| **Mbed OS** | 官方集成 | [ARM mbed](https://os.mbed.com/docs/mbed-os/latest/apis/littlefilesystem.html) |

---

## 核心技术特点

### 混合架构设计

littlefs 采用独特的"双层蛋糕"设计：

```
                    root
                   .--------.--------.
                   | A'| B'|         |
                   '--------'--------'
                .----'   '--------------.
        A       v                 B       v
       .--------.--------.       .--------.--------.
       | C'| D'|         |       | E'|new|         |
       '--------'--------'       '--------'--------'
```

- **元数据层（Metadata pairs）**：小型两区块日志，提供分布式原子更新
- **数据层（COW）**：Copy-on-Write 结构，紧凑存储文件数据且无磨损放大成本

### 资源占用

| 资源类型 | 占用量 | 说明 |
|---------|--------|------|
| **ROM** | ~56KB | 代码空间占用（OH 优化后）|
| **RAM** | ~112KB | 运行时内存占用（OH 优化后）|
| **最小 RAM** | ~1KB | 基本运行（不含缓冲区）|

### 可配置参数

通过 `struct lfs_config` 配置：

| 参数 | 说明 | 典型值 |
|------|------|--------|
| `read_size` | 最小读取大小 | 16-512 bytes |
| `prog_size` | 最小编程大小 | 16-512 bytes |
| `block_size` | 擦除块大小 | 4096-65536 bytes |
| `block_count` | 块数量 | 根据存储容量计算 |
| `block_cycles` | 擦除周期阈值 | 100-1000 |
| `cache_size` | 缓存大小 | 与 read_size 相同 |
| `lookahead_size` | 前瞻缓冲区大小 | 16-512 bytes |

### 编译时宏定义

| 宏 | 功能 | 默认值 |
|---|------|--------|
| `LFS_NAME_MAX` | 最大文件名长度 | 255 |
| `LFS_FILE_MAX` | 最大文件大小 | 2GB |
| `LFS_ATTR_MAX` | 最大属性大小 | 1022 |
| `LFS_READONLY` | 只读模式 | 未定义 |
| `LFS_THREADSAFE` | 线程安全支持 | 未定义 |
| `LFS_NO_MALLOC` | 禁用动态内存分配 | 未定义 |
| `LFS_MIGRATE` | 支持版本迁移 | 未定义 |

---

## 核心 API 概述

### 文件系统操作

```c
int lfs_format(lfs_t *lfs, const struct lfs_config *config);
int lfs_mount(lfs_t *lfs, const struct lfs_config *config);
int lfs_unmount(lfs_t *lfs);
int lfs_remove(lfs_t *lfs, const char *path);
int lfs_rename(lfs_t *lfs, const char *oldpath, const char *newpath);
int lfs_stat(lfs_t *lfs, const char *path, struct lfs_info *info);
```

### 文件操作

```c
int lfs_file_open(lfs_t *lfs, lfs_file_t *file, const char *path, int flags);
int lfs_file_close(lfs_t *lfs, lfs_file_t *file);
lfs_ssize_t lfs_file_read(lfs_t *lfs, lfs_file_t *file, void *buffer, lfs_size_t size);
lfs_ssize_t lfs_file_write(lfs_t *lfs, lfs_file_t *file, const void *buffer, lfs_size_t size);
lfs_soff_t lfs_file_seek(lfs_t *lfs, lfs_file_t *file, lfs_soff_t off, int whence);
int lfs_file_truncate(lfs_t *lfs, lfs_file_t *file, lfs_off_t size);
```

### 目录操作

```c
int lfs_mkdir(lfs_t *lfs, const char *path);
int lfs_dir_open(lfs_t *lfs, lfs_dir_t *dir, const char *path);
int lfs_dir_close(lfs_t *lfs, lfs_dir_t *dir);
int lfs_dir_read(lfs_t *lfs, lfs_dir_t *dir, struct lfs_info *info);
```

### 扩展属性

```c
lfs_ssize_t lfs_getattr(lfs_t *lfs, const char *path, uint8_t type, void *buffer, lfs_size_t size);
int lfs_setattr(lfs_t *lfs, const char *path, uint8_t type, const void *buffer, lfs_size_t size);
int lfs_removeattr(lfs_t *lfs, const char *path, uint8_t type);
```

---

## 上游构建系统

### Makefile 构建

| 目标 | 命令 | 说明 |
|------|------|------|
| 构建库 | `make` 或 `make liblfs.a` | 生成静态库 |
| 运行测试 | `make test` | 执行测试套件 |
| 代码分析 | `make code` | 代码覆盖率分析 |
| 数据分析 | `make data` | 数据结构分析 |
| 堆栈分析 | `make stack` | 堆栈使用分析 |
| 基准测试 | `make bench` | 性能基准测试 |
| 覆盖率 | `make cov` | 测试覆盖率 |

### 关键源文件

| 文件 | 行数 | 说明 |
|------|------|------|
| **lfs.c** | 6546 | 核心实现 |
| **lfs.h** | 801 | 主头文件 |
| **lfs_util.c** | 37 | 工具函数 |
| **lfs_util.h** | 273 | 工具头文件 |
| **总计** | 7657 | - |

### 块设备示例

| 文件 | 说明 |
|------|------|
| **bd/lfs_rambd.c** | RAM 块设备 |
| **bd/lfs_filebd.c** | 文件块设备 |
| **bd/lfs_emubd.c** | 模拟块设备 |
| **bd/lfs_testbd.c** | 测试块设备 |

---

## 与其他文件系统对比

| 特性 | littlefs | FATfs | SPIFFS |
|------|----------|-------|--------|
| **断电恢复** | ✅ | ❌ | ✅ |
| **磨损均衡** | ✅ 动态 | ❌ | ✅ 静态 |
| **RAM 占用** | 严格受限 | 较高 | 严格受限 |
| **PC 互操作** | ❌ | ✅ | ❌ |
| **文件大小** | 2GB | 4GB | 2GB |
| **目录支持** | ✅ | ✅ | ❌ |
| **Flash 优化** | ✅ | ❌ | ✅ |
| **适用场景** | 嵌入式存储 | SD 卡/PC 互操作 | 小容量 Flash |

---

## 上游测试套件

littlefs 包含完整的测试套件（`tests/` 目录）：

| 测试套件 | 说明 | 测试数量 |
|---------|------|----------|
| **test_paths.toml** | 路径测试（最大）| 327195 行 |
| **test_move.toml** | 文件移动测试 | 71009 行 |
| **test_compat.toml** | 兼容性测试 | 42929 行 |
| **test_dirs.toml** | 目录测试 | 36281 行 |
| **test_alloc.toml** | 分配测试 | 24138 行 |
| **test_seek.toml** | 定位测试 | 22299 行 |
| **test_truncate.toml** | 截断测试 | 17553 行 |
| **test_files.toml** | 文件测试 | 17620 行 |
| **test_entries.toml** | 文件项测试 | 22546 行 |
| **test_superblocks.toml** | 超级块测试 | 20012 行 |
| **test_relocations.toml** | 重定位测试 | 18751 行 |
| **test_attrs.toml** | 属性测试 | 12219 行 |
| **test_exhaustion.toml** | 耗尽测试 | 16965 行 |
| **test_evil.toml** | 边界测试 | 10596 行 |
| **test_interspersed.toml** | 混合测试 | 8442 行 |
| **test_bd.toml** | 块设备测试 | 6811 行 |
| **test_badblocks.toml** | 坏块测试 | 7983 行 |
| **test_orphans.toml** | 孤立测试 | 11054 行 |
| **test_powerloss.toml** | 断电测试 | 5842 行 |
| **test_shrink.toml** | 缩减测试 | 3730 行 |

---

## 该库在 OpenHarmony 中的作用和定位

### 核心定位

littlefs 在 OpenHarmony 中作为**嵌入式 MCU 文件系统**提供关键支持：

1. **LiteOS-M 内核标准文件系统**
   - 为轻量级内核提供可靠的文件系统支持
   - 通过 VFS 层适配，与 POSIX 兼容
   - 适用于需要持久化存储的 IoT 设备

2. **UniProton RTOS 文件系统**
   - 为实时操作系统提供文件系统支持
   - 条件编译控制，可按需启用

3. **芯片厂商深度集成**
   - 海思 WS63V100 星闪芯片 SDK 内置 littlefs
   - Rockchip RK2206 芯片 HAL 层适配
   - QEMU 模拟器全覆盖支持

### 适用场景

| 场景 | 描述 |
|------|------|
| **IoT 设备数据持久化** | 配置文件、日志、用户数据存储 |
| **固件更新** | 版本信息、更新状态管理 |
| **数据采集** | 传感器数据缓存和上传 |
| **开发调试** | QEMU 模拟器环境测试 |

### OH 适配特点

| 特点 | 说明 |
|------|------|
| **无代码 Patch** | littlefs 核心代码与上游完全一致 |
| **适配层隔离** | 通过 VFS 适配层（lfs_adapter.c）实现集成 |
| **配置外置** | 编译选项由引用方控制（littlefs.gni + lfs_conf.h）|
| **版本同步** | OH 版本与上游最新版本保持一致（v2.11.2）|

---

## 版本历史

| 版本 | 发布时间 | 主要特性 |
|------|----------|----------|
| **v2.11.2** | 2024 | 当前稳定版本（OH 集成版本）|
| **v2.10** | 2023 | 性能优化，修复多个 bug |
| **v2.9** | 2023 | 改进磨损均衡算法 |
| **v2.8** | 2022 | 新增扩展属性支持 |
| **v2.7** | 2022 | 改进断电恢复机制 |
| **v2.6** | 2021 | 新增文件 truncate 支持 |
| **v2.5** | 2021 | 改进目录操作性能 |
| **v2.4** | 2021 | 新增文件移动支持 |
| **v2.3** | 2021 | 改进错误处理 |
| **v2.2** | 2020 | 新增文件属性支持 |
| **v2.1** | 2020 | 重构元数据存储，提升性能 |
| **v2.0** | 2019 | 重构架构，引入 copy-on-write |

---

## 许可证

littlefs 采用 **BSD 3-Clause License**：

```
Copyright (c) 2022, The littlefs authors.
Copyright (c) 2017, Arm Limited.

Redistribution and use in source and binary forms, with or without modification,
are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors
   may be used to endorse or promote products derived from this software without
   specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
```

**SPDX 标识**: `BSD-3-Clause`

---

**最后更新时间**: 2026-02-08
**上游版本**: v2.11.2
**OH 版本**: 3.1
