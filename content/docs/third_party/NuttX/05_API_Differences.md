# API/接口差异

## 5.1 概述

### 本文档目的

本章节说明 NuttX 代码在 OpenHarmony 中使用时的 **API 兼容性**、**接口变更** 和 **OH 特有扩展**。

### NuttX API 使用原则

NuttX 在 OH 中的集成遵循以下原则：

1. **保持上游兼容性**：尽量保持 NuttX 原生 API 不变
2. **最小修改**：只做必要的适配修改
3. **清晰边界**：OH 特有代码与 NuttX 代码分离

---

## 5.2 API 兼容性状态

### 整体兼容性评估

| API 类别 | 兼容性 | 说明 |
|---------|-------|------|
| **VFS API** | 完全兼容 | POSIX 标准接口，无修改 |
| **文件系统 API** | 完全兼容 | DIRENT、MOUNT 等保持原样 |
| **设备驱动 API** | 完全兼容 | BCH、Video 等框架接口未变 |
| **IPC API** | 完全兼容 | Pipe/FIFO 接口保持 POSIX 标准 |
| **系统调用** | 完全兼容 | open、read、write 等标准调用 |

**结论**：NuttX API 在 OH 中保持 **100% 上游兼容性**，未发现功能性的 API 差异。

---

## 5.3 POSIX API 兼容性

### 标准 POSIX 接口

NuttX 提供的 POSIX 接口在 OH 中完全支持：

| 接口类别 | 示例函数 | 状态 |
|---------|---------|------|
| **文件操作** | `open()`, `read()`, `write()`, `close()` | ✅ 兼容 |
| **目录操作** | `opendir()`, `readdir()`, `closedir()` | ✅ 兼容 |
| **文件属性** | `stat()`, `fstat()`, `lstat()` | ✅ 兼容 |
| **文件系统** | `mount()`, `umount()`, `sync()` | ✅ 兼容 |
| **I/O 控制** | `ioctl()`, `fcntl()` | ✅ 兼容 |
| **管道** | `pipe()`, `mkfifo()` | ✅ 兼容 |
| **文件描述符** | `dup()`, `dup2()`, `select()` | ✅ 兼容 |

### 使用示例

```c
// 所有这些 POSIX 接口在 OH 中正常工作

// 文件操作
int fd = open("/data/file.txt", O_RDWR | O_CREAT);
ssize_t n = read(fd, buf, sizeof(buf));
ssize_t m = write(fd, buf, sizeof(buf));
close(fd);

// 目录操作
DIR *dir = opendir("/system");
struct dirent *ent = readdir(dir);
closedir(dir);

// 高级操作
int ret = stat("/path", &st);
int fd2 = dup(fd);
fd_set fds;
select(maxfd + 1, &fds, NULL, NULL, NULL);
```

---

## 5.4 OH 特有扩展

### 5.4.1 内核集成扩展

#### VFS 操作扩展

**文件**：`//kernel/liteos_a/fs/vfs/operation/*.c`

OH 在 NuttX VFS 基础上添加了特有操作：

| 函数 | 用途 |
|-----|------|
| `vfs_init()` | VFS 初始化（OH 特有） |
| `vfs_chattr()` | 文件属性修改（OH 扩展） |
| `vfs_check()` | 文件检查（OH 扩展） |
| `vfs_fallocate()` | 文件空间分配（扩展） |
| `vfs_cloexec()` | 执行时关闭标志管理 |

#### 模块初始化

OH 特有的初始化机制：

```c
// kernel/liteos_a/fs/vfs/operation/vfs_init.c

/* OH 特有的初始化函数 */
int VfsInit(void)
{
    /* NuttX 基础组件初始化 */
    NuttX_VfsBasicInit();
    
    /* OH 特有初始化 */
    OH_VfsOhSpecificInit();
    
    return 0;
}
```

### 5.4.2 HDF 集成

#### USB 驱动适配

NuttX 的 USB 设备驱动被适配到 OH 的 HDF 框架：

```c
// drivers/hdf_core/adapter/khdf/liteos/model/usb/device/...

/* OH HDF 适配层 */
struct UsbDeviceOps {
    int (*init)(void);
    int (*submit)(struct UsbRequest *req);
    int (*cancel)(int token);
    /* ... */
};

/* 适配 NuttX USBDEV 到 OH HDF */
static int NuttXUsbToHdf(struct NuttXUsbDevice *nuttx_dev)
{
    /* 转换 NuttX USBDEV 接口到 OH HDF 接口 */
    return 0;
}
```

---

## 5.5 接口差异详解

### 5.5.1 文件描述符管理

**状态**：无差异

NuttX 的文件描述符管理在 OH 中完全保留：

```c
/* 标准 NuttX/OH 兼容代码 */

int fd = open("/dev/null", O_RDWR);
if (fd < 0) {
    return -1;
}

/* dup/dup2 完全兼容 */
int fd2 = dup(fd);
int fd3 = dup2(fd, 10);

/* close 行为一致 */
close(fd);
close(fd2);
close(fd3);
```

