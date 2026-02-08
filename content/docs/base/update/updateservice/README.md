# OpenHarmony Update Service Wiki

## 项目概述

本文档为 OpenHarmony **升级服务组件 (update_updateservice)** 的工程 Wiki，旨在帮助新人快速完整理解项目。

### 覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目定位与核心能力 | ✅ |  |
| 目录结构与模块职责 | ✅ |  |
| N-API 接口 | ✅ | JS API 完整清单 |
| 内部架构与 Inner API | ✅ | SA/IPC/模块依赖 |
| GN 构建配置 | ✅ | Targets 与产物 |
| 安全风险评审 | ✅ | 攻击面与修复建议 |
| 故障排查 | ⚠️ | 需补充运行时问题 |

### 更新说明

- **文档生成时间**: 2026-02-06
- **代码版本**: 基于 commit `...` (TODO: 补充)
- **文档维护**: 请随代码变更同步更新 Wiki

### 相关链接

- [OpenHarmony Update 子系统](https://gitee.com/openharmony/update_app)
- [代码仓库](https://gitee.com/openharmony/update_updateservice)
- [上游仓库](https://github.com/openharmony/update_updateservice)

---

## 阅读指南 (推荐顺序)

```
新手推荐路径:
1. README.md          → 项目基本信息
2. 00_Overview.md     → 快速概览
3. 02_N-API.md        → JS API 使用方式
4. 01_Architecture.md → 架构理解

进阶路径:
5. 03_Inner_API.md    → SA/IPC/Inner API 细节
6. 04_GN_Build.md     → 构建配置
7. 05_Security.md     → 安全风险
8. 06_Troubleshooting.md → 常见问题
```

---

## 目录

### 快速入门

- [README.md](README.md) - 本文档
- [SUMMARY.md](SUMMARY.md) - 全站导航
- [00_Overview.md](00_Overview.md) - 项目概览

### 架构设计

- [01_Architecture.md](01_Architecture.md) - 系统架构
- [03_Inner_API.md](03_Inner_API.md) - Inner API 与 IPC

### API 参考

- [02_N-API.md](02_N-API.md) - JS API 完整文档

### 构建部署

- [04_GN_Build.md](04_GN_Build.md) - GN 构建配置

### 安全运维

- [05_Security.md](05_Security.md) - 安全风险评审
- [06_Troubleshooting.md](06_Troubleshooting.md) - 故障排查指南
