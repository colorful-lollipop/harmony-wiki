# 短彩信模块 Wiki

## 概述

本文档集合提供了 OpenHarmony 短彩信模块 (telephony/sms_mms) 的完整技术文档，帮助开发者和架构师快速理解项目。

## 文档范围

### 已覆盖内容

✅ **代码架构**
- 目录结构分析（不含测试）
- 模块职责和边界划分
- 组件交互关系
- 线程模型和事件处理
- 完整调用链分析

✅ **API 接口**
- N-API JS API 完整清单（22 个 API）
- 参数/返回值/同步异步模式
- C/C++ 实现映射
- 权限和错误码定义
- 内部 IPC 接口定义（28 个接口码）

✅ **构建系统**
- GN 目标完整列表
- 编译配置和特性开关
- 产物清单和输出映射
- 依赖关系图
- 关键宏和常量汇总

✅ **安全分析**
- 权限检查机制
- 输入验证模式
- IPC 边界处理
- 潜在安全风险和修复建议
- 攻击面清单

✅ **运维支持**
- 构建/运行/API 使用常见问题
- 调试技巧和日志分析
- 网络问题排查

### 未覆盖内容

❌ **测试相关**（根据约束，不引用测试内容）
- 单元测试实现
- 模糊测试用例
- 性能测试结果

❌ **商业逻辑**
- 具体运营商协议细节
- 网络优化策略
- 业务规则配置

## 读者对象

| 读者类型 | 推荐阅读路径 | 目标 |
|---------|-------------|------|
| **新加入的开发者** | Overview → Directory → N-API Interface | 快速上手开发 |
| **系统架构师** | Overview → Architecture → Internal API | 理解系统设计 |
| **构建工程师** | GN Targets → Config Flags | 掌握编译配置 |
| **安全审计人员** | N-API Interface → Security Review → AttackSurface | 评估安全风险 |
| **调试/维护人员** | Common Issues → Callgraphs → Architecture | 定位问题根源 |

## 文档结构

```
wiki/
├── README.md                 (本文档)
├── SUMMARY.md                (导航和阅读顺序)
├── 00_Overview.md           (项目概览)
├── 01_Directory_Structure.md (目录结构)
├── 02_Architecture.md       (架构说明)
├── 03_NAPI_Interface.md     (N-API 接口)
├── 04_Internal_API.md       (内部 API)
├── 05_GN_Targets.md       (GN 构建系统)
├── 06_Security_Review.md    (安全评审)
├── 07_Common_Issues.md      (常见问题)
└── appendix/
    ├── Callgraphs.md        (调用链)
    └── Config_Flags.md      (配置标志)
```

## 质量标准

每篇文档必须满足：

✅ **目的明确** - 文档用途和适用范围
✅ **关键结论** - 主要发现总结
✅ **相关跳转** - 与 SUMMARY.md 一致的链接
✅ **证据溯源** - 路径+符号+代码片段
✅ **术语统一** - 与 README 和官方文档一致
✅ **无测试引用** - 不引用测试代码作为业务证据

## 如何使用本文档

### 快速查找

1. 按 **SUMMARY.md** 导航到对应章节
2. 使用 **搜索功能** 查找关键词
3. 查看 **证据溯源** 部分定位源码
4. 跟踪 **相关跳转** 链接深入了解

### 学习建议

1. **先读概览**：从 00_Overview.md 开始，建立全局概念
2. **再学接口**：通过 03_NAPI_Interface.md 学习 API 使用
3. **深入架构**：通过 02_Architecture.md 理解内部实现
4. **实践验证**：结合 07_Common_Issues.md 解决实际问题

## 证据来源

本文档基于以下源代码分析：

### 关键文件

| 类别 | 文件 | 说明 |
|------|------|------|
| 服务入口 | `services/sms/sms_service.cpp` | SMS 服务主入口 |
| N-API 绑定 | `frameworks/js/napi/src/napi_sms.cpp` | JS API 实现 |
| IPC 接口 | `interfaces/innerkits/i_sms_service_interface.h` | IPC 接口定义 |
| GSM 实现 | `services/sms/gsm/*.cpp` | GSM 网络处理 |
| CDMA 实现 | `services/sms/cdma/*.cpp` | CDMA 网络处理 |
| MMS 实现 | `services/mms/*.cpp` | MMS 编解码和网络 |
| 构建配置 | `BUILD.gn`, `smsmms.gni` | GN 构建系统 |

### 关键符号

- 类名：`SmsService`, `SmsInterfaceManager`, `SmsSendManager`, `SmsReceiveManager`
- 接口：`ISmsServiceInterface`, `ISendShortMessageCallback`, `IDeliveryShortMessageCallback`
- IPC 码：`SmsServiceInterfaceCode`, `ImsSmsInterfaceCode`
- 常量：`MAX_ADDRESS_LEN`, `MAX_USER_DATA_LEN`, `PDU_BUFFER_MAX_SIZE`

## 更新日志

| 日期 | 版本 | 变更内容 |
|------|------|-----------|
| 2026-02-07 | 1.1 | 新增 04_Internal_API.md（IPC 接口文档）、07_Common_Issues.md（常见问题）、appendix/Callgraphs.md（调用链）、appendix/Config_Flags.md（配置标志） |
| 2026-02-06 | 1.0 | 初始版本创建 |
