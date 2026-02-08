# surface_lite Wiki

> OpenHarmony 轻量图形 Surface Buffer 模块工程文档

---

## 文档概述

本 Wiki 提供 **surface_lite** 模块的完整工程分析文档，帮助开发者快速理解项目架构、接口设计、构建系统和安全特性。

### 适用范围

- 子系统: **graphic** (图形子系统)
- 组件名: **surface_lite** (@ohos/surface_lite)
- 适用系统: **small** (轻量系统)
- 版本: 3.1

### 文档生成信息

- **生成时间**: 2026-02-06
- **代码版本**: OpenHarmony master 分支
- **文档语言**: 中文

---

## 覆盖范围

### ✅ 已覆盖内容

1. **项目概览** - 定位、边界、核心能力
2. **目录结构** - 模块职责划分
3. **架构说明** - 组件图、数据流、状态机
4. **对外 API** - Surface/SurfaceBuffer 接口清单
5. **内部 API** - 模块内部接口和实现细节
6. **GN 构建** - Targets、依赖、产物
7. **安全评审** - 攻击面、风险点、修复建议
8. **调试指南** - 常见问题定位

### ❌ 未覆盖内容

1. **测试代码** - 本 Wiki 不包含 test/ 目录内容
2. **N-API 绑定** - 本模块无 JS 接口
3. **具体使用示例** - 请参考 window_window_manager_lite 文档

---

## 阅读指南

### 新人推荐阅读顺序

1. [00_Overview.md](./00_Overview.md) - 了解项目定位和边界
2. [02_DirectoryStructure.md](./02_DirectoryStructure.md) - 熟悉代码组织
3. [01_Architecture.md](./01_Architecture.md) - 理解核心架构
4. [03_Public_API.md](./03_Public_API.md) - 学习对外接口
5. [05_GN_Build.md](./05_GN_Build.md) - 了解构建方式
6. [06_Security.md](./06_Security.md) - 掌握安全注意事项

### 根据角色选择

| 角色 | 推荐阅读 |
|------|----------|
| **应用开发者** | 00_Overview → 03_Public_API → 05_GN_Build |
| **系统开发者** | 全部内容，重点关注 01_Architecture 和 04_Inner_API |
| **安全审计** | 06_Security → 01_Architecture → 04_Inner_API |
| **维护工程师** | 07_Troubleshooting → 01_Architecture → 05_GN_Build |

---

## 如何更新本文档

本文档基于代码分析自动生成。当代码变更时：

1. 重新分析代码变更部分
2. 更新 `wiki/_work/NOTES.md` 中的事实记录
3. 同步修改对应 Wiki 页面
4. 更新 `SUMMARY.md` 导航（如有新增页面）
5. 在本文档底部记录更新日志

---

## 相关资源

- **源码仓库**: `foundation/graphic/surface_lite`
- **相关子系统**:
  - [window_window_manager_lite](../window_window_manager_lite/) - 窗口管理
  - [graphic_graphic_utils_lite](../graphic_graphic_utils_lite/) - 图形工具
  - [arkui_ui_lite](../arkui_ui_lite/) - UI 框架
- **OpenHarmony 文档**: [图形子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/图形子系统.md)

---

## 更新日志

| 日期 | 版本 | 变更内容 |
|------|------|----------|
| 2026-02-06 | v1.0 | 初始版本，完成全部分析 |

---

*本文档由 OpenHarmony 工程 Agent 自动生成*
