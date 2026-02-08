# OH 构建适配

## 3.1 构建系统概述

### 构建架构

NuttX 代码通过 OpenHarmony 的 **GN（Generate Ninja）构建系统**集成，而不是使用 NuttX 原生的 Kconfig/Make 构建系统。

```
┌─────────────────────────────────────────────────────────────┐
│              OpenHarmony 构建系统架构                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  //build/lite/config/                                        │
│      └── component/lite_component.gni  ◄── 组件定义          │
│                                                              │
│  //kernel/liteos_a/                                          │
│      ├── liteos.gni  ◄── LiteOS 构建配置                     │
│      ├── BUILD.gn    ◄── 内核根构建文件                       │
│      └── fs/vfs/                                          │
│          └── BUILD.gn  ◄── VFS 模块构建                      │
│                                                              │
│  //third_party/NuttX/                                        │
│      └── NuttX.gni  ◄── NuttX 源文件配置                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 构建集成流程

```
1. 定义源文件
   NuttX.gni 定义所有复用模块的源文件列表
   
2. 导入配置
   kernel/liteos_a/.../BUILD.gn import NuttX.gni
   
3. 构建模块
   GN + Ninja 编译 NuttX 源文件
   
4. 链接产物
   生成内核镜像（包含 NuttX 代码）
```

---

## 3.2 NuttX.gni 配置文件详解

### 文件信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `//third_party/NuttX/NuttX.gni` |
| **版权** | 2022-2022 Huawei Device Co., Ltd. |
| **许可证** | BSD 3-Clause |
| **行数** | 131 行 |

### 配置结构

