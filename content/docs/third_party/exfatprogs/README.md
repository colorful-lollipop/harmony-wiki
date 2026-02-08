# exfatprogs Wiki

## 库概述

exfatprogs 是 OpenHarmony third_party 子系统中的 exFAT 文件系统用户空间工具库，为系统提供完整的 exFAT 文件系统创建、检查、修复和管理能力。

| 属性 | 值 |
|------|-----|
| **上游版本** | 1.2.5 |
| **OH 版本号** | 4.1 |
| **许可证** | GPL-2.0-or-later |
| **上游地址** | https://github.com/exfatprogs/exfatprogs |
| **OH 子系统** | thirdparty |
| **Patch 状态** | 无 Patch（干净导入） |

## OpenHarmony 适配概述

exfatprogs 在 OpenHarmony 中的适配具有以下特点：

### 适配策略：最小化修改

- **无 Patch 文件**：该库为干净导入，未应用任何 OH 特有的代码修改
- **构建系统重构**：将上游的 autotools 构建系统适配为 OH 的 GN 构建系统
- **接口保持一致**：对外提供的 API 和工具与上游完全兼容

### 核心能力

1. **libexfat 动态库**：提供 exFAT 文件系统操作的核心 API，被系统其他模块动态链接使用
2. **命令行工具集**：
   - `mkfs.exfat`：创建 exFAT 文件系统
   - `fsck.exfat`：检查和修复 exFAT 文件系统
   - `dump.exfat`：导出 exFAT 文件系统信息
   - `exfatlabel`：管理卷标
   - `tune.exfat`：调整文件系统参数
   - `exfat2img`：导出文件系统镜像

### 技术特性

- 支持大于 4GB 的大文件（`_FILE_OFFSET_BITS=64`）
- 符合 Windows 端 exFAT 工具的性能和质量标准
- 完整的 UTF-8 卷标支持
- 可配置的簇大小和边界对齐

## 文档导航

| 文档 | 说明 | 目标读者 |
|------|------|---------|
| [SUMMARY](SUMMARY.md) | 阅读路线建议 | 所有使用者 |
| [01_Overview](01_Overview.md) | 原始库简介和 OH 定位 | 快速了解背景 |
| [02_Patches](02_Patches.md) | Patch 详细分析 | 维护者和开发者 |
| [03_Build_Integration](03_Build_Integration.md) | OH 构建适配说明 | 构建系统维护者 |
| [04_Usage_in_OH](04_Usage_in_OH.md) | 依赖关系和使用场景 | 系统集成者 |

## 快速开始

### 在 OH 构建中使用 libexfat

```gn
deps += [ "//third_party/exfatprogs:libexfat" ]
```

### 调用 exFAT 文件系统 API

```c
#include <libexfat.h>

int main() {
    struct exfat *ef;
    if (exfat_mount(&ef, "/dev/sda1", "rw") != 0) {
        return -1;
    }
    // 执行文件系统操作
    exfat_unmount(ef);
    return 0;
}
```

## 相关资源

- **上游文档**：https://github.com/exfatprogs/exfatprogs
- **OH bundle.json**：在源码目录的 `bundle.json` 中定义组件元数据
- **构建配置**：完整的 GN 构建配置见源码目录 `BUILD.gn`

## 维护信息

| 项目 | 值 |
|------|-----|
| **最后分析** | 2026-02-07 |
| **分析者** | OpenHarmony Wiki Agent |
| **评估状态** | 已完成 |
