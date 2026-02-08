# API/接口差异

> OpenHarmony 新增的 API 和接口差异

---

## 概述

OpenHarmony 没有修改 FatFs 的标准 API，而是通过以下方式扩展：

| 扩展方式 | 说明 |
|----------|------|
| **新增磁盘 I/O 接口** | 在 `diskio.h` 中新增优化接口 |
| **虚拟分区 API** | 通过 `virpart/` 模块提供虚拟分区接口 |
| **VFS 适配层** | 通过 `fatfs.c` 提供 VFS 接口（非 FatFs 原生 API）|

---

## 1. 新增磁盘 I/O 接口

### 1.1 disk_read_readdir()

**原型**：
```c
// source/diskio.h:43
DRESULT disk_read_readdir(BYTE pdrv, BYTE* buff, LBA_t sector, UINT count);
```

**参数**：
- `pdrv` - 物理驱动号（0: mmcblk0, 1: mmcblk1, etc.）
- `buff` - 数据缓冲区
- `sector` - 起始扇区号
- `count` - 要读取的扇区数

**返回值**：
- `RES_OK` (0) - 成功
- `RES_ERROR` (1) - 读写错误
- `RES_NOTRDY` (3) - 未就绪
- `RES_PARERR` (4) - 参数错误

**用途**：目录读取优化，减少重复的磁盘 I/O 操作。

**可用性**：仅标准系统可用（`#ifndef __LITEOS_M__`）。

### 1.2 disk_raw_read()

**原型**：
```c
// source/diskio.h:44
DRESULT disk_raw_read(int id, void* buff, LBA_t sector, UINT32 count);
```

**参数**：
- `id` - 设备 ID
- `buff` - 数据缓冲区
- `sector` - 起始扇区号
- `count` - 要读取的扇区数

**返回值**：同 `disk_read_readdir()`

**用途**：原始扇区读取，绕过 FatFs 缓存直接读取原始数据。

**可用性**：仅标准系统可用（`#ifndef __LITEOS_M__`）。

### 1.3 disk_raw_write()

**原型**：
```c
// source/diskio.h:45
DRESULT disk_raw_write(int id, void* buff, LBA_t sector, UINT32 count);
```

**参数**：
- `id` - 设备 ID
- `buff` - 数据缓冲区
- `sector` - 起始扇区号
- `count` - 要写入的扇区数

**返回值**：同 `disk_read_readdir()`

**用途**：原始扇区写入，绕过 FatFs 缓存直接写入原始数据。

**可用性**：仅标准系统可用（`#ifndef __LITEOS_M__`）。

---

## 2. 虚拟分区 API

### 2.1 虚拟分区错误码

**文件**：`source/errcode_fat.h`

**错误码范围**：`0x10000000` - `0x1000000D`

| 错误码 | 值（十六进制） | 说明 |
|--------|----------------|------|
| `VIRERR_OK` | 0x00000000 | 操作成功 |
| `VIRERR_BASE` | 0x10000000 | 虚拟分区错误码基地址 |
| `VIRERR_MODIFIED` | 0x10000001 | 分区已被修改 |
| `VIRERR_CHAIN_ERR` | 0x10000002 | 链表错误 |
| `VIRERR_OCCUPIED` | 0x10000003 | 分区已被占用 |
| `VIRERR_NOTCLEAR` | 0x10000004 | 分区未清理 |
| `VIRERR_NOTFIT` | 0x10000005 | 空间不足 |
| `VIRERR_NOTMOUNT` | 0x10000006 | 分区未挂载 |
| `VIRERR_INTER_ERR` | 0x10000007 | 内部错误 |
| `VIRERR_NOPARAM` | 0x10000008 | 参数错误 |
| `VIRERR_PARMLOCKED` | 0x10000009 | 参数已锁定 |
| `VIRERR_PARMNUMERR` | 0x1000000A | 参数数量错误 |
| `VIRERR_PARMPERCENTERR` | 0x1000000B | 百分比参数错误 |
| `VIRERR_PARMNAMEERR` | 0x1000000C | 参数名称错误 |
| `VIRERR_PARMDEVERR` | 0x1000000D | 设备参数错误 |

### 2.2 虚拟分区接口

**文件**：`kernel/liteos_a/fs/fat/virpart/include/virpartff.h`

**TODO(需确认)**：详细的虚拟分区 API 文档需要补充。

---

## 3. VFS 适配层 API

### 3.1 文件操作

VFS 适配层提供了标准 POSIX 文件接口：

```c
// kernel/liteos_a/fs/fat/os_adapt/fatfs.c

int fatfs_open(struct Vnode *vp, const char *path, int oflag, mode_t mode);
ssize_t fatfs_read(struct file *filep, char *buf, size_t len);
ssize_t fatfs_write(struct file *filep, const char *buf, size_t len);
int fatfs_close(struct file *filep);
off_t fatfs_lseek(struct file *filep, off_t offset, int whence);
int fatfs_sync(struct file *filep);
```

### 3.2 目录操作

```c
// kernel/liteos_a/fs/fat/os_adapt/fatfs.c

int fatfs_opendir(struct Vnode *vp, const char *path, struct dir **dir);
int fatfs_readdir(struct dir *dir, struct dirent *de);
int fatfs_closedir(struct dir *dir);
int fatfs_mkdir(struct Vnode *parent, const char *name, mode_t mode);
int fatfs_rmdir(struct Vnode *vp, const char *name);
```

