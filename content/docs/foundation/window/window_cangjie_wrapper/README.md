# window_cangjie_wrapper Wiki

> 为 OpenHarmony 窗口仓颉封装项目生成的技术文档

---

## 概述

本文档集涵盖 `window_cangjie_wrapper` 项目的架构、API、构建系统及安全风险分析。

**项目定位**: 为 OpenHarmony 窗口管理子系统提供仓颉（Cangjie）语言封装层
**适用系统**: OpenHarmony 标准设备（API Level 22+）
**子系统**: window
**组件**: window_cangjie_wrapper
**版本**: 6.1
**许可**: Apache License 2.0

---

## 文档覆盖范围

✅ **已覆盖**:
- 项目概览与定位
- 目录结构与模块职责
- 系统架构（组件图、数据流、线程模型）
- 对外 API（仓颉接口）
- 内部 API（模块接口、依赖方向）
- GN Targets 与编译产物
- 安全风险评审

⚠️ **未覆盖**:
- 测试用例实现细节
- Native 层（window_manager:cj_window_ffi/cj_display_ffi）实现细节
- 性能优化指南

---

## 文档更新

### 如何更新文档

1. 代码变更后，对应更新相关章节
2. 保持证据可追溯性（路径 + 符号 + 行号）
3. 同步更新 `wiki/_work/NOTES.md` 中的事实记录

### 生成时间

- **首次生成**: 2025-02-06
- **最后更新**: 2025-02-06

---

## 快速导航

1. **新人入门**: 先阅读 [SUMMARY.md](./SUMMARY.md)
2. **概览理解**: 阅读 [00_Overview.md](./00_Overview.md)
3. **API 使用**: 参考 [04_External_API.md](./04_External_API.md)
4. **架构深入**: 阅读 [03_Architecture.md](./03_Architecture.md)
5. **构建调试**: 参考 [09_FAQ.md](./09_FAQ.md)

---

## 依赖仓库

本文档引用以下 OpenHarmony 仓库：
- [ability_ability_cangjie_wrapper](https://gitcode.com/openharmony-sig/ability_ability_cangjie_wrapper)
- [arkui_arkui_cangjie_wrapper](https://gitcode.com/openharmony-sig/arkui_arkui_cangjie_wrapper)
- [arkcompiler_cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- [hiviewdfx_hiviewdfx_cangjie_wrapper](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper)
- [multimedia_multimedia_cangjie_wrapper](https://gitcode.com/openharmony-sig/multimedia_multimedia_cangjie_wrapper)
- [ability_ability_runtime](https://gitcode.com/openharmony/ability_ability_runtime)
- [window_window_manager](https://gitcode.com/openharmony/window_window_manager)

---

## 问题反馈

如发现文档错误或缺失，请在 OpenHarmony 社区提交 Issue 或 PR。

---

## 许可证

本文档遵循 Apache License 2.0 许可证。
