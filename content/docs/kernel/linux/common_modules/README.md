# OpenHarmony Linux 内核通用模块 Wiki

> 最后更新: 2026-02-06
> 仓库: [kernel_linux_common_modules](https://gitee.com/openharmony-sig/kernel_linux_common_modules)

---

## 一、项目概述

### 1.1 仓库定位

本仓库（`kernel_linux_common_modules`）是 OpenHarmony 内核通用模块的容器仓库，用于集中存放各内核领域的独立模块。

**核心特性**:
- **通用性**: 模块可在 OpenHarmony 支持的任意 Linux 内核版本上运行
- **隔离性**: 各模块独立目录，便于维护和评审
- **可构建**: 基于 GN 构建系统，支持生成内核模块（.ko）

### 1.2 适用范围

| 类型 | 说明 |
|------|------|
| **适用** | 通用内核模块、无特定平台/硬件依赖 |
| **不适用** | 特定芯片平台驱动、特定硬件驱动 |

### 1.3 许可证

所有模块使用 **GPL-2.0-or-later** 许可证。

**证据来源**: `LICENSE:1-12`

---

## 二、Wiki 覆盖范围

### 2.1 已覆盖模块

| 模块 | 状态 | 复杂度 | 文档位置 |
|------|------|--------|----------|
| tzdriver | 已完成 | 高 | [modules/tzdriver.md](modules/tzdriver.md) |
| memory_security | 已完成 | 高 | [modules/memory_security.md](modules/memory_security.md) |
| pac | 已完成 | 高 | [modules/pac.md](modules/pac.md) |
| container_escape_detection | 已完成 | 高 | [modules/container_escape_detection.md](modules/container_escape_detection.md) |
| code_sign | 已完成 | 中 | [modules/code_sign.md](modules/code_sign.md) |
| qos_auth | 已完成 | 中 | [modules/qos_auth.md](modules/qos_auth.md) |
| newip | 已完成 | 高 | [modules/newip.md](modules/newip.md) |
| xpm | 已完成 | 中 | [modules/xpm.md](modules/xpm.md) |
| ucollection | 已完成 | 低 | [modules/ucollection.md](modules/ucollection.md) |
| module_sample | 已完成 | 低 | [modules/module_sample.md](modules/module_sample.md) |

### 2.2 未覆盖内容

| 类型 | 说明 |
|------|------|
| N-API | 本仓库为内核模块，无用户态 N-API |
| System Ability | 不涉及用户态 IPC 框架 |
| 测试代码 | 按规范不引用测试文件 |

---

## 三、文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 全站导航
├── 00_Overview.md              # 项目概览
├── 01_Modules.md               # 模块清单
├── 02_Architecture.md          # 整体架构
├── 03_Build_System.md          # 构建系统
├── 04_Security_Review.md       # 安全风险评审
├── 05_Troubleshooting.md       # 常见问题
└── modules/
    ├── tzdriver.md             # TrustZone 驱动
    ├── memory_security.md      # 内存安全
    ├── pac.md                  # 指针认证
    ├── container_escape_detection.md  # 容器逃逸检测
    ├── code_sign.md            # 代码签名
    ├── qos_auth.md             # QoS 认证
    ├── newip.md                # 新 IP 协议
    ├── xpm.md                  # 包管理
    ├── ucollection.md          # 集合操作
    └── module_sample.md        # 示例模块
```

---

## 四、快速入门

### 4.1 新人阅读顺序

1. **[00_Overview.md](00_Overview.md)** - 项目整体介绍
2. **[01_Modules.md](01_Modules.md)** - 模块功能概览
3. **[02_Architecture.md](02_Architecture.md)** - 架构设计
4. 选择感兴趣的模块文档深入阅读

### 4.2 贡献者指南

如需为 Wiki 贡献内容：

1. 在对应模块目录下创建/修改 `.md` 文件
2. 确保包含**代码证据**（路径 + 行号）
3. 更新 `SUMMARY.md` 添加导航链接
4. 运行校验脚本确认链接有效

---

## 五、更新日志

| 日期 | 更新内容 | 负责人 |
|------|----------|--------|
| 2026-02-06 | 初始化 Wiki，添加所有模块文档 | Sisyphus |

---

## 六、相关链接

| 链接 | 说明 |
|------|------|
| [OpenHarmony 内核 SIG](https://gitee.com/openharmony/community/blob/master/sig/sig_kernel/sig_kernel_cn.md) | 内核社区 |
| [OAT Tool](https://gitee.com/openharmony-sig/tools_oat/blob/master/README_zh.md) | 开源审视工具 |
| [ko 构建指导](README.md#ko模块指导) | 内核模块构建 |
| [NewIP 开发手册](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/kernel/kernel-standard-newip.md) | NewIP 协议文档 |

---

*本 Wiki 基于代码证据自动生成*
