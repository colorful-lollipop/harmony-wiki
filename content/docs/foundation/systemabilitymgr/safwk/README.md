# System Ability Framework (SAFWK) Wiki

## 文档覆盖范围

本文档覆盖 OpenHarmony System Ability Framework (SAFWK) 的完整技术细节，包括架构设计、API 接口、构建系统和安全评审。

### 已覆盖内容

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概述与定位 | ✅ | 核心功能、运行环境、依赖关系 |
| 目录结构 | ✅ | 模块职责归类（不含测试） |
| 架构设计 | ✅ | 组件图、数据流、线程模型、时序 |
| Inner API | ✅ | C++ SDK 接口、注册宏、生命周期 |
| GN 构建系统 | ✅ | Targets、依赖、产物、安装路径 |
| 安全风险评审 | ✅ | 攻击面、信任边界、风险点 |
| 常见问题 | ✅ | 构建、运行、调试问题 |

### 未覆盖内容

| 模块 | 原因 |
|------|------|
| N-API / JS API | SAFWK 是原生 C++ 框架，不提供 N-API |
| ArkTS 绑定 | JS 层 API 由其他仓库（如 samgr）提供 |

## 更新方式

本文档基于代码自动生成，生成时间：**2026-02-06**

### 手动更新

如需更新文档，请遵循以下步骤：

1. **修改代码后更新**：
   - 修改对应头文件 (.h) 或源文件 (.cpp)
   - 更新 `wiki/_work/NOTES.md` 中的发现记录
   - 运行 `bash` 工具重新扫描相关模块

2. **添加新 API 文档**：
   - 在 `wiki/02_APIs.md` 中添加 API 清单表
   - 包含：函数名、参数、返回值、绑定位置
   - 添加代码证据：文件路径 + 行号

3. **添加新安全风险**：
   - 在 `wiki/04_Security.md` 中添加风险条目
   - 必须包含：证据路径、可利用路径、影响、修复建议

### 版本信息

| 项目 | 值 |
|------|-----|
| SAFWK 版本 | 3.1 |
| 目标系统 | OpenHarmony Standard |
| License | Apache License 2.0 |

## 新人阅读顺序

建议按以下顺序阅读：

1. **[00_Overview.md](00_Overview.md)** - 项目概览，理解 SAFWK 定位
2. **[01_Architecture.md](01_Architecture.md)** - 架构设计，理解核心组件
3. **[02_APIs.md](02_APIs.md)** - API 接口，开发 SA 必备
4. **[03_Build.md](03_Build.md)** - 构建系统，构建/调试参考
5. **[04_Security.md](04_Security.md)** - 安全评审，了解风险点
6. **[05_FAQ.md](05_FAQ.md)** - 常见问题，快速定位问题

## 相关链接

- **OpenHarmony 主仓库**: https://gitee.com/openharmony
- **SAFWK 代码仓库**: https://gitee.com/openharmony/systemabilitymgr_safwk
- **SAMGR 仓库**: https://gitee.com/openharmony/systemabilitymgr_samgr
- **开发者指南**: https://gitee.com/openharmony/docs/blob/master/zh-cn/readme.md
