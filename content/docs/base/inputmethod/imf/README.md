# OpenHarmony 输入法框架 (inputmethod_imf) Wiki

## 文档说明

本 Wiki 旨在帮助开发者快速理解和掌握 OpenHarmony 输入法框架的架构、API、构建和安全相关知识。

## 覆盖范围

### 已覆盖

- 项目概述与定位
- 四大核心模块架构
- N-API 接口清单（inputMethod、InputMethodController、inputMethodList 等）
- Inner API 接口
- NDK C 接口
- GN 构建系统与 Targets
- 编译产物与安装路径
- 权限与安全机制
- System Ability 模式

### 未覆盖

- 详细代码实现细节
- 性能优化指南
- 调试技巧（待补充）
- 最佳实践示例（待补充）

## 更新方式

当代码发生以下变化时，需要同步更新本 Wiki：

1. 新增/删除 N-API 接口
2. 修改 Inner API 签名
3. 调整 GN 构建配置
4. 新增安全相关逻辑
5. 新增/移除模块

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于 `/base/inputmethod/imf` 仓库
- **目标读者**: OpenHarmony 输入法开发者

## 文档导航

请从 [SUMMARY.md](./SUMMARY.md) 开始阅读，获取完整的文档导航。
