# Rockchip OpenHarmony Wiki

## 简介

本 Wiki 面向 Rockchip OpenHarmony 仓库 (`device/soc/rockchip`)，提供完整的工程文档，帮助开发者快速理解项目架构、接口定义、构建系统和安全风险。

## 覆盖范围

### 包含内容
- 项目定位与核心能力
- 目录结构与模块职责
- HDI/VDI 接口文档
- GN 构建系统梳理
- 编译产物说明
- 安全风险评审
- 常见问题与调试

### 支持芯片平台
| 芯片 | 内核类型 | 主要特性 |
|------|----------|----------|
| RK2206 | LiteOS-M | IoT/可穿戴，HDF驱动 |
| RK3399 | Linux | 高端平板/工业 |
| RK3566 | Linux | 中端多媒体 |
| RK3568 | Linux | 工业/边缘计算，完整多媒体 |
| RK3588 | Linux | 旗舰/AI计算，8K编解码 |

### 不包含内容
- 测试相关代码 (`test/`, `tests/`, `unittest/`, `fuzz/`)
- N-API 接口（本仓库为纯硬件适配层，无 N-API 实现）
- 上层框架实现（如 `drivers/peripheral`、`foundation`）

## 更新方式

本文档基于代码生成，建议随代码更新同步维护：

1. **代码变更时**：同步更新对应 Wiki 章节
2. **新增模块时**：在相关文档中添加说明
3. **接口变更时**：更新 HDI/VDI 接口文档

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于当前工作目录最新代码
- **文档版本**: v1.0

## 快速导航

- [项目概览](00_Overview.md)
- [架构说明](01_Architecture.md)
- [目录结构](02_Directory_Structure.md)
- [HDI/VDI 接口](03_HDI_Interfaces.md)
- [GN 构建系统](04_GN_Build.md)
- [编译产物](05_Compilation_Products.md)
- [安全风险评审](06_Security_Analysis.md)
- [常见问题](07_Troubleshooting.md)

## 贡献指南

如需更新本文档：
1. 修改 `wiki/` 目录下的对应 `.md` 文件
2. 更新 `SUMMARY.md` 中的链接
3. 在本文档中记录变更历史

## 变更历史

| 日期 | 版本 | 变更内容 |
|------|------|----------|
| 2026-02-06 | v1.0 | 初始版本，完成全站文档 |
