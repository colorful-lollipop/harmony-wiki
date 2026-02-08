# ncurses OpenHarmony Wiki

## 库概览

**ncurses** (new curses) 是一个提供字符用户界面 (TUI) 功能的终端控制库，在 OpenHarmony 中作为系统基础库使用。

| 属性 | 内容 |
|------|------|
| **版本** | 6.5 (2024年4月27日发布) |
| **许可证** | MIT |
| **上游地址** | https://invisible-mirror.net/archives/ncurses/ncurses-6.5.tar.gz |
| **OH 维护者** | liyiming13@huawei.com |

## OpenHarmony 适配概述

ncurses 在 OpenHarmony 中应用了 **6 个 Patch**，主要涵盖：

### Patch 分类

| 类别 | Patch | 说明 |
|------|-------|------|
| **终端类型增强** | ncurses-kbs.patch, ncurses-urxvt.patch | 修复退格键，添加 urxvt 支持 |
| **构建系统适配** | ncurses-libs.patch, ncurses-config.patch | 调整库链接，简化配置脚本 |
| **OH 平台适配** | cross_compile_support_ohos.patch | 添加 OpenHarmony 交叉编译支持 |
| **安全修复** | backport-0002-CVE-2023-29491-env-access.patch | 修复 CVE-2023-29491 漏洞 |

### 关键特性

- ✅ **无 BUILD.gn**: 使用传统 autotools 构建系统
- ✅ **API 100% 兼容**: 与上游 ncurses 6.5 完全兼容
- ✅ **安全修复**: 包含 CVE-2023-29491 关键安全修复
- ⚠️ **依赖待确认**: 未在 BUILD.gn 中发现直接依赖者

## 文档导航

### 快速开始

| 文档 | 内容 | 适合读者 |
|------|------|----------|
| [01_Overview.md](01_Overview.md) | 库简介、在 OH 中的作用 | 所有读者 |
| [02_Patches.md](02_Patches.md) | **核心文档** - Patch 详细分析 | 开发者、维护者 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建系统适配说明 | 构建工程师 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系与使用场景 | 开发者 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异分析 | 应用开发者 |
| [06_Security.md](06_Security.md) | 安全风险分析 | 安全工程师 |

### 阅读路线建议

#### 路线 1: 快速了解 (5分钟)
1. 阅读本文档 (README.md)
2. 浏览 [01_Overview.md](01_Overview.md) 的前半部分
3. 查看 [02_Patches.md](02_Patches.md) 的 Patch 清单表

#### 路线 2: Patch 分析 (20分钟)
1. [01_Overview.md](01_Overview.md) - 了解背景
2. [02_Patches.md](02_Patches.md) - 详细 Patch 分析
3. [06_Security.md](06_Security.md) - 安全修复详情

#### 路线 3: 深度理解 (40分钟)
1. 完整阅读所有文档
2. 重点关注 [02_Patches.md](02_Patches.md) 和 [03_Build_Integration.md](03_Build_Integration.md)
3. 查看源码 Patch 文件进行对照

#### 路线 4: 应用开发 (15分钟)
1. [01_Overview.md](01_Overview.md) - 了解库功能
2. [05_API_Differences.md](05_API_Differences.md) - 确认 API 兼容性
3. 参考标准 ncurses 文档进行开发

## 关键信息速查

### Patch 应用状态

| Patch | 类型 | 状态 |
|-------|------|------|
| ncurses-kbs.patch | 终端增强 | ✅ 已应用 |
| ncurses-urxvt.patch | 终端增强 | ✅ 已应用 |
| ncurses-libs.patch | 构建适配 | ✅ 已应用 |
| ncurses-config.patch | 构建适配 | ✅ 已应用 |
| cross_compile_support_ohos.patch | **OH 特有** | ✅ 已应用 |
| CVE-2023-29491.patch | **安全修复** | ✅ 已应用 |

### 安全状况

```
┌────────────────────────────────────────┐
│ 安全评级: ✅ 安全                      │
│                                        │
│ • CVE-2023-29491: 已修复               │
│ • 历史 CVE: 全部已修复                 │
│ • OH Patch: 无新增风险                 │
└────────────────────────────────────────┘
```

### 构建系统

| 项目 | 配置 |
|------|------|
| 构建系统 | Autotools (configure + make) |
| BUILD.gn | ❌ 无 |
| spec 文件 | ✅ ncurses.spec |
| 交叉编译 | ✅ 支持 ohos 平台 |

## 注意事项

### ⚠️ 待确认事项

1. **依赖关系**: 未在 BUILD.gn 中发现直接依赖 ncurses 的组件
   - 建议进一步调查实际使用场景
   - 可能用于开发工具、terminfo 数据库或 prebuilt 二进制

2. **使用场景**: 该库在 OH 中的具体使用方式需要验证
   - 是否为系统预装库？
   - 哪些工具实际使用了 ncurses？

### 维护建议

1. **升级时保留 OH 特有 Patch**:
   - `cross_compile_support_ohos.patch`
   - `ncurses-config.patch`
   - `ncurses-libs.patch`

2. **定期安全更新**:
   - 监控上游安全公告
   - 及时应用新的 CVE 修复

3. **推向上游**:
   - 终端类型增强 Patch 可尝试推向上游
   - OH 特有 Patch 需保留

## 参考资源

### 官方文档
- ncurses 主页: https://invisible-island.net/ncurses/
- 手册页: https://invisible-island.net/ncurses/man/
- FAQ: https://invisible-island.net/ncurses/ncurses.faq.html

### OpenHarmony 相关
- OH 源码中的 Patch 文件位于库根目录
- spec 文件: `ncurses.spec`

### 安全资源
- CVE 数据库: https://cve.mitre.org/
- NVD: https://nvd.nist.gov/

---

*最后更新: 2025-02-08*
*文档版本: 1.0*
