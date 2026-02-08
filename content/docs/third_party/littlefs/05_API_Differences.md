# API/接口差异

> **结论**: littlefs 在 OpenHarmony 中**没有 API 差异**，核心 API 与上游完全一致，通过适配层实现 POSIX 兼容。

---

## OH 新增 API

**无新增 API**

littlefs 核心库在 OpenHarmony 中没有新增任何 API。所有核心 API 与上游 v2.11.2 版本完全一致。

---

## 行为变更 API

**无行为变更**

littlefs 在 OpenHarmony 中的行为与上游完全一致，没有针对 OH 的特殊行为修改。

---

## 废弃或禁用的功能

**无废弃或禁用功能**

littlefs 的所有功能在 OpenHarmony 中均可用，没有废弃或禁用任何功能。

---

## VFS 适配层 API

虽然 littlefs 核心 API 没有差异，但 OH 通过 VFS 适配层（lfs_adapter.c）提供了 POSIX 兼容接口。

### POSIX API 映射

| POSIX API | LittleFS API | 实现 | 说明 |
|-----------|--------------|------|------|
| `mount()` | `lfs_mount()` | LfsMount() | 挂载文件系统 |
| `umount()` | `lfs_unmount()` | LfsUmount() | 卸载文件系统 |
| `statfs()` | - | LfsStatfs() | 获取文件系统状态 |
| `open()` | `lfs_file_open()` | LfsOpen() | 打开文件 |
| `close()` | `lfs_file_close()` | LfsClose() | 关闭文件 |
| `read()` | `lfs_file_read()` | LfsRead() | 读取文件 |
| `write()` | `lfs_file_write()` | LfsWrite() | 写入文件 |
| `lseek()` | `lfs_file_seek()` | LfsSeek() | 定位文件 |
| `stat()` | `lfs_stat()` | LfsStat() | 获取文件状态 |
| `unlink()` | `lfs_remove()` | LfsUnlink() | 删除文件 |
| `rename()` | `lfs_rename()` | LfsRename() | 重命名文件 |
| `fsync()` | `lfs_file_sync()` | LfsFsync() | 同步文件 |
| `mkdir()` | `lfs_mkdir()` | LfsMkdir() | 创建目录 |
| `rmdir()` | `lfs_remove()` | LfsRmdir() | 删除目录 |
| `opendir()` | `lfs_dir_open()` | LfsOpendir() | 打开目录 |
| `closedir()` | `lfs_dir_close()` | LfsClosedir() | 关闭目录 |
| `readdir()` | `lfs_dir_read()` | LfsReaddir() | 读取目录 |
| `rewinddir()` | - | LfsRewinddir() | 重置目录位置 |

### VFS 结构体

#### MountOps

```c
static const struct MountOps g_lfsMnt = {
    .mount = LfsMount,
    .umount = LfsUmount,
    .statfs = LfsStatfs,
};
```

#### FileOps

```c
static const struct FileOps g_lfsFops = {
    .open = LfsOpen,
    .close = LfsClose,
    .read = LfsRead,
    .write = LfsWrite,
    .lseek = LfsSeek,
    .stat = LfsStat,
    .unlink = LfsUnlink,
    .rename = LfsRename,
    .fsync = LfsFsync,
};
```

#### FsManagementOps

```c
static const struct FsManagementOps g_lfsMgt = {
    .mkdir = LfsMkdir,
    .rmdir = LfsRmdir,
    .opendir = LfsOpendir,
    .closedir = LfsClosedir,
    .readdir = LfsReaddir,
    .rewinddir = LfsRewinddir,
};
```

---

## HAL 层 API

OH 的 HAL 层（littlefs_hal.c）提供了块设备操作接口。

### 块设备操作 API

| API | 说明 |
|------|------|
| `littlefs_hal_read()` | 读取块设备 |
| `littlefs_hal_prog()` | 编程块设备 |
| `littlefs_hal_erase()` | 擦除块设备 |
| `littlefs_hal_sync()` | 同步块设备 |

### 块设备配置

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

---

## 配置 API

### lfs_conf.h 配置参数

