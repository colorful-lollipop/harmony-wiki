# 00_Overview.md

## 目的

本文档提供 XTS Tools 项目的整体概览，包括项目定位、边界、核心能力、运行环境与关键概念。帮助新人在进入细节前建立全局认知。

## 适用范围

- 适用对象：OpenHarmony XTS Tools 仓库
- 不涵盖：上层 ACTS 测试用例（/test/xts/acts）、产品端业务逻辑

---

## 项目定位与边界

- XTS（X Test Suite）子系统包含认证测试套件，当前支持 ACTS（应用兼容性测试），未来支持 DCTS（设备兼容性测试）。
  - 证据：README.md:19-20 "The X test suite (XTS) subsystem contains..."
  - 证据：README.md:21-24 "This subsystem contains the ACTS and tools software package."

- tools 包为测试工具集合，服务于 ACTS 测试用例的开发与运行。
  - 边界：tools 不实现具体系统功能或服务，而是提供测试框架、构建工具、脚本、示例与检查工具。

---

## 核心能力

- 测试框架封装与适配
  - HCTest（C 语言，Mini 系统）——基于 Unity 框架增强
  - HCPPTest（C++，Small/Standard 系统）——基于 Googletest 框架增强
  - HJSUnit（JavaScript，Standard 系统）——基于 Jasmine 框架
  - 证据：README.md:250-497 框架与宏定义

- 构建与测试管理
  - GN 构建系统（BUILD.gn/.gni）
  - 测试套件打包、调度、执行（build/suite.py 等）

- 工具链与检查
  - 标准检查（standard_check/hvigor 检查）
  - 文档格式化（xts-project-tools/format-tc-doc）
  - HVIGR 更新管理（hvigor-update）

- 示例与样例
  - AppSampleD/E、ServerSampleD/E 等，用于演示测试用例与框架集成

---

## 运行环境

- 支持的 OpenHarmony 系统类型
  - Mini 系统（≥128 KiB，MCU）
  - Small 系统（≥1 MiB，应用处理器）
  - Standard 系统（≥128 MiB，应用处理器）
  - 证据：README.md:26-40

- 开发环境
  - GN/Ninja 构建系统
  - Python 工具链（standard_check、xts-project-tools）

---

## 关键概念

| 概念 | 含义 | 证据位置 |
|-------|------|---------|
| ACTS | Application Compatibility Test Suite | README.md:19 |
| DCTS | Device Compatibility Test Suite（未来支持） | README.md:19 |
| HCTest | C 语言测试框架（Mini 系统） | README.md:250-310 |
| HCPPTest | C++ 测试框架（Small/Standard 系统） | README.md:370-483 |
| HJSUnit | JavaScript 测试框架（Standard 系统） | README.md:511-649 |
| 测试用例级别 | Level0-Level4（Smoke → Rare） | README.md:63-109 |
| 测试粒度 | SmallTest/MediumTest/LargeTest | README.md:112-144 |
| 测试类型 | Function/Performance/Power/Reliability/Security 等 | README.md:147-211 |

---

## 相关跳转

- [目录结构与模块职责](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
- [GN Targets](05_GN_Targets.md)
- [安全风险评审](07_Security_Review.md)
