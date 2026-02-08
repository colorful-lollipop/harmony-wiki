# Hilog Lite 文档导航

本文档为 OpenHarmony DFX 子系统 hilog_lite 组件的完整工程 Wiki。

## 新人阅读路线

建议阅读顺序：

```
1. 概览 → 2. 架构 → 3. API → 4. 内部模块 → 5/6. 构建 → 7. 安全
```

---

## 核心文档

| 文档 | 描述 | 阅读优先级 |
|------|------|------------|
| [README](README.md) | 文档覆盖范围、更新方式、版本信息 | ⭐ 必读 |
| [01_Overview](01_Overview.md) | 项目定位、边界、运行环境、关键概念 | ⭐ 必读 |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型、时序图 | ⭐ 必读 |
| [03_Native_API](03_Native_API.md) | Native API 接口清单与使用指南 | ⭐ 必读 |
| [04_Internal_API](04_Internal_API.md) | 内部模块接口、依赖关系 | 🔧 进阶 |
| [05_GN_Targets](05_GN_Targets.md) | 构建目标梳理、依赖关系 | 🔧 进阶 |
| [06_Build_Outputs](06_Build_Outputs.md) | 编译产物、安装路径、加载关系 | 🔧 进阶 |
| [07_Security_Review](07_Security_Review.md) | 安全风险评审、攻击面分析 | ⚠️ 安全相关 |

---

## 附录

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链图示 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 关键配置项与 Feature Flags |

---

## 快速索引

### API 速查

| 系统类型 | 入口文件 | 主要宏 |
|----------|----------|--------|
| 轻量系统 | `interfaces/native/kits/hilog_lite/hiview_log.h` | `HILOG_DEBUG`, `HILOG_INFO` |
| 小型系统 | `interfaces/native/kits/hilog/log.h` | `HILOG_DEBUG`, `HILOG_INFO` |
| JS/ACE Lite | `frameworks/js/builtin/src/hilog_module.cpp` | `debug()`, `info()`, `warn()` |

### 构建产物

| 产物类型 | 目标 | 输出 |
|----------|------|------|
| 静态库 | `frameworks/mini:hilog_lite_static` | `libhilog_lite_static.a` |
| 动态库 | `frameworks/featured:hilog_shared` | `libhilog_shared.so` |
| 可执行 | `services/hilogcat:hilogcat` | `hilogcat` |
| 可执行 | `services/apphilogcat:apphilogcat` | `apphilogcat` |

### 配置项

| 配置 | 作用 | 默认值 |
|------|------|--------|
| `hilog_lite_file_size` | 日志文件大小 | 8192 |
| `hilog_lite_limit_level_default` | 默认限流级别 | 30 |
| `hilog_lite_disable_privacy_feature` | 禁用隐私功能 | false |

---

## 相关链接

- **OpenHarmony DFX 子系统**: [文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md)
- **hilog_lite 仓库**: [Gitee](https://gitee.com/openharmony/hiviewdfx_hilog_lite)
- **问题反馈**: [Issue Tracker](https://gitee.com/openharmony/hiviewdfx_hilog_lite/issues)

---

## 修订历史

| 日期 | 版本 | 变更说明 |
|------|------|----------|
| 2026-02-06 | 1.0 | 初始版本，基于代码证据生成 |
