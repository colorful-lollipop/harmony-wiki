# OpenHarmony Camera 示例项目 Wiki

## 项目概述

本项目是 OpenHarmony 媒体子系统的示例应用集合，提供相机、图库、桌面、设置等核心应用实现。

**项目信息**:
- **仓库名**: camera_sample_app
- **版本**: 3.1
- **许可证**: Apache License 2.0
- **目标系统**: OpenHarmony Lite (small)

**适用对象**: OpenHarmony 开发者、系统架构师、安全审计人员

---

## Wiki 范围

### 已覆盖内容
- 项目整体架构与模块职责
- 目录结构与代码组织
- 构建系统（GN）分析
- 编译产物与部署
- 安全风险分析与建议

### 未覆盖内容
- 详细的 UI 实现细节
- 具体的算法实现
- 第三方库（wpa_supplicant 等）内部实现

---

## 如何更新本文档

1. **代码变更时**: 若模块接口或架构发生重大变化，需同步更新对应 Wiki 页面
2. **新增模块时**: 需在 `SUMMARY.md` 中添加导航，并按模板创建新模块文档
3. **安全漏洞修复后**: 更新 `Security.md` 中的风险状态

---

## 生成信息

- **生成日期**: 2026-02-05
- **代码版本**: Git HEAD (待确认具体 commit)
- **文档版本**: v1.0

---

## 快速导航

- [站点导航 (SUMMARY)](./SUMMARY.md)
- [项目首页/概览](./index.md)
- [目录结构](./Structure.md)
- [架构说明](./Architecture.md)
- [对外接口](./External_API.md)
- [内部接口](./Internal_API.md)
- [GN 构建系统](./Build_System.md)
- [编译产物](./Build_Outputs.md)
- [安全风险分析](./Security.md)
- [常见问题](./FAQ.md)
