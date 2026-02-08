# HiLog 文档导航

> 生成时间: 2026-02-07（更新）

---

## 新人学习路线

**适用对象**：需要理解 HiLog 作用、快速上手使用的开发者

### 第一步：快速概览（30 分钟）

阅读顺序：
1. [项目概览](01_Overview.md) - 了解项目定位、核心能力、运行环境
2. [目录结构](02_Directory_Structure.md) - 熟悉代码组织和模块职责

**预期收获**：
- 理解 HiLog 在 OpenHarmony 中的角色
- 知道如何快速定位相关代码

---

### 第二步：架构理解（1 小时）

阅读顺序：
1. [架构说明](03_Architecture.md) - 理解组件交互、数据流、线程模型
2. [N-API 接口](04_NAPI_Interface.md) - 了解 JS 调用链路
3. [内部 API](05_Internal_API.md) - 了解 Native API 和依赖

**预期收获**：
- 理解日志从应用到落盘的完整流程
- 知道 hilogd 的线程模型和并发机制
- 了解 Unix Domain Socket 通信机制

---

### 第三步：构建与调试（30 分钟）

阅读顺序：
1. [GN Targets](06_GN_Targets.md) - 理解构建系统
2. [编译产物](07_Build_Artifacts.md) - 了解生成文件和安装位置
3. [常见问题](09_Troubleshooting.md) - 学习调试技巧

**预期收获**：
- 知道如何修改和编译 HiLog
- 了解各 targets 之间的关系
- 掌握日志未打印、日志丢失等问题定位方法

---

## 安全研究路线

**适用对象**：需要评估安全风险、识别攻击面的安全研究员

### 第一步：攻击面识别（30 分钟）

阅读顺序：
1. [安全分析 - 攻击面](08_Security_Analysis.md#攻击面分析) - 识别所有外部输入入口
2. [目录结构](02_Directory_Structure.md) - 定位关键代码位置
3. [架构说明](03_Architecture.md) - 理解信任边界和数据流

**预期收获**：
- 识别所有外部输入点（N-API、Socket、文件、命令行）
- 理解数据流向和信任边界
- 定位敏感操作（文件写入、缓冲区管理）

---

### 第二步：漏洞分析（1 小时）

阅读顺序：
1. [安全分析 - 可被利用点](08_Security_Analysis.md#可被利用点) - 深入分析已知漏洞
2. [N-API 接口](04_NAPI_Interface.md) - 了解 JS 层输入处理
3. [调用链图](appendix/Callgraphs.md) - 追踪从输入到敏感操作的路径

**预期收获**：
- 了解输入验证缺陷、内存安全问题
- 掌握权限绕过、日志注入等漏洞
- 能够构造利用路径

---

### 第三步：修复建议（30 分钟）

阅读顺序：
1. [安全分析 - 修复建议](08_Security_Analysis.md#修复建议) - 查看优先级修复列表
2. [配置标志](appendix/Config_Flags.md) - 了解安全相关配置
3. [架构说明 - 安全机制](03_Architecture.md#安全机制) - 理解现有防护

**预期收获**：
- 知道如何修复已知安全风险
- 了解哪些 feature flags 影响安全
- 理解 UID 检查、流控等防护机制

---

## 文档结构

### 核心文档

| 文档 | 用途 | 预计阅读时间 |
|------|------|-------------|
| [01_Overview.md](01_Overview.md) | 项目定位、核心能力、关键概念 | 30 分钟 |
| [02_Directory_Structure.md](02_Directory_Structure.md) | 目录组织、模块职责 | 20 分钟 |
| [03_Architecture.md](03_Architecture.md) | 组件图、数据流、线程模型 | 60 分钟 |
| [04_NAPI_Interface.md](04_NAPI_Interface.md) | JS API 清单、调用链、参数校验 | 45 分钟 |
| [05_Internal_API.md](05_Internal_API.md) | 模块接口、依赖方向、稳定性 | 40 分钟 |
| [06_GN_Targets.md](06_GN_Targets.md) | targets 列表、类型、依赖、产物 | 30 分钟 |
| [07_Build_Artifacts.md](07_Build_Artifacts.md) | 编译产物、安装路径、运行时加载 | 20 分钟 |
| [08_Security_Analysis.md](08_Security_Analysis.md) | 攻击面、可被利用点、修复建议 | 60 分钟 |
| [09_Troubleshooting.md](09_Troubleshooting.md) | 常见问题、定位路径 | 30 分钟 |

### 附录文档

| 文档 | 用途 | 预计阅读时间 |
|------|------|-------------|
| [Callgraphs.md](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） | 30 分钟 |
| [Config_Flags.md](appendix/Config_Flags.md) | 关键宏、feature flags | 20 分钟 |

---

## 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| 日志类型 | Log Type | LOG_APP、LOG_CORE、LOG_INIT、LOG_KMSG 等 |
| 日志级别 | Log Level | DEBUG、INFO、WARN、ERROR、FATAL |
| Domain | Domain | 日志域标识，用于区分子系统/模块 |
| Tag | Tag | 日志标签，用于标识类、文件或服务 |
| 流控 | Flow Control | 防止日志流量过大的机制 |
| 落盘 | Persist | 将缓冲区日志写入文件 |
| Unix Domain Socket | UDS | 用于进程间通信的本地 socket |
| 环形缓冲区 | Ring Buffer | 循环使用的固定大小缓冲区 |
| SO_PASSCRED | Socket 凭证传递 | 传递发送方 PID/UID 的 socket 选项 |

---

## 快速参考

### 常用命令

| 需求 | 命令 |
|------|------|
| 查看日志 | `hilog` 或 `hilog -t app` |
| 过滤 tag | `hilog -T MyTag` |
| 过滤 PID | `hilog -P 1234` |
| 设置日志级别 | `hilog -b D` |
| 清空 buffer | `hilog -r` |
| 查询 buffer 大小 | `hilog -g` |
| 设置 buffer 大小 | `hilog -G 8M` |
| 启动日志落盘 | `hilog -w start -l 8M -n 100` |

### 关键代码位置

| 功能 | 文件路径 |
|------|---------|
| N-API 注册 | `interfaces/js/kits/napi/src/hilog/module.cpp` |
| hilogd 主入口 | `services/hilogd/main.cpp` |
| 环形缓冲区 | `services/hilogd/log_buffer.cpp` |
| 日志收集器 | `services/hilogd/log_collector.cpp` |
| Socket 通信 | `frameworks/libhilog/socket/` |
| 服务控制 | `services/hilogd/service_controller.cpp` |
| 日志落盘 | `services/hilogd/log_persister.cpp` |

---

## 注意事项

1. **测试代码排除**: 本文档不引用 test/、unittest/、fuzz/ 目录内容
2. **证据要求**: 所有关键结论都标注了证据源（文件路径+行号/符号）
3. **版本对应**: 本文档基于 HiLog 3.1 版本生成，如版本有差异请注意
4. **平台差异**: 部分实现存在平台差异（ohos、windows、mac、linux）

---

## 贡献与反馈

如发现文档错误或有改进建议，请：
1. 检查 `wiki/_work/NOTES.md` 中的证据索引
2. 验证代码与文档的一致性
3. 提交 PR 或联系维护者更新文档
