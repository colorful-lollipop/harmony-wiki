# API 与接口差异分析

## 一、概述

### 1.1 差异分析结论

经过对 NTFS-3G 库源代码和构建配置的全面分析，**该库在 OpenHarmony 中的 API 与上游版本不存在显著差异**。这一结论基于以下观察：

**第一，无源代码修改**：该库没有任何 Patch 文件，源代码与上游版本完全一致。API 定义头文件（位于 include/ntfs-3g/ 和 include/fuse-lite/ 目录）未经过 OpenHarmony 特定修改。

**第二，构建配置隔离**：所有 OpenHarmony 适配工作都通过 BUILD.gn 构建配置和预生成的 config.h 文件完成，未触及 API 层面。

**第三，功能子集化**：OpenHarmony 版本通过禁用某些编译选项（如 ENABLE_CRYPTO、ENABLE_DEBUG）来排除特定功能，而非修改 API。这种方式保持了 API 的完整性，只是限制了可用功能。

### 1.2 文档目的

本文档旨在说明以下事项：

记录 OpenHarmony 版本相对于上游版本的功能裁剪情况。

提供 API 使用时的兼容性说明。

指导开发者在使用 NTFS-3G API 时需要注意的事项。

## 二、功能差异

### 2.1 启用的功能

以下功能在 OpenHarmony 版本的 NTFS-3G 中保持启用状态，API 完整可用：

| 功能模块 | 功能描述 | API 可用性 |
|---------|---------|-----------|
| **核心 NTFS 读写** | 文件和目录的创建、读取、写入、删除 | 完整可用 |
| **大文件支持** | 支持大于 4GB 的文件 | 完整可用 |
| **扩展属性** | xattr 操作 | 完整可用 |
| **符号链接** | 软链接和交接点 | 完整可用 |
| **FUSE 接口** | 用户空间文件系统框架 | 完整可用 |
| **基本工具程序** | fsck.ntfs、mount.ntfs、ntfsfix、ntfslabel | 完整可用 |

### 2.2 禁用的功能

以下功能在 OpenHarmony 版本中被禁用，相关的 API 和工具不可用：

| 功能 | config.h 宏 | 禁用原因 |
|------|------------|---------|
| **插件支持** | DISABLE_PLUGINS | 减少复杂性，降低攻击面 |
| **加密文件支持** | ENABLE_CRYPTO | 简化安全审计，避免依赖 |
| **调试功能** | ENABLE_DEBUG | 生产环境优化 |
| **DCE UUID 生成** | ENABLE_UUID | 非核心功能 |
| **nfconv 补丁** | ENABLE_NFCONV | 非必要功能 |
| **Windows 磁盘几何** | ENABLE_HD | 移动设备通常不需要 |

**开发者注意**：如果应用依赖上述任何被禁用的功能，需要重新评估兼容性或考虑启用相应编译选项（需要修改 config.h）。

## 三、兼容性说明

### 3.1 POSIX 兼容性

NTFS-3G 的核心 API 设计遵循 POSIX 标准，在 OpenHarmony（作为类 Unix 系统）上具有良好的兼容性。

**支持的 POSIX 接口**：

- 文件操作：open、read、write、close、lseek 等标准文件 I/O。
- 目录操作：opendir、readdir、closedir 等目录遍历接口。
- 文件属性：stat、fstat、chmod、chown 等属性操作。
- 符号链接：readlink、symlink 等链接操作。

**注意事项**：虽然 API 兼容，但 NTFS 文件系统的某些特性（如文件权限模型）与 POSIX 存在差异。在跨平台场景下，建议测试文件权限的预期行为。

### 3.2 FUSE 接口兼容性

libfuse-lite 提供了 FUSE 接口的完整实现，与标准 FUSE API 保持兼容。

**可用 FUSE 操作**：

- fuse_main：FUSE 程序入口。
- fuse_new：创建 FUSE 实例。
- fuse_mount：挂载文件系统。
- fuse_loop：事件循环。

**注意事项**：libfuse-lite 是 FUSE 的轻量级实现，某些高级 FUSE 选项可能不可用。建议参考 libfuse-lite 源码中的实现限制。

### 3.3 编译器兼容性

OpenHarmony 版本使用标准 C 编译器（GCC 或 Clang）编译，与上游版本保持一致。

**编译标志**：

| 标志 | 值 | 说明 |
|------|-----|------|
| 标准 | C99 或更高 | 标准 C 编程 |
| 警告 | -Wno-error, -Wno-address-of-packed-member | 宽松警告策略 |
| 大文件 | -D_LARGEFILE_SOURCE, -D_FILE_OFFSET_BITS=64 | 大文件支持 |

### 3.4 平台特定行为

虽然 API 层面无差异，但某些行为可能因平台差异而略有不同：

**路径处理**：Windows 使用反斜杠（\\）作为路径分隔符，而 Unix 使用正斜杠（/）。NTFS-3G API 统一使用 Unix 风格路径。

