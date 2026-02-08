# OpenHarmony utils_lite 工程 Wiki

> 本 Wiki 为 OpenHarmony `commonlibrary/utils_lite` 仓库的工程文档，旨在帮助开发者快速理解项目架构、API 使用、构建配置及安全注意事项。

## 文档覆盖范围

| 类别 | 状态 | 说明 |
|------|------|------|
| 项目概述 | ✅ 已完成 | 项目定位、能力说明、支持平台 |
| 目录结构 | ✅ 已完成 | 模块职责归类、文件布局 |
| N-API 参考 | ✅ 已完成 | JS 接口清单、参数说明、调用链 |
| 内部 API | ✅ 已完成 | C/C++ 接口、模块依赖 |
| GN 构建 | ✅ 已完成 | Targets 清单、编译配置、产物映射 |
| 编译产物 | ✅ 已完成 | .so/.a 文件、安装路径、加载关系 |
| 安全评审 | ⚠️ 部分完成 | 攻击面分析、风险清单 |
| 故障排查 | ✅ 已完成 | 常见问题与定位路径 |

## 文档更新方式

本 Wiki 由自动化工具根据代码仓库内容生成。更新方式如下：

### 方式一：手动更新

1. 修改对应章节的 Markdown 文件
2. 提交更改到仓库

### 方式二：重新生成

执行 Wiki 生成脚本（如果有）以同步最新代码变更。

### 更新原则

- **API 变更**：同步更新 N-API 参考章节
- **构建配置变更**：更新 GN 构建章节
- **新增模块**：添加新文档并更新 SUMMARY.md
- **安全修复**：更新安全评审章节

## 生成信息

- **生成时间**：2026-02-06
- **仓库版本**：基于当前 HEAD
- **文档语言**：中文（简体）
- **维护者**：OpenHarmony Community

## 快速索引

### 核心模块

- [文件操作 API](04_NAPI_Reference.md)：`UtilsFileOpen`、`UtilsFileRead`、`UtilsFileWrite` 等
- [KV 存储 API](04_NAPI_Reference.md)：`UtilsGetValue`、`UtilsSetValue`、`UtilsDeleteValue`
- [定时器 API](04_NAPI_Reference.md)：`KalTimerCreate`、`StartTimerTask` 等
- [内部 C API](05_Inner_API.md)：各模块内部接口说明

### 构建配置

- [GN Targets](06_GN_Build.md)：完整 targets 清单
- [编译产物](07_Build_Artifacts.md)：产物路径与加载关系

### 安全与故障

- [安全风险评审](08_Security_Review.md)：攻击面分析与修复建议
- [故障排查](09_Troubleshooting.md)：常见问题与解决方案

## 反馈与贡献

如发现文档错误或有改进建议，请提交 Issue 或 Pull Request 到 [OpenHarmony utils_lite 仓库](https://gitee.com/openharmony/commonlibrary_utils_lite)。