```bash
# ============================================================
# NuttX 源文件配置
# ============================================================

# ---------- 驱动程序源文件 ----------

NUTTX_DRIVERS_BCH_SRC_FILES = [
  "//third_party/NuttX/drivers/bch/bchdev_driver.c",
  "//third_party/NuttX/drivers/bch/bchdev_register.c",
  "//third_party/NuttX/drivers/bch/bchdev_unregister.c",
  "//third_party/NuttX/drivers/bch/bchlib_cache.c",
  "//third_party/NuttX/drivers/bch/bchlib_read.c",
  "//third_party/NuttX/drivers/bch/bchlib_sem.c",
  "//third_party/NuttX/drivers/bch/bchlib_setup.c",
  "//third_party/NuttX/drivers/bch/bchlib_teardown.c",
  "//third_party/NuttX/drivers/bch/bchlib_write.c",
]

NUTTX_DRIVERS_PIPES_SRC_FILES = [
  "//third_party/NuttX/drivers/pipes/fifo.c",
  "//third_party/NuttX/drivers/pipes/pipe.c",
  "//third_party/NuttX/drivers/pipes/pipe_common.c",
]

NUTTX_DRIVERS_PIPES_INCLUDE_DIRS = [ "//third_party/NuttX/drivers/pipes" ]

NUTTX_DRIVERS_VIDEO_SRC_FILES = [ "//third_party/NuttX/drivers/video/fb.c" ]

NUTTX_DRIVERS_VIDEO_INCLUDE_DIRS = [ "//third_party/NuttX/include/nuttx/video" ]

# ---------- 文件系统源文件 ----------

NUTTX_FS_DIRENT_SRC_FILES = [
  "//third_party/NuttX/fs/dirent/fs_closedir.c",
  "//third_party/NuttX/fs/dirent/fs_opendir.c",
  "//third_party/NuttX/fs/dirent/fs_readdir.c",
  "//third_party/NuttX/fs/dirent/fs_rewinddir.c",
  "//third_party/NuttX/fs/dirent/fs_seekdir.c",
  "//third_party/NuttX/fs/dirent/fs_telldir.c",
]

NUTTX_FS_DRIVER_SRC_FILES = [
  "//third_party/NuttX/fs/driver/fs_blockproxy.c",
  "//third_party/NuttX/fs/driver/fs_closeblockdriver.c",
  "//third_party/NuttX/fs/driver/fs_findblockdriver.c",
  "//third_party/NuttX/fs/driver/fs_openblockdriver.c",
  "//third_party/NuttX/fs/driver/fs_registerblockdriver.c",
  "//third_party/NuttX/fs/driver/fs_registerdriver.c",
  "//third_party/NuttX/fs/driver/fs_unregisterblockdriver.c",
  "//third_party/NuttX/fs/driver/fs_unregisterdriver.c",
]

NUTTX_FS_INODE_SRC_FILES = [ "//third_party/NuttX/fs/inode/fs_files.c" ]

NUTTX_FS_MOUNT_SRC_FILES = [
  "//third_party/NuttX/fs/mount/fs_foreachmountpoint.c",
  "//third_party/NuttX/fs/mount/fs_mount.c",
  "//third_party/NuttX/fs/mount/fs_sync.c",
  "//third_party/NuttX/fs/mount/fs_umount.c",
]

NUTTX_FS_VFS_SRC_FILES = [
  "//third_party/NuttX/fs/vfs/fs_close.c",
  "//third_party/NuttX/fs/vfs/fs_dup.c",
  "//third_party/NuttX/fs/vfs/fs_dup2.c",
  "//third_party/NuttX/fs/vfs/fs_dupfd.c",
  "//third_party/NuttX/fs/vfs/fs_dupfd2.c",
  "//third_party/NuttX/fs/vfs/fs_fcntl.c",
  "//third_party/NuttX/fs/vfs/fs_fsync.c",
  "//third_party/NuttX/fs/vfs/fs_getfilep.c",
  "//third_party/NuttX/fs/vfs/fs_ioctl.c",
  "//third_party/NuttX/fs/vfs/fs_link.c",
  "//third_party/NuttX/fs/vfs/fs_lseek.c",
  "//third_party/NuttX/fs/vfs/fs_lseek64.c",
  "//third_party/NuttX/fs/vfs/fs_mkdir.c",
  "//third_party/NuttX/fs/vfs/fs_open.c",
  "//third_party/NuttX/fs/vfs/fs_poll.c",
  "//third_party/NutttX/fs/vfs/fs_pread.c",
  "//third_party/NuttX/fs/vfs/fs_pread64.c",
  "//third_party/NuttX/fs/vfs/fs_pwrite.c",
  "//third_party/NuttX/fs/vfs/fs_pwrite64.c",
  "//third_party/NuttX/fs/vfs/fs_read.c",
  "//third_party/NuttX/fs/vfs/fs_readlink.c",
  "//third_party/NuttX/fs/vfs/fs_rename.c",
  "//third_party/NuttX/fs/vfs/fs_rmdir.c",
  "//third_party/NuttX/fs/vfs/fs_select.c",
  "//third_party/NuttX/fs/vfs/fs_sendfile.c",
  "//third_party/NuttX/fs/vfs/fs_stat.c",
  "//third_party/NuttX/fs/vfs/fs_statfs.c",
  "//third_party/NuttX/fs/vfs/fs_symlink.c",
  "//third_party/NuttX/fs/vfs/fs_truncate.c",
  "//third_party/NuttX/fs/vfs/fs_truncate64.c",
  "//third_party/NuttX/fs/vfs/fs_unlink.c",
  "//third_party/NuttX/fs/vfs/fs_write.c",
]

NUTTX_FS_NFS_SRC_FILES = [
  "//third_party/NuttX/fs/nfs/nfs_adapter.c",
  "//third_party/NuttX/fs/nfs/nfs_util.c",
  "//third_party/NuttX/fs/nfs/rpc_clnt.c",
]

NUTTX_FS_TMPFS_SRC_FILES = [ "//third_party/NuttX/fs/tmpfs/fs_tmpfs.c" ]

NUTTX_FS_ROMFS_SRC_FILES = [
  "//third_party/NuttX/fs/romfs/fs_romfs.c",
  "//third_party/NuttX/fs/romfs/fs_romfsutil.c",
]
```

---

## 3.3 模块列表

### 模块统计

| 模块类别 | 模块数量 | 源文件数量 |
|---------|---------|-----------|
| 驱动程序 | 3 | 13 |
| 文件系统 | 6 | 52 |
| **总计** | **9** | **65** |

### 驱动程序模块

| 模块 | 源文件数 | 主要功能 |
|-----|---------|---------|
| **NUTTX_DRIVERS_BCH** | 9 | 块设备缓存（Block Cache）驱动 |
| **NUTTX_DRIVERS_PIPES** | 3 | 管道（Pipe/FIFO）驱动 |
| **NUTTX_DRIVERS_VIDEO** | 1 | 帧缓冲（Framebuffer）显示驱动 |

### 文件系统模块

| 模块 | 源文件数 | 主要功能 |
|-----|---------|---------|
| **NUTTX_FS_DIRENT** | 6 | 目录操作（opendir、readdir 等） |
| **NUTTX_FS_DRIVER** | 8 | 块设备驱动注册管理 |
| **NUTTX_FS_INODE** | 1 | inode 节点管理 |
| **NUTTX_FS_MOUNT** | 4 | 文件系统挂载操作 |
| **NUTTX_FS_VFS** | 27 | 虚拟文件系统核心 |
| **NUTTX_FS_NFS** | 3 | NFS 网络文件系统 |
| **NUTTX_FS_TMPFS** | 1 | 临时文件系统 |
| **NUTTX_FS_ROMFS** | 2 | ROM 只读文件系统 |

