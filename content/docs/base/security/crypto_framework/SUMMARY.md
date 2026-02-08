# Crypto Framework - 文档导航

> 本文档提供 crypto_framework 项目的完整导航，帮助开发者快速定位所需信息。

## 新人阅读路线

### 快速入门（30分钟）

1. **[00_Overview.md](00_Overview.md)** - 项目概览
   - 什么是 crypto_framework
   - 核心能力和特性
   - 适用的使用场景

2. **[01_Project_Positioning.md](01_Project_Positioning.md)** - 项目定位
   - 项目边界和职责
   - 与其他组件的关系
   - 核心概念说明

3. **[02_Directory_Structure.md](02_Directory_Structure.md)** - 目录结构
   - 代码组织方式
   - 各模块职责说明
   - 快速定位文件

### 深入理解（2小时）

4. **[03_Architecture.md](03_Architecture.md)** - 架构设计
   - 分层架构设计
   - 组件关系图
   - 数据流转过程

5. **[04_External_API.md](04_External_API.md)** - 对外 API
   - N-API 接口清单
   - Native Kits API 参考
   - API 使用示例

6. **[05_Internal_API.md](05_Internal_API.md)** - 内部 API
   - 框架内部接口
   - SPI 接口设计
   - 模块间依赖

### 运维与集成（1小时）

7. **[06_GN_Targets.md](06_GN_Targets.md)** - 构建系统
   - GN targets 结构
   - 依赖关系分析
   - 条件编译配置

8. **[07_Build_Artifacts.md](07_Build_Artifacts.md)** - 编译产物
   - 输出产物清单
   - 安装路径说明
   - 运行时加载关系

### 安全审查（1小时）

9. **[08_Security_Review.md](08_Security_Review.md)** - 安全风险
   - 攻击面分析
   - 信任边界识别
   - 潜在利用点

### 问题排查（按需）

10. **[09_Troubleshooting.md](09_Troubleshooting.md)** - 常见问题
    - 构建问题
    - 运行时问题
    - 调试技巧

---

## 按主题浏览

### API 相关

- **N-API 接口**: [04_External_API.md](04_External_API.md#n-api-接口)
- **Native API**: [04_External_API.md](04_External_API.md#native-kits-api)
- **错误码定义**: [04_External_API.md](04_External_API.md#错误码定义)
- **参数类型**: [04_External_API.md](04_External_API.md#参数类型)

### 架构与设计

- **分层架构**: [03_Architecture.md](03_Architecture.md#分层架构)
- **SPI 机制**: [03_Architecture.md](03_Architecture.md#spi-机制)
- **插件系统**: [03_Architecture.md](03_Architecture.md#插件系统)
- **线程模型**: [03_Architecture.md](03_Architecture.md#线程模型)

### 构建与部署

- **GN Targets**: [06_GN_Targets.md](06_GN_Targets.md#主要-targets)
- **依赖关系**: [06_GN_Targets.md](06_GN_Targets.md#target-依赖关系)
- **编译产物**: [07_Build_Artifacts.md](07_Build_Artifacts.md#主要编译产物)
- **安装路径**: [07_Build_Artifacts.md](07_Build_Artifacts.md#安装路径)

### 安全与审计

- **攻击面**: [08_Security_Review.md](08_Security_Review.md#攻击面分析)
- **信任边界**: [08_Security_Review.md](08_Security_Review.md#信任边界)
- **潜在风险**: [08_Security_Review.md](08_Security_Review.md#潜在利用点)
- **安全建议**: [08_Security_Review.md](08_Security_Review.md#安全加固建议)

### 算法与功能

- **支持的算法**: [00_Overview.md](00_Overview.md#支持的算法)
- **对称加密**: [03_Architecture.md](03_Architecture.md#对称加密流程)
- **非对称加密**: [03_Architecture.md](03_Architecture.md#非对称加密流程)
- **签名验签**: [03_Architecture.md](03_Architecture.md#签名验签流程)
- **密钥协商**: [03_Architecture.md](03_Architecture.md#密钥协商流程)

---

## 文档索引

### 核心文档

| 文档 | 说明 | 阅读时长 |
|------|------|----------|
| [README.md](README.md) | 文档概览和维护指南 | 5 分钟 |
| [SUMMARY.md](SUMMARY.md) | 全站导航和路线图 | 10 分钟 |
| [00_Overview.md](00_Overview.md) | 项目概览和快速入门 | 15 分钟 |
| [01_Project_Positioning.md](01_Project_Positioning.md) | 项目定位、边界、核心能力 | 10 分钟 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构与模块职责 | 20 分钟 |
| [03_Architecture.md](03_Architecture.md) | 架构说明（组件图/数据流/时序） | 30 分钟 |
| [04_External_API.md](04_External_API.md) | 对外 API（N-API/导出符号/权限） | 30 分钟 |
| [05_Internal_API.md](05_Internal_API.md) | 内部 API（模块接口/依赖） | 30 分钟 |
| [06_GN_Targets.md](06_GN_Targets.md) | GN 目标梳理 | 20 分钟 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物（.so/.a/.hap） | 15 分钟 |
| [08_Security_Review.md](08_Security_Review.md) | 安全风险评审 | 30 分钟 |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题与定位 | 20 分钟 |

### 附录文档（可选）

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏/feature flags |

---

## 代码证据索引

本文档中的所有关键结论都有代码证据支持。证据类型包括：

- **文件路径**: 指向具体源文件或头文件
- **行号引用**: 标注关键代码所在行
- **符号名**: 函数、类、常量、宏的名称
- **代码片段**: 最小必要的代码示例

### 证据查找方式

1. **N-API 接口**: 搜索 `napi_*.cpp` 文件
2. **Native API**: 搜索 `interfaces/kits/native/include/` 目录
3. **内部接口**: 搜索 `interfaces/inner_api/` 目录
4. **SPI 接口**: 搜索 `frameworks/spi/` 目录
5. **BUILD 配置**: 搜索 `BUILD.gn` 和 `.gni` 文件

---

## 更新日志

### 2026-02-06

- ✅ 创建 Wiki 文档框架
- ✅ 完成 Phase 0-1：初始化和全局扫描
- ✅ 完成 N-API 接口梳理
- ✅ 完成目录结构分析
- ✅ 完成 GN Targets 分析
- ✅ 完成错误码定义整理
- ✅ 确认无 IPC/权限相关代码

### 待更新

- [ ] 完善加密算法详细列表
- [ ] 补充调用链图（Mermaid）
- [ ] 添加安全风险详细分析
- [ ] 完善常见问题解答

---

## 贡献者

- 生成时间: 2026-02-06
- 生成工具: OpenHarmony Wiki Generator Agent
- 代码版本: 基于 2026-02-05 的代码快照