**字符编码**：Windows 使用 UTF-16 存储文件名，Unix 使用 UTF-8。NTFS-3G 在内部进行编码转换。

**权限模型**：NTFS 使用 ACL（访问控制列表），POSIX 使用简单的 owner/group/other 权限模型。NTFS-3G 提供映射机制。

## 四、API 使用指南

### 4.1 头文件包含

使用 NTFS-3G API 时，需要包含正确的头文件：

```c
/* NTFS 核心 API */
#include <ntfs-3g/ntfs.h>
#include <ntfs-3g/volume.h>
#include <ntfs-3g/inode.h>
#include <ntfs-3g/attrib.h>
#include <ntfs-3g/types.h>

/* FUSE 接口 */
#include <fuse-lite/fuse.h>
#include <fuse-lite/fuse_lowlevel.h>
```

### 4.2 编译链接

在 OpenHarmony 项目中编译使用 NTFS-3G API 的代码时，需要正确配置编译和链接选项：

**BUILD.gn 配置**：

```gn
deps += [
  "//third_party/ntfs-3g/libntfs-3g:libntfs_3g",
]

include_dirs += [
  "//third_party/ntfs-3g/include",
  "//third_party/ntfs-3g/include/ntfs-3g",
]
```

### 4.3 常见 API 使用模式

**模式一：挂载 NTFS 卷**

```c
#include <ntfs-3g/ntfs.h>

ntfs_volume *vol;
ntfschar ntfs_uc[] = { 'D', ':', '\\', 0 };
const char *mountpoint = "/mnt/usb0";
const char *device = "/dev/sda1";

/* 挂载卷 */
vol = ntfs_mount(device, NTFS_MNT_RDWR);
if (vol == NULL) {
    /* 处理错误 */
    perror("ntfs_mount");
    return -1;
}

/* 执行文件系统操作... */

/* 卸载卷 */
ntfs_umount(vol);
```

**模式二：打开和读取文件**

```c
#include <ntfs-3g/ntfs.h>

ntfs_inode *inode;
char *buf;
size_t buf_size = 4096;

inode = ntfs_inode_open(vol, 5);  /* 打开 MFT 记录号为 5 的文件 */
if (inode == NULL) {
    perror("ntfs_inode_open");
    return -1;
}

buf = malloc(buf_size);
if (buf == NULL) {
    ntfs_inode_close(inode);
    return -1;
}

/* 读取文件数据 */
s64 bytes_read = ntfs_pread(inode, 0, buf_size, buf);
if (bytes_read < 0) {
    perror("ntfs_pread");
    free(buf);
    ntfs_inode_close(inode);
    return -1;
}

/* 处理读取的数据... */

free(buf);
ntfs_inode_close(inode);
```

**模式三：创建目录**

```c
#include <ntfs-3g/ntfs.h>

ntfs_inode *parent;
ntfs_inode *new_dir;
char *path = "/mnt/usb0/new_directory";

/* 打开父目录 */
parent = ntfs_pathname_to_inode(vol, NULL, "/mnt/usb0");
if (parent == NULL) {
    perror("ntfs_pathname_to_inode");
    return -1;
}

/* 创建新目录 */
new_dir = ntfs_create(parent, "new_directory", S_IFDIR | 0755, NULL, NULL);
if (new_dir == NULL) {
    perror("ntfs_create");
    ntfs_inode_close(parent);
    return -1;
}

ntfs_inode_close(new_dir);
ntfs_inode_close(parent);
```

## 五、已知限制

### 5.1 功能限制

**加密文件不可访问**：由于 ENABLE_CRYPTO 被禁用，无法打开或读取 EFS 加密的文件。尝试操作加密文件将返回错误。

**插件不可用**：DISABLE_PLUGINS 意味着无法加载自定义插件来扩展功能。

**调试信息受限**：ENABLE_DEBUG 被禁用，无法获取详细的调试日志。

### 5.2 平台限制

**Windows 特定功能**：某些 Windows 特有的功能（如 NTFS 交换数据流 ADS 的某些高级用法）可能不受支持。

**64 位偏移量限制**：虽然支持大文件，但在某些嵌入式平台上可能存在文件系统大小的物理限制。

### 5.3 性能限制

用户空间文件系统实现相比内核实现存在固有的性能开销，特别是对于大量小文件的随机访问场景。建议在性能敏感的场景中进行充分测试。

## 六、版本兼容性

### 6.1 API 稳定性

NTFS-3G 的核心 API 自 2017 年版本以来保持相对稳定。OpenHarmony 版本基于上游 2022.10.3，具有良好的 API 成熟度。

### 6.2 升级考虑

当升级 NTFS-3G 到上游新版本时：

**API 变更检查**：阅读上游 CHANGELOG，确认 API 是否有重大变更。

**头文件兼容性**：比较新旧版本的头文件，确保 API 签名一致。

**行为变化**：某些 Bug Fix 可能改变原有行为，需要进行兼容性测试。

---

*本文档最后更新于 2026 年 2 月 7 日*