---

## 3.4 与 OH BUILD.gn 的集成

### 集成方式

OH 模块通过 `import()` 导入 `NuttX.gni`，然后在 `sources` 中使用预定义的变量：

```bash
# kernel/liteos_a/fs/vfs/BUILD.gn 片段

import("//kernel/liteos_a/liteos.gni")
import("$THIRDPARTY_NUTTX_DIR/NuttX.gni")  # ◄── 导入 NuttX 配置

module_switch = defined(LOSCFG_FS_VFS)
module_name = get_path_info(rebase_path("."), "name")

kernel_module(module_name) {
  sources = [
    # VFS 特有的实现
    "epoll/fs_epoll.c",
    "mount.c",
    "operation/fullpath.c",
    # ... 其他 VFS 特有代码
    
    # ◄── 包含 NuttX VFS 源文件
    sources += NUTTX_FS_DIRENT_SRC_FILES
    sources += NUTTX_FS_DRIVER_SRC_FILES
    sources += NUTTX_FS_INODE_SRC_FILES
    sources += NUTTX_FS_MOUNT_SRC_FILES
    sources += NUTTX_FS_VFS_SRC_FILES
  ]
  
  include_dirs = [
    "//third_party/NuttX/include",
    "//third_party/NuttX/drivers/pipes",
    "//third_party/NuttX/include/nuttx/video",
  ]
}
```

---

## 3.5 依赖模块详细说明

### 3.5.1 文件系统模块

#### VFS 模块（fs/vfs）

**BUILD.gn 路径**：`//kernel/liteos_a/fs/vfs/BUILD.gn`

**功能**：虚拟文件系统层，提供统一的文件操作接口

**包含的 NuttX 源文件**：
- DIRENT：目录操作（6 个文件）
- DRIVER：驱动注册（8 个文件）
- INODE：inode 管理（1 个文件）
- MOUNT：挂载操作（4 个文件）
- VFS：核心操作（27 个文件）

**特有代码**：
- `epoll/`：epoll 事件通知
- `operation/`：VFS 操作实现

#### RAMFS 模块（fs/ramfs）

**BUILD.gn 路径**：`//kernel/liteos_a/fs/ramfs/BUILD.gn`

**功能**：基于内存的临时文件系统

**包含的 NuttX 源文件**：
- `NUTTX_FS_TMPFS_SRC_FILES`（1 个文件）

#### ROMFS 模块（fs/romfs）

**BUILD.gn 路径**：`//kernel/liteos_a/fs/romfs/BUILD.gn`

**功能**：ROM 只读文件系统，用于存储只读资源

**包含的 NuttX 源文件**：
- `NUTTX_FS_ROMFS_SRC_FILES`（2 个文件）

#### NFS 模块（fs/nfs）

**BUILD.gn 路径**：`//kernel/liteos_a/fs/nfs/BUILD.gn`

**功能**：网络文件系统，支持远程文件访问

**包含的 NuttX 源文件**：
- `NUTTX_FS_NFS_SRC_FILES`（3 个文件）

### 3.5.2 驱动程序模块

#### BCH 驱动（drivers/char/bch）

**BUILD.gn 路径**：`//kernel/liteos_a/drivers/char/bch/BUILD.gn`

**功能**：块设备缓存驱动，提高块设备访问效率

**包含的 NuttX 源文件**：
- `NUTTX_DRIVERS_BCH_SRC_FILES`（9 个文件）

#### Video 驱动（drivers/char/video）

**BUILD.gn 路径**：`//kernel/liteos_a/drivers/char/video/BUILD.gn`

**功能**：帧缓冲显示驱动，支持图形显示

**包含的 NuttX 源文件**：
- `NUTTX_DRIVERS_VIDEO_SRC_FILES`（1 个文件）

**头文件路径**：
- `NUTTX_DRIVERS_VIDEO_INCLUDE_DIRS`

#### Pipes 驱动（kernel/extended/pipes）

**BUILD.gn 路径**：`//kernel/liteos_a/kernel/extended/pipes/BUILD.gn`

**功能**：管道 IPC 机制，支持进程间通信

**包含的 NuttX 源文件**：
- `NUTTX_DRIVERS_PIPES_SRC_FILES`（3 个文件）

