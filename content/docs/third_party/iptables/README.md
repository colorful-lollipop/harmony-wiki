# iptables Wiki

## 库概览

iptables 是 Linux 系统上用于配置 IPv4/IPv6 数据包过滤和 NAT 的用户空间工具。在 OpenHarmony 中，iptables 作为网络基础设施组件，为系统提供防火墙、网络地址转换和流量控制功能。

| 属性 | 详情 |
|-----|------|
| **上游版本** | 1.8.11 (README.OpenSource) / 1.8.7 (bundle.json) |
| **许可证** | GPL V2.0 |
| **上游地址** | https://netfilter.org/ |
| **OH 组件名** | @ohos/iptables |
| **OH 子系统** | thirdparty → communication (netmanager_base) |

## OH 适配概述

iptables 在 OpenHarmony 中采用**轻度适配**策略：

- ✅ **源码改动最小**: 仅 1 个 Patch（musl 兼容性修复）
- ✅ **完整 GN 构建**: 4 个 BUILD.gn 文件覆盖完整构建流程
- ✅ **静态链接**: 所有扩展静态编译，无运行时依赖
- ✅ **功能完整**: 保留所有上游功能，无 API 变更

## 核心 Patch

| Patch | 目的 | 文件 |
|-------|------|------|
| 0001-musl-build-fix.patch | musl libc 兼容性 | 修改 3 个扩展的头文件包含路径 |

## 主要使用模块

| 模块 | 子系统 | 用途 |
|-----|--------|------|
| netmanager_base | communication | 核心网络管理、防火墙、流量控制 |
| enterprise_device_management | customization | 企业防火墙策略管理 |

## 文档导航

### 📚 完整文档

| 文档 | 内容 | 推荐度 |
|-----|------|-------|
| [01_Overview.md](./01_Overview.md) | 库简介、OH 定位、功能概述 | ⭐⭐⭐⭐⭐ |
| [02_Patches.md](./02_Patches.md) | **核心文档**: Patch 详细分析 | ⭐⭐⭐⭐⭐ |
| [03_Build_Integration.md](./03_Build_Integration.md) | GN 构建配置详解 | ⭐⭐⭐⭐ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | ⭐⭐⭐⭐⭐ |
| [05_API_Differences.md](./05_API_Differences.md) | API 兼容性与差异 | ⭐⭐⭐ |
| [06_Security.md](./06_Security.md) | 安全风险分析 | ⭐⭐⭐⭐ |

### 🚀 快速开始

**了解 iptables 在 OH 中的作用**: 阅读 [01_Overview.md](./01_Overview.md)

**查看 Patch 详情**: 阅读 [02_Patches.md](./02_Patches.md)

**理解依赖关系**: 阅读 [04_Usage_in_OH.md](./04_Usage_in_OH.md)

**升级前必读**: 阅读 [02_Patches.md#升级建议](./02_Patches.md#升级建议) 和 [06_Security.md](./06_Security.md)

## 关键信息速查

### 构建命令

```bash
# 构建 iptables
./build.sh --product {product_name} --build-target //third_party/iptables:iptables

# 构建所有目标
./build.sh --product {product_name} --build-target //third_party/iptables:all
```

### 生成的可执行文件

| 文件 | 路径 | 说明 |
|-----|------|------|
| iptables | /system/bin/iptables | IPv4 防火墙配置 |
| ip6tables | /system/bin/ip6tables | IPv6 防火墙配置 (软链接) |
| iptables-save | /system/bin/iptables-save | 规则保存 |
| iptables-restore | /system/bin/iptables-restore | 规则恢复 |

### 核心静态库

| 库名 | 说明 |
|-----|------|
| libip4tc | IPv4 表控制库 |
| libip6tc | IPv6 表控制库 |
| libxtables | xtables 共享库 |
| libext | 通用扩展模块 |
| libext4 | IPv4 扩展模块 |
| libext6 | IPv6 扩展模块 |

## 版本历史

| 日期 | 版本 | 变更 |
|-----|------|------|
| 2024-11-25 | 1.8.11 | 添加 musl 兼容性 Patch |
| - | 1.8.7 | OH bundle.json 记录版本 |

## 维护者

- **Owner**: heqianmo@huawei.com
- **Patch 作者**: Violet Purcell (musl-build-fix.patch)

## 许可证

本项目采用 GPL V2.0 许可证。详见 [COPYING](../COPYING) 文件。

---

*本文档是 OpenHarmony third_party/iptables 库的 Wiki，重点说明该库在 OH 中的集成与适配。*