| 配置项 | 值 | 说明 |
|-------|-----|------|
| `LITTLE_FS_STANDARD_NAME_LENGTH` | 50 | 标准文件名长度 |
| `LITTLE_FS_MAX_NAME_LEN` | 255 | 最大文件名长度 |
| `LITTLEFS_MAX_LFN_LEN` | 255 | 最大长文件名长度 |
| `MAX_DEF_BUF_NUM` | 21 | 默认缓冲区数量 |
| `MAX_WRITE_FILE_LEN` | 500 | 最大写文件长度 |
| `MAX_READ_FILE_LEN` | 500 | 最大读文件长度 |
| `LFS_MAX_OPEN_DIRS` | 10 | 最大打开目录数 |

### 使用方式

```c
#include "lfs_conf.h"

char name[LITTLE_FS_MAX_NAME_LEN];
```

---

## 线程安全 API

### 线程安全回调

当启用 `LFS_THREADSAFE=1` 时，需要在 `lfs_config` 中提供 lock/unlock 回调：

```c
struct lfs_config cfg = {
    // ... 其他配置 ...
    .lock = lfs_lock,
    .unlock = lfs_unlock,
};

void lfs_lock(const struct lfs_config *c) {
    // 加锁操作
    pthread_mutex_lock(&lock);
}

void lfs_unlock(const struct lfs_config *c) {
    // 解锁操作
    pthread_mutex_unlock(&lock);
}
```

---

## 错误码

### LittleFS 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `LFS_ERR_OK` | 0 | 成功 |
| `LFS_ERR_IO` | -5 | I/O 错误 |
| `LFS_ERR_CORRUPT` | -84 | 损坏错误 |
| `LFS_ERR_NOENT` | -2 | 不存在 |
| `LFS_ERR_EXIST` | -17 | 已存在 |
| `LFS_ERR_NOTDIR` | -20 | 不是目录 |
| `LFS_ERR_ISDIR` | -21 | 是目录 |
| `LFS_ERR_NOTEMPTY` | -39 | 目录非空 |
| `LFS_ERR_BADF` | -9 | 坏文件描述符 |
| `LFS_ERR_FBIG` | -27 | 文件过大 |
| `LFS_ERR_INVAL` | -22 | 无效参数 |
| `LFS_ERR_NOSPC` | -28 | 无空间 |
| `LFS_ERR_NOMEM` | -12 | 内存不足 |

### VFS 错误码映射

| LittleFS 错误码 | VFS 错误码 | errno |
|----------------|-----------|-------|
| `LFS_ERR_OK` | 0 | 0 |
| `LFS_ERR_IO` | -1 | EIO |
| `LFS_ERR_CORRUPT` | -1 | EIO |
| `LFS_ERR_NOENT` | -1 | ENOENT |
| `LFS_ERR_EXIST` | -1 | EEXIST |
| `LFS_ERR_NOTDIR` | -1 | ENOTDIR |
| `LFS_ERR_ISDIR` | -1 | EISDIR |
| `LFS_ERR_NOTEMPTY` | -1 | ENOTEMPTY |
| `LFS_ERR_BADF` | -1 | EBADF |
| `LFS_ERR_FBIG` | -1 | EFBIG |
| `LFS_ERR_INVAL` | -1 | EINVAL |
| `LFS_ERR_NOSPC` | -1 | ENOSPC |
| `LFS_ERR_NOMEM` | -1 | ENOMEM |

---

## API 使用示例

### 文件操作

```c
#include "lfs.h"

lfs_t lfs;
lfs_file_t file;

// 挂载文件系统
int err = lfs_mount(&lfs, &cfg);
if (err) {
    lfs_format(&lfs, &cfg);
    lfs_mount(&lfs, &cfg);
}

// 打开文件
err = lfs_file_open(&lfs, &file, "hello.txt", LFS_O_RDWR | LFS_O_CREAT);
if (err < 0) {
    printf("open failed: %d\n", err);
}

// 写入文件
lfs_file_write(&lfs, &file, "Hello World", 11);

// 读取文件
char buffer[64];
lfs_ssize_t len = lfs_file_read(&lfs, &file, buffer, sizeof(buffer));
printf("read: %.*s\n", (int)len, buffer);

// 关闭文件
lfs_file_close(&lfs, &file);

// 卸载文件系统
lfs_unmount(&lfs);
```

