# 文档导航

本文档面向需要理解或集成 **OpenHarmony 生态规则管控服务** 的开发者，提供双路线导航以满足不同受众需求。

---

## 推荐阅读顺序

### 🌱 新人学习路线

**目标**: 快速理解项目定位、架构和使用方法

1. **[00_Overview.md](00_Overview.md)** - 项目概览、定位、核心能力
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 目录结构与模块职责
3. **[02_Architecture.md](02_Architecture.md)** - 架构设计与数据流
4. **[03_Inner_API.md](03_Inner_API.md)** - Inner API 接口规范
5. **[04_GN_Build.md](04_GN_Build.md)** - GN 构建配置与编译产物
6. **[appendix/FAQ.md](appendix/FAQ.md)** - 常见问题与调试指南

**预计时间**: 30-45 分钟

**输出**: 能够理解项目定位、找到核心代码、知道如何调用接口

---

### 🔒 安全研究路线

**目标**: 快速识别攻击面、理解安全机制、发现潜在风险

1. **[05_Security_Review.md](05_Security_Review.md)** - 安全风险评审（⚠️ 优先）
   - 威胁模型与信任边界
   - 攻击面分析
   - 5 类安全风险详解
2. **[02_Architecture.md](02_Architecture.md)** - 架构设计与数据流
   - IPC 通信流程
   - 服务生命周期
3. **[03_Inner_API.md](03_Inner_API.md)** - Inner API 接口规范
   - 输入参数分析（Want、CallerInfo）
   - 错误码处理
4. **[00_Overview.md](00_Overview.md)** - 项目概览
   - 运行环境与权限要求
5. **[appendix/Callgraphs.md](appendix/Callgraphs.md)** - 关键调用链
   - 完整的数据流向
6. **[appendix/FAQ.md](appendix/FAQ.md)** - 安全相关问题

**预计时间**: 45-60 分钟

**输出**: 能够识别所有外部输入入口、定位敏感操作、理解权限校验逻辑

---

## 快速跳转

### 按受众分类

| 主题 | 新人 | 安全研究员 | 文档 |
|------|------|-----------|------|
| 项目定位 | ✅ | ✅ | [00_Overview.md](00_Overview.md) |
| 架构设计 | ✅ | ✅ | [02_Architecture.md](02_Architecture.md) |
| API 接口清单 | ✅ | ✅ | [03_Inner_API.md](03_Inner_API.md) |
| 编译构建 | ✅ | - | [04_GN_Build.md](04_GN_Build.md) |
| 安全机制 | - | ✅ | [05_Security_Review.md](05_Security_Review.md) |
| 调用链分析 | - | ✅ | [appendix/Callgraphs.md](appendix/Callgraphs.md) |
| 常见问题 | ✅ | ✅ | [appendix/FAQ.md](appendix/FAQ.md) |

### 按主题分类

| 主题 | 文档 |
|------|------|
| API 接口清单 | [03_Inner_API.md](03_Inner_API.md) |
| 编译构建 | [04_GN_Build.md](04_GN_Build.md) |
| 安全机制 | [05_Security_Review.md](05_Security_Review.md) |
| 错误码 | [03_Inner_API.md#错误码](03_Inner_API.md#错误码) |
| SA 配置 | [appendix/Config_Flags.md](appendix/Config_Flags.md) |
| 常见问题 | [appendix/FAQ.md](appendix/FAQ.md) |

---

## 文档状态

| 文档 | 状态 | 更新时间 |
|------|------|---------|
| [00_Overview.md](00_Overview.md) | ✅ 已完成 | 2026-02-06 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | ✅ 已完成 | 2026-02-06 |
| [02_Architecture.md](02_Architecture.md) | ✅ 已完成 | 2026-02-06 |
| [03_Inner_API.md](03_Inner_API.md) | ✅ 已完成 | 2026-02-06 |
| [04_GN_Build.md](04_GN_Build.md) | ✅ 已完成 | 2026-02-06 |
| [05_Security_Review.md](05_Security_Review.md) | ✅ 已完成 | 2026-02-06 |
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | ✅ 已完成 | 2026-02-06 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | ✅ 已完成 | 2026-02-06 |
| [appendix/FAQ.md](appendix/FAQ.md) | ✅ 已完成 | 2026-02-07 |

**最后复核**: 2026-02-07
