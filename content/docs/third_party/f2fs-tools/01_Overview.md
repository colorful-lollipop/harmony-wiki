# 01 - 原始库简介与 OH 定位

## 1.1 原始库信息

### 基本信息

| 属性 | 值 |
|-----|-----|
| **库名称** | f2fs-tools |
| **版本** | v1.16.0 |
| **发布日期** | 2023-04-11 |
| **许可证** | GPL-2.0 |
| **原始作者** | Jaegeuk Kim <jaegeuk@kernel.org> |
| **上游仓库** | https://git.kernel.org/pub/scm/linux/kernel/git/jaegeuk/f2fs-tools.git |
| **邮件列表** | linux-f2fs-devel@lists.sourceforge.net |

### 原始功能

f2fs-tools 是 **F2FS（Flash-Friendly File System）** 的用户空间工具集，专为闪存存储设备（如 eMMC、SSD、SD 卡）优化设计。

#### 包含的工具

| 工具 | 功能描述 |
|-----|---------|
| `mkfs.f2fs` | 创建 F2FS 文件系统，格式化存储设备 |
| `fsck.f2fs` | 检查和修复 F2FS 文件系统错误 |
| `dump.f2fs` | 转储 F2FS 文件系统元数据（fsck 的符号链接） |
| `defrag.f2fs` | F2FS 文件系统碎片整理（fsck 的符号链接） |
| `resize.f2fs` | 调整 F2FS 文件系统大小（fsck 的符号链接） |
| `sload.f2fs` | 将文件加载到 F2FS 镜像（fsck 的符号链接） |
| `f2fscrypt` | F2FS 文件系统加密管理 |
| `f2fstat` | 显示 F2FS 文件系统统计信息 |
| `fibmap.f2fs` | 显示文件的块映射信息 |
| `libf2fs.so` | F2FS 操作共享库 |

### 上游依赖

原始 f2fs-tools 依赖以下库：
- **libuuid**: UUID 生成（通常来自 e2fsprogs）
- **libselinux**: SELinux 支持（可选）
- **libblkid**: 块设备识别（可选）

---

## 1.2 F2FS 文件系统简介

### 设计目标

F2FS 是由三星开发的日志结构文件系统（Log-structured File System），针对闪存设备的特性进行了优化：

- **磨损均衡（Wear Leveling）**: 均匀分布写入操作，延长闪存寿命
- **断电保护**: 日志结构确保意外断电后的数据一致性
- **Trim/Discard 支持**: 优化 SSD 的垃圾回收
- **多流写入**: 分离热数据和冷数据，减少写放大

### 关键数据结构

```
F2FS 磁盘布局：
┌─────────────────────────────────────────┐
│           Superblock (SB)               │
├─────────────────────────────────────────┤
│     Checkpoint Pack (CP) - 主           │
├─────────────────────────────────────────┤
│     Checkpoint Pack (CP) - 备           │
├─────────────────────────────────────────┤
│         Segment Info Table (SIT)        │
├─────────────────────────────────────────┤
│         Node Address Table (NAT)        │
├─────────────────────────────────────────┤
│         Segment Summary Area (SSA)      │
├─────────────────────────────────────────┤
│              Main Area                  │
│    ┌───────────────────────────────┐    │
│    │  Node Blocks (inode, dentry)  │    │
│    ├───────────────────────────────┤    │
│    │       Data Blocks             │    │
│    └───────────────────────────────┘    │
└─────────────────────────────────────────┘
```

---

## 1.3 OpenHarmony 集成情况

### OH 组件信息

| 属性 | 值 |
|-----|-----|
| **OH 组件名** | @ohos/f2fs-tools |
| **OH 版本** | 3.1 |
| **所属子系统** | thirdparty |
| **发布形式** | code-segment |
| **目标路径** | third_party/f2fs-tools |
| **适配系统** | standard |

### 依赖组件

```json
{
  "deps": {
    "components": [
      "bounds_checking_function",
      "e2fsprogs"
    ],
    "third_party": []
  }
}
```

### 导出的 Inner Kits

| Kit 名称 | 类型 | 说明 |
|---------|------|------|
| `//third_party/f2fs-tools/lib:libf2fs` | 共享库 | 提供 `utf8data.h` 头文件 |
| `//third_party/f2fs-tools/fsck:fsck.f2fs` | 可执行文件 | 文件系统检查修复工具 |
| `//third_party/f2fs-tools/mkfs:mkfs.f2fs` | 可执行文件 | 格式化工具 |

---

## 1.4 f2fs-tools 在 OpenHarmony 中的作用

### 核心定位

f2fs-tools 是 OpenHarmony 系统**存储管理的基础设施**，承担以下关键职责：

#### 1. 系统启动保障
- **init 服务**在启动时调用 `fsck.f2fs` 检查 `/data` 分区
- 确保文件系统一致性，防止损坏导致系统无法启动

#### 2. 数据分区管理
- **storage_daemon** 负责 `/data` 分区的日常管理
- 支持格式化、检查、调整大小等操作
- 为应用提供可靠的存储服务

#### 3. OTA 升级支持
- 系统升级过程中可能需要重新格式化数据分区
- 构建系统使用主机工具链版本生成系统镜像

### 在 OH 架构中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                        应用层                                │
├─────────────────────────────────────────────────────────────┤
│                   Storage Manager                           │
│              (foundation/filemanagement/storage_service)   │
├─────────────────────────────────────────────────────────────┤
│                   Storage Daemon                            │
│              (storage_service/services/storage_daemon)     │
│                     ↓ 依赖 f2fs-tools                       │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  mkfs.f2fs   │  │  fsck.f2fs   │  │  libf2fs.so  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
├─────────────────────────────────────────────────────────────┤
│                   Kernel (F2FS 驱动)                        │
├─────────────────────────────────────────────────────────────┤
│                   Block Device (/dev/block)                 │
└─────────────────────────────────────────────────────────────┘
```

### 为什么 OH 选择 F2FS

| 优势 | 说明 |
|-----|------|
| **闪存优化** | 针对移动设备常用的 eMMC/UFS 优化 |
| **性能** | 随机写入性能优于 ext4，适合 SQLite 等数据库操作 |
| **可靠性** | 日志结构提供更好的断电保护 |
| **Android 验证** | 已被 Android 广泛采用，经过大规模验证 |
| **维护活跃** | 内核主线支持，持续更新 |

---

## 1.5 OH 版本与上游的差异概览

### 无 Patch 文件的集成方式

与许多 third_party 库不同，f2fs-tools **没有独立的 .patch 文件**。所有修改直接在源代码中维护，通过 `WITH_OHOS` 宏进行条件编译。

### 主要差异类别

| 类别 | 差异内容 |
|-----|---------|
| **构建系统** | 使用 GN/Ninja 替代 autotools |
| **新增源文件** | libf2fs_log.c, libf2fs_dmd.c, extra_fsck.c |
| **新增头文件** | f2fs_log.h, f2fs_dmd.h, f2fs_dmd_cfg.h 等 |
| **条件编译** | WITH_OHOS 宏控制 OH 特有代码 |
| **日志增强** | 增加 kmsg 和文件日志系统 |
| **DFX 诊断** | 增加错误统计和性能监控 |
| **安全加固** | 使用 bounds_checking_function 安全函数 |

### 版本一致性

当前 OH 集成的版本基于 **v1.16.0**，但需要注意：
- Git 历史显示后续合入了部分上游 bugfix
- OH 特有代码与上游代码通过宏隔离
- 建议定期同步上游安全修复
