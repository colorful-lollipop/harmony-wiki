# 文档导航

## 新人阅读路线

```
建议阅读顺序:
1. README.md (本文档概述)
2. 01_Overview.md (项目定位)
3. 02_Architecture.md (整体架构)
4. 03_JSI_API.md (JS接口)
5. 04_InnerAPI.md (内部接口)
6. 05_Build.md (构建配置)
7. 06_Security.md (安全评估)
```

## 文档清单

### 核心文档

| 文档 | 描述 | 最后更新 |
|------|------|---------|
| [README](README.md) | Wiki 使用指南 | 2026-02-06 |
| [SUMMARY](SUMMARY.md) | 全站导航 | 2026-02-06 |
| [01_Overview](01_Overview.md) | 项目概览与定位 | 2026-02-06 |
| [02_Architecture](02_Architecture.md) | 系统架构说明 | 2026-02-06 |
| [03_JSI_API](03_JSI_API.md) | JavaScript 接口 | 2026-02-06 |
| [04_InnerAPI](04_InnerAPI.md) | Inner API 接口 | 2026-02-06 |
| [05_Build](05_Build.md) | GN 构建配置 | 2026-02-06 |
| [06_Security](06_Security.md) | 安全风险评估 | 2026-02-06 |

### 附录

| 文档 | 描述 |
|------|------|
| [Callgraphs](appendix/Callgraphs.md) | 关键调用链图示 |

## 模块速查

### 代码位置

| 模块 | 路径 |
|------|------|
| JS 接口 | `interfaces/kit/js/` |
| Inner API | `interfaces/innerkits/` |
| 框架 (Small) | `framework/small/` |
| 框架 (Mini) | `framework/mini/` |
| 核心业务 | `services/core/` |
| 构建配置 | `build/` |

### 产物位置

| 产物 | 描述 |
|------|------|
| `libkit_device_attest.so` | JS 接口共享库 |
| `libdevattest_server.so` | Small 系统服务器 |
| `libdevattest_client.so` | Small 系统客户端 |
| `libdevattest_core.so` | 核心业务库 |
| `libdevattest_sdk.a` | Mini 系统静态库 |

## 相关资源

- [bundle.json](../bundle.json) - 组件配置
- [README.md](../README.md) - 项目原始文档
- [OpenHarmony 文档](https://gitee.com/openharmony/docs)
