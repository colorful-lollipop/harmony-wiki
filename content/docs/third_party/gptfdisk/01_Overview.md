# 01 库概览与 OH 定位

## 1.1 原始库信息

### 基本信息

| 属性 | 值 |
|------|-----|
| **名称** | GPT fdisk |
| **上游版本** | 1.0.10 |
| **发布日期** | 2024-02-19 |
| **作者** | Roderick W. Smith |
| **许可证** | GPL-2.0-only |
| **上游地址** | https://sourceforge.net/projects/gptfdisk |

### 功能描述

GPT fdisk 是一套用于操作 GUID 分区表 (GPT) 磁盘的工具集， loosely modeled on Linux fdisk。主要特点：

- **GPT 支持**: 创建、修改、删除 GPT 分区
- **无损转换**: MBR ↔ GPT 双向转换不丢失数据
- **混合 MBR**: 创建 Hybrid MBR 兼容旧系统
- **脚本友好**: sgdisk 适合自动化脚本

### 上游组件

| 组件 | 类型 | 功能 | OH 包含 |
|------|------|------|---------|
| gdisk | 可执行文件 | 交互式 TUI 分区工具 | ❌ |
| cgdisk | 可执行文件 | curses GUI 分区工具 | ❌ |
| sgdisk | 可执行文件 | 命令行脚本化工具 | ✅ |
| fixparts | 可执行文件 | MBR 修复工具 | ❌ |
| libgpt | 库 | 核心 GPT 操作库 | ✅ (静态链接) |

### 上游依赖

| 依赖 | 用途 | OH 替代 |
|------|------|---------|
| libuuid | GPT GUID 生成 | e2fsprogs:libext2_uuid |
| popt | 命令行解析 | popt:popt_static |
| ncurses | cgdisk UI | 不使用 |

---

## 1.2 OpenHarmony 中的定位

### 系统定位

在 OpenHarmony 中，gptfdisk 是**存储子系统的基础工具组件**，提供底层磁盘分区能力。

```
┌─────────────────────────────────────────────────────┐
│                 应用层 (Applications)                │
├─────────────────────────────────────────────────────┤
│              文件管理 (File Management)              │
├─────────────────────────────────────────────────────┤
│           Storage Service (storage_daemon)           │
│  ┌───────────────────────────────────────────────┐  │
│  │  VolumeManager / DiskInfo                      │  │
│  │       ↓ (调用 sgdisk)                          │  │
│  └───────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│           sgdisk (gptfdisk)                        │
│  ┌───────────────────────────────────────────────┐  │
│  │  GPT 分区表操作 / MBR 分区表操作                │  │
│  └───────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│              块设备层 (Block Device)                 │
└─────────────────────────────────────────────────────┘
```

### 所属子系统

| 属性 | 值 |
|------|-----|
| **子系统** | thirdparty |
| **组件名** | gptfdisk |
| **Bundle** | @ohos/gptfdisk |
| **系统类型** | standard (标准系统) |

### 核心作用

**磁盘生命周期管理**:

1. **磁盘初始化**: 新设备首次启动时创建分区
2. **分区重置**: 恢复出厂设置时清除并重建分区
3. **分区查询**: 获取当前分区表信息用于显示或决策

### 典型使用场景

| 场景 | 操作 | sgdisk 命令 |
|------|------|-------------|
| 设备首次启动 | 创建 userdata 分区 | `--new=0:0:-0 --typecode=0:0c00` |
| 恢复出厂设置 | 清除分区表 | `--zap-all` |
| 分区信息展示 | 导出分区详情 | `--ohos-dump` |
| MBR 兼容 | GPT 转 MBR | `--gpttombr=1` |

---

## 1.3 OH 与上游差异对比

### 功能裁剪

| 功能 | 上游 | OH | 原因 |
|------|------|-----|------|
| gdisk | ✅ | ❌ | 嵌入式不需要交互式 TUI |
| cgdisk | ✅ | ❌ | 嵌入式不需要图形界面 |
| sgdisk | ✅ | ✅ | 脚本化操作，必需 |
| fixparts | ✅ | ❌ | OH 使用 GPT，不需要 MBR 修复 |
| ICU 支持 | 可选 | ❌ | 不需要 Unicode 分区名 |

### 构建差异

| 对比项 | 上游 | OH |
|-------|------|-----|
| **构建系统** | Makefile | GN (BUILD.gn) |
| **构建目标** | 本地编译 | 交叉编译 |
| **依赖获取** | 系统包管理器 | GN 依赖声明 |
| **安装路径** | /usr/local/sbin | /system/bin |
| **编译器标志** | 默认 | 添加 OH 特定警告抑制 |

### 代码差异

| 文件 | 差异类型 | 说明 |
|------|---------|------|
| sgdisk.cc | **功能扩展** | 新增 `ohos_dump()` 和 `--ohos-dump` |
| BUILD.gn | **新增** | OH 构建配置 |
| bundle.json | **新增** | OH 组件元数据 |

---

## 1.4 技术规格

### 分区表支持

| 类型 | 支持 | 说明 |
|------|------|------|
| **GPT** | ✅ 完整支持 | 主要使用格式 |
| **MBR** | ✅ 读取/转换 | 兼容旧设备 |
| **Hybrid MBR** | ✅ 支持 | GPT + 保护性 MBR |
| **BSD disklabel** | ✅ 转换支持 | 可转换为 GPT |

### 分区类型代码 (部分)

| GUID | 类型 | OH 使用 |
|------|------|---------|
| 0x0c01 | Microsoft reserved | 可能 |
| 0x0c00 | Microsoft basic data | ✅ userdata |
| 0x8300 | Linux filesystem | 可能 |
| 0xef00 | EFI System | 可能 |

完整列表见: `parttypes.cc`

### 性能特征

| 指标 | 值 | 说明 |
|------|-----|------|
| **二进制大小** | ~200KB | 仅 sgdisk |
| **内存占用** | 低 | 命令行工具，无常驻内存 |
| **执行时间** | <100ms | 分区表操作 |
| **依赖库大小** | ~100KB | popt + uuid |

---

## 1.5 相关资源

### 上游文档

- [项目主页](https://sourceforge.net/projects/gptfdisk)
- [作者书籍](http://www.rodsbooks.com/gdisk/)
- [Man Pages](https://www.rodsbooks.com/gdisk/sgdisk.html)

### OH 相关组件

- [storage_service](../../foundation/filemanagement/storage_service) - 主要使用者
- [e2fsprogs](../e2fsprogs) - UUID 库
- [popt](../popt) - 命令行解析

### 规范参考

- [UEFI Specification](https://uefi.org/specifications) - GPT 规范
- [Microsoft GPT FAQ](https://docs.microsoft.com/en-us/windows-server/storage/disk-management/gpt-faq)

---

## 1.6 版本历史

| OH 版本 | 上游版本 | 变更 |
|---------|---------|------|
| 3.1 | 1.0.10 | 初始集成 |

### 上游版本历史 (节选)

| 版本 | 日期 | 关键变更 |
|------|------|---------|
| 1.0.10 | 2024-02-19 | popt 兼容性修复，新增分区类型代码 |
| 1.0.9 | 2022-04-14 | 分区对齐支持，Windows 构建支持 |
| 1.0.8 | 2021-06-09 | 大端系统修复 |
| 1.0.7 | 2021-03-10 | Bug 修复 |

完整历史见: `NEWS` 文件
