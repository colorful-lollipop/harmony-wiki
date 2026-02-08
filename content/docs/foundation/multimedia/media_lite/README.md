# Media Lite Wiki

本文档提供 media_lite 项目的全面技术文档，帮助开发者快速理解项目架构、接口和实现细节。

---

## 概述

**项目名称**: @ohos/media_lite
**描述**: 提供播放、录制、解析、解码等接口能力，并提供媒体播放录制引擎服务化能力
**版本**: 3.1
**子系统**: multimedia
**适配系统**: mini, small
**语言要求**: C++11 或以上
**许可证**: Apache License 2.0

---

## 覆盖范围

### 已覆盖

- [x] 项目定位、边界、核心能力
- [x] 目录结构与模块职责
- [x] 架构说明（组件图、数据流、线程模型）
- [x] 对外 JSI（JavaScript Interface）接口
- [x] 内部 API 和模块接口
- [x] GN Targets 和编译产物
- [x] 权限鉴权机制
- [x] 安全风险评审

### 未覆盖

- [ ] 完整的错误码定义和映射（TODO：需要进一步分析 media_errors.h）
- [ ] 详细的线程模型和资源生命周期管理
- [ ] Surface 生命周期的完整分析
- [ ] 内存管理策略和泄漏风险点
- [ ] 性能优化建议

---

## 更新方式

本文档基于代码自动生成，如需更新：

1. **代码变更后**：更新对应模块的文档章节
2. **架构调整**：更新 `02_Architecture.md` 和相关图示
3. **新增 API**：更新 `03_JSI_Interfaces.md` 或 `04_Inner_API.md`
4. **安全修复**：更新 `07_Security_Audit.md`

建议的更新频率：
- 重大架构变更：立即更新
- API 新增/修改：每个版本更新
- 安全修复：每个版本更新

---

## 生成时间

生成时间：2026-02-07
代码库版本：master (基于最新代码分析）

---

## 相关文档

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [媒体子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/%E5%AA%8C%E5%AA%9F%E7%BB%8F%E7%BB%9F%E7%BB%9F%E7%BB%9F.md)

### Wiki 工作文件

- [项目评估](_work/ASSESSMENT.md) - 项目类型判定、受众需求分析、文档策略
- [工作笔记](_work/NOTES.md) - 代码证据汇总和发现记录
- [任务计划](_work/PLAN.md) - Wiki 生成计划和进度追踪
