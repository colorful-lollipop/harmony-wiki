# display_manager Wiki 文档导航

> 完整的文档导航与新人阅读路径

---

## 快速导航

### 核心文档

| 文档 | 描述 | 预计阅读时间 |
|------|------|-------------|
| [00_Overview.md](00_Overview.md) | 项目概览、核心能力、运行环境 | 5 分钟 |
| [01_Project_Scope.md](01_Project_Scope.md) | 项目定位、边界、关键概念 | 10 分钟 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录结构与模块职责 | 10 分钟 |
| [03_Architecture.md](03_Architecture.md) | 架构设计、组件图、数据流 | 30 分钟 |
| [04_NAPI_Interface.md](04_NAPI_Interface.md) | 对外 N-API（JS API） | 20 分钟 |
| [05_Internal_API.md](05_Internal_API.md) | 内部 API、依赖关系、接口稳定性 | 25 分钟 |
| [06_GN_Targets.md](06_GN_Targets.md) | GN 构建系统、编译产物 | 15 分钟 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物、安装路径、运行时 | 10 分钟 |
| [08_Security_Analysis.md](08_Security_Analysis.md) | 安全风险评审、可被利用点 | 30 分钟 |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题、定位路径 | 15 分钟 |

**总计阅读时间**：约 2.5 小时

---

## 双路线导航

### 路线一：新人学习路线

适合初学者快速上手和理解项目：

#### 快速入门（30 分钟）
1. [00_Overview.md](00_Overview.md) - 项目概览、核心能力
2. [01_Project_Scope.md](01_Project_Scope.md) - 项目定位与边界
3. [04_NAPI_Interface.md](04_NAPI_Interface.md) - JS API 使用

#### 深入理解（1.5 小时）
1. [02_Directory_Structure.md](02_Directory_Structure.md) - 代码组织
2. [03_Architecture.md](03_Architecture.md) - 架构设计
3. [05_Internal_API.md](05_Internal_API.md) - 内部接口
4. [06_GN_Targets.md](06_GN_Targets.md) - 构建系统

#### 实践与调试（30 分钟）
1. [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物
2. [09_Troubleshooting.md](09_Troubleshooting.md) - 常见问题

### 路线二：安全研究路线

适合安全研究员进行代码审计和风险评估：

#### 快速评估（30 分钟）
1. [00_Overview.md](00_Overview.md) - 快速了解项目
2. [01_Project_Scope.md](01_Project_Scope.md#安全边界) - 安全边界
3. [08_Security_Analysis.md](08_Security_Analysis.md#执行摘要) - 安全态势

#### 攻击面分析（1 小时）
1. [04_NAPI_Interface.md](04_NAPI_Interface.md) - 外部输入入口
2. [05_Internal_API.md](05_Internal_API.md) - IPC 接口
3. [08_Security_Analysis.md](08_Security_Analysis.md#攻击面分析) - 攻击面

#### 深度审计（1.5 小时）
1. [03_Architecture.md](03_Architecture.md#信任边界) - 信任边界
2. [08_Security_Analysis.md](08_Security_Analysis.md#详细风险评估) - 风险详情
3. [appendix/Callgraphs.md](appendix/Callgraphs.md) - 调用链追踪

---

## 附录文档

| 文档 | 描述 | 适用场景 |
|------|------|---------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链追踪 | 调试性能分析 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 配置标志说明 | 功能定制 |

---

## 按主题查找

### API 接口
- **JS API**：参见 [04_NAPI_Interface.md](04_NAPI_Interface.md)
- **内部 API**：参见 [05_Internal_API.md](05_Internal_API.md)

### 构建相关
- **GN Targets**：参见 [06_GN_Targets.md](06_GN_Targets.md)
- **编译产物**：参见 [07_Build_Artifacts.md](07_Build_Artifacts.md)
- **配置选项**：参见 [appendix/Config_Flags.md](appendix/Config_Flags.md)

### 架构与安全
- **架构设计**：参见 [03_Architecture.md](03_Architecture.md)
- **安全分析**：参见 [08_Security_Analysis.md](08_Security_Analysis.md)

---

## 文档索引

### 关键概念

- [BrightnessManager](03_Architecture.md#亮度管理器brightnessmanager)
- [ScreenController](03_Architecture.md#屏幕控制器screencontroller)
- [DisplayPowerMgrService](03_Architecture.md#显示电源服务displaypowermgrservice)
- [System Ability](03_Architecture.md#system-ability-注册)
- [ZIDL 接口](03_Architecture.md#zidl-通信架构)

### 功能特性

- [屏幕开关控制](00_Overview.md#核心能力)
- [亮度调节](00_Overview.md#核心能力)
- [自动亮度](00_Overview.md#核心能力)
- [渐变动画](03_Architecture.md#渐变动画器gradualanimator)

---

## 更新日志

- **2026-02-06**：初始版本 v1.0，基于代码扫描生成完整文档骨架

---

## 贡献指南

如发现文档错误或需要补充，请遵循以下步骤：
1. 在代码中查找证据（文件路径、符号名、行号）
2. 在相应文档中标注 `TODO(需确认)` 说明缺失证据
3. 提交 PR 时引用相关证据
4. 更新本文档的更新日志