**头文件路径**：
- `NUTTX_DRIVERS_PIPES_INCLUDE_DIRS`

### 3.5.3 USB 驱动

**BUILD.gn 路径**：`//drivers/hdf_core/adapter/khdf/liteos/model/usb/device/BUILD.gn`

**功能**：USB 设备驱动

**引用**：
```bash
USB_DEVICE_ROOT = "$LITEOSTHIRDPARTY/NuttX/drivers/usbdev"
```

---

## 3.6 构建配置选项

### 模块开关

NuttX 模块的启用/禁用通过 OH 的 Kconfig 配置系统控制：

| 模块 | Kconfig 选项 | 默认值 |
|-----|-------------|-------|
| VFS | `LOSCFG_FS_VFS` | 启用 |
| RAMFS | `LOSCFG_FS_RAMFS` | 启用 |
| ROMFS | `LOSCFG_FS_ROMFS` | 启用 |
| NFS | `LOSCFG_FS_NFS` | 可选 |
| BCH | `LOSCFG_DRIVERS_CHAR_BCH` | 启用 |
| Video | `LOSCFG_DRIVERS_CHAR_VIDEO` | 启用 |
| Pipes | `LOSCFG_KERNEL_EXTENDED_PIPES` | 启用 |

### 配置示例

```bash
# kernel/liteos_a/fs/vfs/BUILD.gn

module_switch = defined(LOSCFG_FS_VFS)  # ◄── 根据 Kconfig 开关
module_name = get_path_info(rebase_path("."), "name")

kernel_module(module_name) {
  if (module_switch) {
    sources = [ ... ]  # 包含 NuttX 源文件
  }
}
```

---

## 3.7 与上游构建系统的差异

### 构建系统对比

| 特性 | NuttX 原生 | OH 集成 |
|-----|-----------|--------|
| **构建工具** | Make/Kconfig | GN/Ninja |
| **配置方式** | Kconfig | GN + Kconfig |
| **文件选择** | 整个源码树 | NuttX.gni 精确选择 |
| **头文件路径** | 自动搜索 | 手动指定 |
| **模块开关** | Kconfig | Kconfig + GN 条件 |

### 差异处理

OH 集成 NuttX 时需要处理以下差异：

| 差异 | 处理方式 |
|-----|---------|
| **头文件路径** | 通过 `include_dirs` 显式指定 |
| **条件编译** | 使用 GN 条件表达式 |
| **依赖声明** | 在 BUILD.gn 中声明 deps |
| **编译选项** | 通过 `cflags` 传递 |

---

## 3.8 构建验证

### 构建命令

```bash
# 完整构建
hb build

# 仅构建内核
hb build -p kernel

# 清理构建
hb build -c
```

### 构建产物

```
out/<product>/
├── liteos.bin          # 内核镜像
├── OHOS_Image          # 系统镜像
└── kernel.map          # 内核符号表
```

### 验证点

| 验证项 | 预期结果 |
|-------|---------|
| NuttX 源文件编译 | 无编译错误 |
| 链接产物大小 | 符合预期 ROM 700KB |
| 功能测试 | VFS 操作正常 |

---

## 3.9 常见问题

### Q1: 如何添加新的 NuttX 模块？

1. 在 `NuttX.gni` 中添加源文件列表
2. 在目标模块的 BUILD.gn 中导入并使用
3. 验证构建

### Q2: 如何禁用不需要的 NuttX 模块？

1. 在 Kconfig 中关闭对应选项
2. 对应模块将不会被编译

### Q3: 升级 NuttX 版本后需要做什么？

1. 同步新的源文件到 `third_party/NuttX/`
2. 更新 `NuttX.gni` 中的文件列表
3. 验证构建和测试

---

## 3.10 总结

### 构建特点

1. **选择性集成**：通过 NuttX.gni 精确选择文件
2. **模块化构建**：按模块独立编译
3. **条件编译**：通过 Kconfig 控制模块开关
4. **统一构建**：使用 OH GN 构建系统

### 配置清单

| 配置项 | 文件 | 说明 |
|-------|------|------|
| 源文件定义 | `NuttX.gni` | 所有复用模块的源文件列表 |
| 驱动程序集成 | `kernel/liteos_a/drivers/char/*/BUILD.gn` | BCH、Video 驱动 |
| 文件系统集成 | `kernel/liteos_a/fs/*/BUILD.gn` | VFS、RAMFS、ROMFS、NFS |
| IPC 集成 | `kernel/liteos_a/kernel/extended/pipes/BUILD.gn` | 管道驱动 |
