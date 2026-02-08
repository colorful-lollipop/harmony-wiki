# DHCP 组件 Wiki

## 文档说明

本 Wiki 为 OpenHarmony DHCP 组件（@ohos/dhcp）的工程文档，旨在帮助开发者快速理解项目架构、接口设计和实现细节。

**生成时间**: 2025-02-06 13:19:16
**最后更新**: 2026-02-07 05:39:41
**组件版本**: 3.1.0
**组件路径**: `/foundation/communication/dhcp`

---

## 覆盖范围

### 已覆盖内容
- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ 系统架构（组件图、数据流、线程模型）
- ✅ C API 接口文档（17个导出方法）
- ✅ 内部 API 与接口稳定性
- ✅ GN 构建系统（Targets、依赖关系）
- ✅ 编译产物与运行时加载
- ✅ 安全风险评审（基于代码证据）
- ✅ 常见问题与调试指南

### 未覆盖内容
- ❌ JavaScript/N-API 绑定（JS API 在 communication_wifi 仓库）
- ❌ 测试用例实现细节
- ❌ 性能优化建议
- ❌ 详细时序图（待补充）
- ❌ 配置选项完整列表（待补充）

---

## 文档更新方式

本文档基于代码自动分析生成。当代码变更时，建议按以下方式更新：

1. **API 变更**: 修改 `04_C_API_Reference.md`
2. **架构调整**: 修改 `03_Architecture.md`
3. **新增 Target**: 修改 `06_Build_System.md`
4. **安全修复**: 更新 `08_Security_Review.md`

---

## 文档使用建议

### 新人阅读顺序
1. [00_Overview.md](00_Overview.md) - 快速了解项目
2. [01_Project_Positioning.md](01_Project_Positioning.md) - 理解边界与能力
3. [03_Architecture.md](03_Architecture.md) - 掌握系统架构
4. [04_C_API_Reference.md](04_C_API_Reference.md) - 学习接口使用
5. [08_Security_Review.md](08_Security_Review.md) - 了解安全要求

### 接口开发者
重点阅读：
- [04_C_API_Reference.md](04_C_API_Reference.md) - C API 完整文档
- [05_Inner_API.md](05_Inner_API.md) - 内部接口与稳定性

### 系统集成开发者
重点阅读：
- [03_Architecture.md](03_Architecture.md) - 组件交互与 IPC
- [06_Build_System.md](06_Build_System.md) - 构建配置与依赖
- [07_Compile_Artifacts.md](07_Compile_Artifacts.md) - 产物与安装

### 安全审计员
重点阅读：
- [08_Security_Review.md](08_Security_Review.md) - 安全风险清单
- [03_Architecture.md](03_Architecture.md) - 信任边界分析

---

## 证据追踪

本文档所有关键结论均包含代码证据引用：
- **文件路径**: 绝对路径 + 行号（如 `services/dhcp_client/src/dhcp_client_service_impl.cpp:100`）
- **符号名称**: 函数、类、宏定义
- **调用链**: 关键 API 的完整调用路径

如发现文档与代码不符，请以代码为准。

---

## 问题反馈

如有问题或建议，请通过以下方式反馈：
- 在 OpenHarmony 仓库提交 Issue
- 指出具体代码位置与文档偏差

---

## 许可证

本文档遵循 Apache License 2.0，与 DHCP 组件保持一致。
