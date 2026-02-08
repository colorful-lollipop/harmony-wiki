# OpenHarmony Sensor 子系统 Wiki

> 本文档系统化地记录了 OpenHarmony Sensor 子系统的架构、API、构建系统和安全风险。
>
> **生成时间**: 2026-02-06
> **仓库路径**: `/base/sensors/sensor`
> **版本**: 3.1

---

## 文档范围

### 已覆盖内容

- [ ] 项目概览与核心概念
- [ ] 目录结构与模块职责
- [ ] 架构说明（组件图/数据流/线程模型）
- [ ] 对外 N-API 完整参考
- [ ] 内部模块接口
- [ ] GN 构建目标梳理
- [ ] 编译产物与加载关系
- [ ] 安全风险评审
- [ ] 常见构建/运行/调试问题

### 未覆盖内容

- 测试代码（test/ 目录）
- 其他 OpenHarmony 子系统（如 sensors_miscdevice）
- 传感器硬件驱动层（HDF drivers）

---

## 如何更新本文档

本文档基于代码分析生成。当传感器子系统代码更新时，建议：

1. **接口变更**: 更新 `03_NAPI_Reference.md` 和 `04_Internal_API.md`
2. **架构变更**: 更新 `02_Architecture.md` 中的组件图和数据流
3. **新增传感器**: 更新 `00_Overview.md` 中的传感器类型列表
4. **构建配置变更**: 更新 `05_GN_Targets.md` 中的 target 和依赖
5. **安全修复**: 更新 `07_Security_Audit.md` 中的风险清单

---

## 快速导航

如果你是**新人**，建议按以下顺序阅读：

1. [00_Overview.md](00_Overview.md) - 了解传感器子系统整体架构
2. [01_Directory_Structure.md](01_Directory_Structure.md) - 熟悉代码组织
3. [03_NAPI_Reference.md](03_NAPI_Reference.md) - 学习如何使用传感器 JS API
4. [02_Architecture.md](02_Architecture.md) - 理解数据流和线程模型
5. [05_GN_Targets.md](05_GN_Targets.md) - 了解构建系统

如果你是**开发者**，可能需要：

- [03_NAPI_Reference.md](03_NAPI_Reference.md) - 参考 API 细节
- [04_Internal_API.md](04_Internal_API.md) - 理解内部接口
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 了解产物和依赖
- [08_Troubleshooting.md](08_Troubleshooting.md) - 解决问题

如果你是**安全审计人员**，重点关注：

- [07_Security_Audit.md](07_Security_Audit.md) - 安全风险分析和可利用点

---

## 文档维护

### 贡献者

如果你发现本文档有误或需要更新，请：

1. 修改对应的 Markdown 文件
2. 更新 `SUMMARY.md` 中的链接（如有新增文件）
3. 记录修改原因和证据（路径+符号）到相关章节
4. 提交 PR 并更新 `wiki/_work/PLAN.md` 的进度

---

## 术语表

| 术语 | 定义 |
|------|------|
| SA | System Ability，OpenHarmony 中的系统能力管理机制 |
| N-API | Node-API，用于 JavaScript 和 C/C++ 之间的绑定 |
| IDL | Interface Definition Language，用于定义 IPC 接口 |
| HDI | Hardware Driver Interface，硬件驱动接口 |
| HDF | Hardware Driver Foundation，硬件驱动框架 |
| NDK | Native Development Kit，原生开发工具包 |
| CJ | Cangjie，OpenHarmony 编程语言 |
| ETS | Extension TypeScript，ArkTS 的扩展 |
| Token | 访问令牌，用于权限管理 |
| FFI | Foreign Function Interface，外部函数接口 |
