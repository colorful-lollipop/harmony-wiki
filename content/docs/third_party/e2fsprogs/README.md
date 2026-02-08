# e2fsprogs OpenHarmony 适配文档

## 概述

本文档是 e2fsprogs 在 OpenHarmony 中集成与适配的详细说明。e2fsprogs 是 ext2/ext3/ext4 文件系统管理的核心工具集，在 OpenHarmony 中承担着系统镜像制作、文件系统维护等关键功能。

## 文档导航

| 文档 | 内容 | 阅读建议 |
|------|------|---------|
| [01_Overview.md](./01_Overview.md) | 库简介、OH 定位 | 首先阅读 |
| [02_Patches.md](./02_Patches.md) | **核心文档**：Patch 详细分析 | **重点阅读** |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建适配 | 开发者必读 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | OH 中的依赖与使用 | 架构师参考 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异说明 | 按需阅读 |
| [06_Security.md](./06_Security.md) | 安全风险分析 | 安全工程师参考 |

## 快速入门

### 库基本信息

| 项目 | 内容 |
|------|------|
| **上游版本** | 1.47.2 |
| **OH 版本** | 4.1 |
| **许可证** | LGPL/BSD/GPL/MIT 多重许可 |
| **上游地址** | https://git.kernel.org/pub/scm/fs/ext2/e2fsprogs.git |

### OpenHarmony 中的主要功能

1. **镜像制作** (`e2fsdroid`): 创建 system.img, vendor.img 等分区镜像
2. **文件系统检查** (`e2fsck`): 系统启动时检查和修复文件系统
3. **分区调整** (`resize2fs`): OTA 升级时动态调整分区大小
4. **块设备识别** (`blkid`, `libblkid`): 识别存储设备分区类型

### Patch 数量

共有 **8 个 Patch**，分为三类：
- 🛠️ **编译适配**: 2 个 (musl 适配、编译修复)
- ✨ **功能增强**: 5 个 (DAC 配置、HMFS 支持、中文卷标等)
- 🐛 **Bugfix**: 1 个 (NTFS 大簇支持)

## 关键适配点

### 1. DAC 配置机制 (Patch 1003)

OpenHarmony 实现了自定义的 DAC (Discretionary Access Control) 配置机制，替代了 Android 的 fs_config。

```cpp
// contrib/android/dac_config.cpp
int LoadDacConfig(const char* fn);
void GetDacConfig(const char* path, int dir, char* targetOutPath,
        unsigned* uid, unsigned* gid, unsigned* mode,
        uint64_t* capabilities);
```

**特点**:
- CSV 格式配置文件
- 支持 Linux capabilities
- 支持通配符匹配

### 2. HMFS 文件系统支持 (Patch 1006)

添加了对华为移动文件系统 (HMFS) 的识别支持：

```c
// lib/blkid/probe.c
{ "hmfs",    1,      0,  4, "\x24\x20\xf5\xfe", probe_hmfs },
```

### 3. 中文卷标支持 (Patch 1005)

VFAT 文件系统的中文卷标自动转码 (GBK → UTF-8)。

## 使用示例

### 使用 e2fsdroid 创建镜像

```bash
# 创建 ext4 镜像
e2fsdroid -T -1 -C fs_config.txt -S file_contexts \
    -s shared_library -a system -f system/ \
    -L system -D system -b 4096 system.img
```

### 使用 blkid 识别分区

```bash
# 识别所有块设备
blkid

# 跳过指定文件系统类型
blkid -n mdraid
```

## 维护信息

- **维护者**: zhangdaiyue1@huawei.com
- **子系统**: thirdparty
- **最后更新**: 2026-02-08

## 相关链接

- [上游文档](http://e2fsprogs.sourceforge.net/)
- [ext4 维基](https://ext4.wiki.kernel.org/)
- [OpenHarmony 存储子系统文档](https://gitee.com/openharmony/docs)

---

**注意**: 本文档仅关注 OpenHarmony 对 e2fsprogs 的定制化内容。关于原始库的使用方法，请参考上游文档。
