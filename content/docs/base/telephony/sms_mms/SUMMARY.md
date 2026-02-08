# 短彩信模块 (sms_mms) 文档导航

本文档提供 OpenHarmony 短彩信模块 (telephony/sms_mms) 的完整技术文档导航。

## 文档生成信息

- **生成时间**: 2026-02-06
- **仓库**: openharmony/telephony_sms_mms
- **版本**: 4.0
- **覆盖范围**: 代码架构、API 接口、构建系统、安全分析

## 新人阅读顺序（推荐）

如果您是第一次接触短彩信模块，建议按以下顺序阅读：

1. **[项目概览](00_Overview.md)** - 了解项目定位、边界、核心能力
2. **[目录结构](01_Directory_Structure.md)** - 熟悉代码组织方式和模块职责
3. **[架构说明](02_Architecture.md)** - 理解组件交互、数据流、线程模型
4. **[N-API 接口](03_NAPI_Interface.md)** - 学习对外 JS API 使用方法
5. **[内部 API](04_Internal_API.md)** - 了解内部模块接口和依赖关系
6. **[GN 目标与编译](05_GN_Targets.md)** - 掌握构建系统和产物
7. **[安全风险评审](06_Security_Review.md)** - 了解安全机制和风险点
8. **[常见问题](07_Common_Issues.md)** - 了解开发调试常见问题

### 快速参考路线

| 目标 | 推荐阅读 |
|------|----------|
| **快速上手开发** | Overview → Directory → N-API Interface |
| **理解系统设计** | Overview → Architecture → Internal API |
| **掌握编译配置** | GN Targets → Config_Flags |
| **安全审计** | N-API Interface → Security Review → AttackSurface |
| **调试问题** | Common_Issues → Callgraphs → Architecture |

## 文档索引

### 核心文档

| 文档 | 描述 | 读者对象 |
|------|------|----------|
| [00_Overview.md](00_Overview.md) | 项目定位、边界、核心能力、运行环境、关键概念 | 所有人 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构与模块职责（不含测试） | 开发者 |
| [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型、关键时序 | 架构师、高级开发者 |
| [03_NAPI_Interface.md](03_NAPI_Interface.md) | 对外 N-API（JS API 面）、导出符号、权限/参数/错误码 | 应用开发者 |
| [04_Internal_API.md](04_Internal_API.md) | 内部 API、模块接口、依赖方向、稳定性、可替换点 | 系统开发者 |
| [05_GN_Targets.md](05_GN_Targets.md) | targets 列表、类型、依赖、产物、开关 | 构建工程师 |
| [06_Security_Review.md](06_Security_Review.md) | 攻击面、信任边界、可被利用点、修复建议 | 安全审计人员 |
| [07_Common_Issues.md](07_Common_Issues.md) | 构建/运行/API/权限常见问题 | 开发者 |

### 附录

| 文档 | 描述 | 读者对象 |
|------|------|----------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） | 调试人员 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏/feature flags | 配置人员 |

### 附录

| 文档 | 描述 | 读者对象 |
|------|------|----------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） | 调试人员 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏/feature flags | 配置人员 |

## 关键概念速查

### 系统能力
- **SA ID**: 4008 (TELEPHONY_SMS_MMS_SYS_ABILITY_ID)
- **进程名**: telephony
- **系统能力**: SystemCapability.Telephony.SmsMms

### 核心模块
- **SmsInterfaceManager**: 接口管理类，负责创建和管理发送/接收/Misc 管理器
- **SmsSendManager**: 发送管理类，根据网络制式调度 GSM/CDMA/IMS 发送
- **SmsReceiveManager**: 接收管理类，监听 RIL 新短信并分发处理
- **SmsMiscManager**: 杂项管理类，处理 SIM 操作、小区广播、SMSC 配置
- **MmsSendManager / MmsReceiveManager**: MMS 发送/接收管理器（可选特性）

### 网络制式
- **GSM**: GSM 网络短信处理
- **CDMA**: CDMA 网络短信处理
- **IMS**: IP 多媒体子系统短信（通过 IMS 服务）

### JS API 命名空间
- `@ohos.telephony.sms` - SMS/MMS 所有 API

## 证据溯源

本文档中的所有关键结论均基于以下证据源：

- **源代码路径**: 所有引用均包含完整文件路径（含行号）
- **关键符号**: 函数名、类名、宏定义、接口码
- **代码片段**: 最小必要代码片段或调用链描述
- **构建文件**: BUILD.gn、bundle.json、.gni 配置

## 维护说明

### 如何随代码更新文档

1. **定期同步**：代码变更后，同步更新对应文档章节
2. **证据验证**：所有新增内容必须有代码证据支持
3. **术语一致**：保持与 README_zh.md 和官方文档术语一致
4. **链接检查**：修改文件路径或删除文件后，更新 SUMMARY.md 和内部链接

### 贡献指南

本文档仅记录事实，不包含个人观点。更新时请遵循：

1. 提供代码证据（路径+行号）
2. 不引用测试相关内容
3. 保持中英文术语统一
4. 确保所有链接有效

## 相关资源

- **官方 API 文档**: https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-telephony-kit/js-apis-sms.md
- **相关仓**: [telephony_core_service](https://gitee.com/openharmony/telephony_core_service)
- **OpenHarmony 文档**: https://gitee.com/openharmony/docs

## 版本历史

| 版本 | 日期 | 变更内容 |
|------|------|-----------|
| 1.1 | 2026-02-07 | 新增 04_Internal_API.md、07_Common_Issues.md、Callgraphs.md、Config_Flags.md |
| 1.0 | 2026-02-06 | 初始版本，完整覆盖架构、API、构建、安全 |