### 目录操作

```c
#include "lfs.h"

// 创建目录
int err = lfs_mkdir(&lfs, "mydir");
if (err < 0) {
    printf("mkdir failed: %d\n", err);
}

// 打开目录
lfs_dir_t dir;
err = lfs_dir_open(&lfs, &dir, "mydir");
if (err < 0) {
    printf("opendir failed: %d\n", err);
}

// 读取目录
struct lfs_info info;
while (lfs_dir_read(&lfs, &dir, &info) > 0) {
    printf("%s (type=%d, size=%d)\n", info.name, info.type, info.size);
}

// 关闭目录
lfs_dir_close(&lfs, &dir);
```

### 扩展属性

```c
#include "lfs.h"

// 设置属性
const char *value = "myvalue";
int err = lfs_setattr(&lfs, "file.txt", 1, value, strlen(value));
if (err < 0) {
    printf("setattr failed: %d\n", err);
}

// 获取属性
char buffer[64];
lfs_ssize_t len = lfs_getattr(&lfs, "file.txt", 1, buffer, sizeof(buffer));
if (len >= 0) {
    printf("attr: %.*s\n", (int)len, buffer);
}

// 删除属性
err = lfs_removeattr(&lfs, "file.txt", 1);
if (err < 0) {
    printf("removeattr failed: %d\n", err);
}
```

---

## API 兼容性说明

### 与上游兼容性

| 项目 | OH 版本 | 上游版本 | 兼容性 |
|------|---------|----------|--------|
| **核心 API** | v2.11.2 | v2.11.2 | ✅ 完全兼容 |
| **数据结构** | v2.11.2 | v2.11.2 | ✅ 完全兼容 |
| **错误码** | v2.11.2 | v2.11.2 | ✅ 完全兼容 |
| **磁盘格式** | lfs2.1 | lfs2.1 | ✅ 完全兼容 |

### POSIX 兼容性

通过 VFS 适配层，littlefs 在 OH 中提供 POSIX 兼容接口：

| POSIX API | 支持状态 | 说明 |
|-----------|----------|------|
| `open()` | ✅ 支持 | 支持标准 flags |
| `close()` | ✅ 支持 | - |
| `read()` | ✅ 支持 | - |
| `write()` | ✅ 支持 | - |
| `lseek()` | ✅ 支持 | 支持标准 whence |
| `stat()` | ✅ 支持 | - |
| `fstat()` | ✅ 支持 | - |
| `unlink()` | ✅ 支持 | - |
| `rename()` | ✅ 支持 | - |
| `fsync()` | ✅ 支持 | - |
| `mkdir()` | ✅ 支持 | - |
| `rmdir()` | ✅ 支持 | - |
| `opendir()` | ✅ 支持 | - |
| `closedir()` | ✅ 支持 | - |
| `readdir()` | ✅ 支持 | - |
| `rewinddir()` | ✅ 支持 | - |

---

## 总结

### 关键发现

1. **无 API 差异**: littlefs 核心 API 与上游 v2.11.2 完全一致
2. **VFS 适配层**: 通过 lfs_adapter.c 提供 POSIX 兼容接口
3. **HAL 层**: 通过 littlefs_hal.c 提供块设备操作接口
4. **配置外置**: 编译时宏和运行时配置参数外置

### 最佳实践

1. **使用 POSIX API**: 在 OH 应用中使用标准 POSIX API，而非直接调用 littlefs API
2. **错误处理**: 正确处理错误码，检查返回值
3. **线程安全**: 多线程环境启用 `LFS_THREADSAFE` 并提供 lock/unlock 回调
4. **资源管理**: 及时关闭文件和目录，释放资源

---

**最后更新时间**: 2026-02-08
**上游版本**: v2.11.2
**OH 版本**: 3.1
**API 差异**: 无