### 5.5.2 目录遍历

**状态**：无差异

```c
/* 标准 POSIX 目录操作 */

DIR *dir = opendir("/system");
if (!dir) {
    return -1;
}

struct dirent *ent;
while ((ent = readdir(dir)) != NULL) {
    printf("Found: %s\n", ent->d_name);
}

closedir(dir);
```

### 5.5.3 文件挂载

**状态**：无差异

```c
/* 标准挂载接口 */

int ret = mount("/dev/block0", "/mnt", "vfat", 0, NULL);
if (ret < 0) {
    perror("mount failed");
}

/* 卸载 */
umount("/mnt");
sync();
```

---

## 5.6 头文件使用

### 5.6.1 必需头文件

使用 NuttX API 需要包含正确的头文件：

| 功能 | 头文件 |
|-----|-------|
| 基础 I/O | `<fcntl.h>`, `<unistd.h>`, `<sys/stat.h>` |
| 目录操作 | `<dirent.h>` |
| 设备操作 | `<nuttx/drivers/bch.h>`, `<nuttx/video/fb.h>` |
| 视频帧缓冲 | `<nuttx/video/fb.h>` |

### 5.6.2 头文件搜索路径

OH 构建系统自动添加以下头文件路径：

```
third_party/NuttX/include/
third_party/NuttX/include/nuttx/
third_party/NuttX/include/nuttx/video/
third_party/NuttX/drivers/pipes/
```

---

## 5.7 API 使用注意事项

### 5.7.1 错误处理

NuttX 使用标准的 POSIX 错误码：

| 错误码 | 含义 |
|-------|------|
| `ENOENT` | 文件不存在 |
| `EACCES` | 权限拒绝 |
| `EBADF` | 错误的文件描述符 |
| `EINVAL` | 无效参数 |
| `ENOSPC` | 空间不足 |
| `EIO` | I/O 错误 |

### 5.7.2 阻塞 vs 非阻塞

```c
/* 非阻塞 I/O 示例 */

int fd = open("/dev/null", O_RDWR | O_NONBLOCK);

char buf[64];
ssize_t n = read(fd, buf, sizeof(buf));
if (n < 0) {
    if (errno == EAGAIN) {
        /* 数据暂时不可用 */
    }
}
```

---

## 5.8 与上游的差异总结

### 已确认的无差异领域

| 领域 | 状态 | 说明 |
|-----|------|------|
| **POSIX 文件 I/O** | ✅ 无差异 | open/read/write/close |
| **目录操作** | ✅ 无差异 | opendir/readdir/closedir |
| **文件属性** | ✅ 无差异 | stat/fstat/lstat |
| **挂载管理** | ✅ 无差异 | mount/umount/sync |
| **管道 IPC** | ✅ 无差异 | pipe/mkfifo |
| **描述符操作** | ✅ 无差异 | dup/dup2/fcntl |
| **I/O 多路复用** | ✅ 无差异 | select/poll |

### 潜在的 OH 扩展（待确认）

| 领域 | 潜在扩展 | 状态 | 说明 |
|-----|---------|------|------|
| **VFS 初始化** | `VfsInit()` | ⚠️ 待确认 | OH 特有初始化 |
| **HDF 适配** | USB 驱动适配 | ⚠️ 待确认 | OH 设备框架 |
| **权限管理** | OH 特有权限 | ⚠️ 待确认 | 安全子系统 |

---

## 5.9 迁移指南

### 从 Linux 迁移

NuttX API 与 Linux POSIX 高度兼容，迁移时需注意：

| Linux 特性 | NuttX 支持 | 说明 |
|-----------|-----------|------|
| `epoll` | ✅ | 支持 epoll_create/epoll_ctl/epoll_wait |
| `inotify` | ❌ | 不支持，建议使用其他机制 |
| `AIO` | ⚠️ | 有限支持 |
| `sendfile` | ✅ | 支持 `sendfile()` |

### 从其他 RTOS 迁移

| 源系统 | 迁移难度 | 主要差异 |
|-------|---------|---------|
| FreeRTOS | 中 | API 完全不同 |
| Zephyr | 中 | 需适配驱动框架 |
| RT-Thread | 低 | 类似 POSIX 接口 |
| VxWorks | 低 | 兼容大部分 POSIX |

---

## 5.10 总结

### API 兼容性结论

1. **核心 API 完全兼容**：NuttX 的 POSIX 兼容 API 在 OH 中 100% 可用
2. **无破坏性变更**：上游 API 无修改，保持向后兼容
3. **最小 OH 扩展**：只在必要时添加 OH 特有功能
4. **清晰的分层**：NuttX 代码与 OH 特有代码分离

### 开发者建议

- ✅ 使用标准 POSIX API，无需担心兼容性问题
- ✅ 参考 NuttX 官方文档获取 API 详细信息
- ⚠️ 注意 NuttX 与 Linux 的细微差异（如 inotify）
- ⚠️ OH 特有扩展需要在 OH 文档中查阅
