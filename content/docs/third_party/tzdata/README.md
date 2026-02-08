# tzdata - OpenHarmony Wiki

本文档是 IANA Time Zone Database (tzdata) 在 OpenHarmony 中的集成与适配说明。

## 库概览

| 属性 | 值 |
|-----|-----|
| **原始库名称** | tzdata / tzdb / zoneinfo |
| **上游版本** | 2025b |
| **上游地址** | https://github.com/eggert/tz |
| **许可证** | Public Domain / BSD 3-Clause |
| **OH 组件名称** | @ohos/tzdata |
| **OH 组件版本** | 4.0 |
| **所属子系统** | thirdparty |

## OpenHarmony 适配概述

tzdata 在 OpenHarmony 中的集成方式**不同于常规第三方库**：

### 关键特点

1. **无代码 Patch**：本库没有任何 Patch 文件，因为 tzdata 主要是时区数据而非代码库
2. **预构建数据**：OH 使用预构建的二进制时区数据文件，而非从源码编译
3. **纯数据组件**：仅作为时区数据库的资源提供者，不参与代码链接
4. **平台差异化**：支持标准系统和穿戴设备的差异化时区列表

### 适配方式

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 系统                          │
│                                                              │
│   ┌──────────────────┐      ┌──────────────────────┐        │
│   │  tzdata 组件     │      │   ICU 时区组件        │        │
│   │  (本库)          │      │   (global/timezone)   │        │
│   ├──────────────────┤      ├──────────────────────┤        │
│   │ /system/etc/     │      │ /system/etc/         │        │
│   │   zoneinfo/      │      │   icu_tzdata/        │        │
│   │   - tzdata       │      │   - zoneinfo64.res   │        │
│   │   - timezone_    │      └──────────────────────┘        │
│   │     list.cfg     │                                       │
│   └──────────────────┘                                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 文档导航

### 快速阅读路线

- **了解本库在 OH 中的作用** → [01_Overview.md](./01_Overview.md)
- **查看 Patch 分析** → [02_Patches.md](./02_Patches.md)（本库无 Patch）
- **理解构建集成** → [03_Build_Integration.md](./03_Build_Integration.md)
- **查看依赖关系** → [04_Usage_in_OH.md](./04_Usage_in_OH.md)
- **安全风险分析** → [06_Security.md](./06_Security.md)

### 按角色阅读

| 角色 | 推荐阅读 |
|-----|---------|
| **系统开发者** | 全部文档 |
| **应用开发者** | 01_Overview.md, 04_Usage_in_OH.md |
| **安全工程师** | 06_Security.md |
| **升级维护者** | 02_Patches.md, 03_Build_Integration.md |

## 版本说明

### 当前版本状态

- **README.OpenSource 声明**: 2025b
- **version 文件**: 2024a
- **NEWS 最新版本**: 2025b

**注意**: version 文件 (2024a) 与 README.OpenSource (2025b) 存在版本不一致的情况。

## 相关资源

- [IANA Time Zone Database 官网](https://www.iana.org/time-zones)
- [上游 GitHub 仓库](https://github.com/eggert/tz)
- [时区数据库理论说明](./theory.html)（本库原始文档）

---

*本文档由 OpenHarmony Wiki 生成工具创建，最后更新于 2025-02-08*