### 3.3 文件系统操作

```c
// kernel/liteos_a/fs/fat/os_adapt/fatfs.c

int fatfs_mount(const char *source, const char *target, const char *filesystemtype,
              unsigned long mountflags, const void *data);
int fatfs_unmount(const char *target, unsigned long flags);
int fatfs_statfs(const char *path, struct statfs *buf);
```

---

## 4. API 差异总结

| 类别 | 标准 FatFs | OH 适配 | OH 特有 |
|------|-----------|---------|---------|
| **FatFs API** | f_open, f_read, f_write, etc. | 未修改 | - |
| **磁盘 I/O** | disk_read, disk_write, disk_ioctl | 新增 3 个接口 | `disk_read_readdir()`, `disk_raw_read()`, `disk_raw_write()` |
| **虚拟分区** | 不支持 | - | 完整实现（virpart/） |
| **VFS 接口** | 不提供 | 提供完整的 VFS 适配 | - |
| **错误码** | FR_OK, FR_DISK_ERR, etc. | - | 虚拟分区错误码（`VIRERR_*`） |

---

## 5. 使用示例

### 5.1 使用 FatFs 原生 API（不推荐）

```c
#include "ff.h"

// 挂载文件系统
FATFS fs;
FRESULT res = f_mount(&fs, "0:", 1);
if (res != FR_OK) {
    // 处理错误
}

// 打开文件
FIL file;
res = f_open(&file, "test.txt", FA_READ | FA_WRITE);
if (res != FR_OK) {
    // 处理错误
}

// 写入数据
UINT bw;
res = f_write(&file, "Hello", 5, &bw);

// 关闭文件
f_close(&file);

// 卸载文件系统
f_mount(NULL, "0:", 0);
```

**注意**：在 OH 中，推荐通过 VFS 接口访问文件，不直接使用 FatFs API。

### 5.2 使用 VFS 接口（推荐）

```c
#include <fcntl.h>
#include <unistd.h>

// 打开文件
int fd = open("/mnt/sdcard/test.txt", O_RDWR | O_CREAT);
if (fd < 0) {
    perror("open failed");
    return -1;
}

// 写入数据
write(fd, "Hello", 5);

// 关闭文件
close(fd);
```

**流程**：VFS → FatFs 适配层 → FatFs API

### 5.3 使用虚拟分区（TODO）

```c
// TODO: 需要补充虚拟分区使用示例
```

---

## 6. API 兼容性

### 6.1 标准应用兼容性

FatFs 的标准 API 未被修改，因此：

- ✅ 现有的 FatFs 应用代码可以在 OH 中编译和运行
- ✅ FatFs 官方文档的示例代码可以在 OH 中使用
- ⚠️ 但推荐通过 VFS 接口访问文件

### 6.2 LiteOS-M vs 标准系统

| API | LiteOS-M | 标准系统 |
|-----|----------|----------|
| `disk_read_readdir()` | ❌ 不可用 | ✅ 可用 |
| `disk_raw_read()` | ❌ 不可用 | ✅ 可用 |
| `disk_raw_write()` | ❌ 不可用 | ✅ 可用 |
| 虚拟分区 API | ❌ 不可用 | ✅ 可用（配置） |
| VFS 接口 | ✅ 可用 | ✅ 可用 |

---

## 7. 升级注意事项

### 7.1 磁盘 I/O 接口兼容性

升级 FatFs 上游版本时，需要确保：

- 新增的磁盘 I/O 接口仍然可用
- 如果新版本不支持，需要在 `diskio.c` 中重新实现

### 7.2 虚拟分区兼容性

虚拟分区功能依赖特定版本的 FatFs，升级时：

- 测试虚拟分区功能是否正常
- 如有必要，更新 `virpartff.c` 以适配新版本
- 考虑将虚拟分区功能独立于 FatFs 版本

### 7.3 VFS 适配层兼容性

VFS 适配层（`fatfs.c`）可能需要更新：

- FatFs API 变更需要更新适配层
- 新增的 FatFs 功能需要在适配层中暴露
- 测试所有 VFS 接口是否正常

---

## 8. 总结

### OH 新增 API 清单

| 类别 | 接口/功能 | 说明 |
|------|------------|------|
| **磁盘 I/O** | `disk_read_readdir()` | 目录读取优化 |
| **磁盘 I/O** | `disk_raw_read()` | 原始扇区读取 |
| **磁盘 I/O** | `disk_raw_write()` | 原始扇区写入 |
| **虚拟分区** | 虚拟分区 API（TODO） | 在单一 FAT 卷上创建多个逻辑分区 |
| **错误码** | `VIRERR_*` (13 个） | 虚拟分区错误码 |

### API 差异特点

1. **不修改 FatFs 标准 API**：标准应用兼容
2. **通过扩展接口增强**：不破坏原有功能
3. **独立适配层**：VFS 接口与 FatFs 解耦
4. **条件编译**：LiteOS-M 和标准系统不同

---

## 参考资料

### 相关文档

- **[ASSESSMENT.md](wiki/_work/ASSESSMENT.md)** - 项目评估结果
- **[02_Patches.md](02_Patches.md)** - Patch 详细分析
- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 依赖关系与使用

### 外部资源

- **FatFs API 文档**：http://elm-chan.org/fsw/ff/00index_e.html
- **OpenHarmony VFS 文档**：https://gitee.com/openharmony/docs

---

**文档版本**：1.0
**最后更新**：2026-02-08
