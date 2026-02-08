# CUPS OpenHarmony Wiki

> CUPS (Common UNIX Printing System) v2.4.14 在 OpenHarmony 中的集成文档

## 文档导航

### 核心文档

| 文档 | 描述 | 必读 |
|-----|------|-----|
| [README](README.md) | 库概览、OH 适配概述、文档导航 | ✅ |
| [01_Overview](01_Overview.md) | 原始库简介、OH 作用和定位 | ✅ |
| [02_Patches](02_Patches.md) | **核心文档** - 所有 Patch 详细分析 | ✅✅ |
| [03_Build_Integration](03_Build_Integration.md) | BUILD.gn 构建适配详解 | ⭐ |
| [04_Usage_in_OH](04_Usage_in_OH.md) | 依赖关系、使用场景、架构图 | ⭐ |

### 安全文档

| 文档 | 描述 |
|-----|------|
| [06_Security](06_Security.md) | CVE 修复记录、安全风险分析 |

### 工作文档

| 文档 | 描述 |
|-----|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 |

---

## 快速摘要

### CUPS 在 OpenHarmony 中的作用

CUPS 是 OpenHarmony **打印子系统**的核心组件，提供：

- **打印任务管理**：作业队列、状态监控、取消操作
- **打印机发现**：IPP、USB、网络打印机自动发现
- **PPD 驱动支持**：PostScript Printer Description 驱动解析
- **打印过滤器**：图像到 PDF/光栅格式转换
- **IPP 协议支持**：Internet Printing Protocol 实现

### OH 特有适配

| 适配领域 | 主要 Patch | 功能 |
|---------|-----------|------|
| **USB 打印** | `ohos-usb-print.patch`, `ohos-usb-manager.patch` | OH USB 服务集成 |
| **日志系统** | `ohos-hilog-print.patch`, `cups-log-datamasking.patch` | HiLog 集成、数据脱敏 |
| **网络安全** | `ohos_ip_conflict.patch`, `ohos-ipp-authenticate.patch` | IP 冲突处理、认证适配 |
| **系统安全** | `cups-log-datamasking.patch` | 日志隐私保护 |

### 依赖关系

```
print_service (打印服务)
    │
    ├──► cups (CUPS 核心库) ← 16 个安全补丁
    │         │
    │         ├──► 基础库: openssl, zlib, libusb
    │         └──► OH 服务: hilog, usb_manager, ipc
    │
    └──► cups-filters (过滤器库)
              ├──► libjpeg-turbo, libpng
              └──► 图像处理: PDF/光栅转换
```

---

## Patch 统计

| 类别 | 数量 | 说明 |
|-----|------|------|
| **OH 特有适配** | 22 个 | USB、日志、网络、安全 |
| **CUPS 功能增强** | 7 个 | 作业监控、超时处理、认证 |
| **CVE 安全修复** | 9 个 | 回溯到 v2.4.14 的安全补丁 |

---

## 版本信息

| 属性 | 值 |
|------|-----|
| **上游版本** | v2.4.14 |
| **OH 版本** | 4.0 |
| **许可证** | Apache License 2.0 |
| **所属子系统** | thirdparty |
| **维护者** | baozewei@huawei.com |

---

## 开始阅读

建议阅读顺序：

1. **[01_Overview.md](01_Overview.md)** - 了解 CUPS 在 OH 中的定位
2. **[02_Patches.md](02_Patches.md)** - 深入理解 OH 适配细节
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 查看依赖架构
4. **[03_Build_Integration.md](03_Build_Integration.md)** - 了解构建配置
