# GPT fdisk Wiki

> OpenHarmony 第三方库文档 - GPT fdisk  
> 最后更新: 2026-02-07

---

## 库概览

| 属性 | 值 |
|------|-----|
| **库名称** | GPT fdisk |
| **上游版本** | 1.0.10 |
| **OH 组件版本** | 3.1 |
| **许可证** | GPL-2.0-only |
| **上游地址** | https://sourceforge.net/projects/gptfdisk |
| **OH 路径** | `third_party/gptfdisk` |
| **所属子系统** | thirdparty |
| **维护者** | gudehe@huawei.com |

---

## 原始库简介

GPT fdisk 是一套磁盘分区工具，主要用于操作 GUID 分区表 (GPT) 磁盘。包含四个相关程序：

| 程序 | 描述 | OH 中是否编译 |
|------|------|---------------|
| **gdisk** | 交互式文本模式 GPT 分区工具 | ❌ 否 |
| **cgdisk** | curses 图形界面 GPT 分区工具 | ❌ 否 |
| **sgdisk** | 命令行驱动的脚本化工具 | ✅ 是 |
| **fixparts** | MBR 分区修复工具 | ❌ 否 |

### 核心功能

- GPT 磁盘分区创建、删除、修改
- MBR 到 GPT 无损转换
- 创建混合 MBR (Hybrid MBR)
- BSD disklabel 转换

---

## OH 适配概述

### 适配方式

OpenHarmony 对 gptfdisk 的适配采用**源码级修改**方式，而非传统 Patch 文件。

### 关键适配点

| 适配项 | 说明 |
|-------|------|
| **构建系统** | 从 Makefile 迁移到 GN (BUILD.gn) |
| **功能裁剪** | 仅编译 sgdisk，移除交互式工具 |
| **功能扩展** | 新增 `--ohos-dump` 选项 |
| **依赖管理** | 通过 GN 依赖 e2fsprogs 和 popt |

### OH 特有功能

**`--ohos-dump` 选项**

机器可读格式导出分区表信息，专为 Storage Service 设计：

```
DISK [mbr|gpt] [guid]
PART [n] [type] [guid] [description]
```

---

## 文档导航

### 快速开始

- [01_Overview.md](./01_Overview.md) - 原始库简介与 OH 定位
- [02_Patches.md](./02_Patches.md) - OH 特有修改分析 (**核心文档**)
- [03_Build_Integration.md](./03_Build_Integration.md) - GN 构建配置详解
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用场景
- [05_API_Differences.md](./05_API_Differences.md) - 接口差异说明
- [06_Security.md](./06_Security.md) - 安全风险分析

### 工程文档

- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估结果
- [_work/NOTES.md](./_work/NOTES.md) - 分析过程记录
- [_work/PLAN.md](./_work/PLAN.md) - 任务进度

---

## 关键信息摘要

### 依赖关系

```
storage_service (storage_daemon)
    ↓
sgdisk (gptfdisk)
    ↓
    +-- e2fsprogs:libext2_uuid
    +-- popt:popt_static
```

### 安装位置

| 文件 | 路径 |
|------|------|
| sgdisk | `/system/bin/sgdisk` |

### 使用场景

- **磁盘初始化**: 新设备首次分区
- **分区重置**: 恢复出厂设置时清除分区
- **分区信息导出**: 读取磁盘 GPT/MBR 信息

---

## 维护者须知

### 升级上游版本时

1. **保留修改**: 确保 `sgdisk.cc` 中的 `ohos_dump()` 函数和 `--ohos-dump` 处理逻辑
2. **依赖检查**: 确认 e2fsprogs 和 popt 版本兼容性
3. **功能测试**: 验证 storage_service 磁盘操作功能

### 常见问题

| 问题 | 解决方案 |
|------|---------|
| sgdisk 命令未找到 | 确认 `install_enable = true` 且镜像包含 system 分区 |
| 分区操作失败 | 检查设备节点权限和 SELinux 策略 |
| --ohos-dump 输出异常 | 确认磁盘未损坏且分区表格式正确 |

---

## 参考链接

- [上游项目主页](https://sourceforge.net/projects/gptfdisk)
- [作者文档](http://www.rodsbooks.com/gdisk/)
- [GPT 规范](https://uefi.org/specifications)
- [OpenHarmony 存储子系统](../foundation/filemanagement/storage_service)
