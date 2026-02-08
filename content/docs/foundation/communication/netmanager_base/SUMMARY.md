# SUMMARY

NetManager Base Wiki - OpenHarmony 网络管理基础组件工程文档

---

## 快速导航

- [Wiki 首页](README.md) - 文档范围与使用指南

## 核心文档

### 项目概览
- [00 项目概览](00_Overview.md) - 项目定位、核心能力、运行环境、关键概念

### 架构与结构
- [01 目录结构](01_Directory_Structure.md) - 模块组织与职责划分
- [02 架构设计](02_Architecture.md) - 组件图、数据流、线程模型、时序图

### API 文档
- [03 N-API JS 接口](03_NAPI_JS_API.md) - JS API 完整清单、调用链、参数校验
- [04 内部 API](04_Inner_API.md) - C++ 接口定义、数据结构、稳定性

### 构建与产物
- [05 GN 构建系统](05_GN_Build.md) - 构建目标、依赖关系、特性开关
- [06 编译产物](06_Build_Artifacts.md) - 输出文件、安装路径、加载关系

### 安全与运维
- [07 安全风险评审](07_Security_Review.md) - 攻击面、信任边界、可被利用点
- [08 常见问题](08_Troubleshooting.md) - 构建/运行/调试问题定位

---

## 新人阅读路线

### 第一天：了解项目
1. 阅读 [Wiki 首页](README.md) - 了解文档范围
2. 阅读 [00 项目概览](00_Overview.md) - 理解项目定位和核心能力
3. 阅读 [01 目录结构](01_Directory_Structure.md) - 熟悉代码组织

### 第二天：理解架构
4. 阅读 [02 架构设计](02_Architecture.md) - 理解组件关系和数据流
5. 浏览 [05 GN 构建系统](05_GN_Build.md) - 了解构建系统

### 第三至五天：深入学习
6. 根据工作方向选择阅读：
   - **JS 接口开发**: [03 N-API JS 接口](03_NAPI_JS_API.md)
   - **C++ 服务开发**: [04 内部 API](04_Inner_API.md)
   - **构建维护**: [05 GN 构建系统](05_GN_Build.md) + [06 编译产物](06_Build_Artifacts.md)
   - **安全审计**: [07 安全风险评审](07_Security_Review.md)

### 持续参考
7. [08 常见问题](08_Troubleshooting.md) - 遇到问题时查阅

---

## 安全研究路线

**目标**: 快速识别攻击面、信任边界、潜在漏洞点
**预计总时长**: 4-6 小时（按深度调整）

### 阶段 1：快速了解攻击面（30 分钟）

**目标**: 10 分钟内识别所有外部输入入口

1. 阅读 [Wiki 首页](README.md) - 了解文档范围
2. 阅读 [安全风险总览](07_Security_Review.md#11-攻击面概览) - 攻击面分类
3. 浏览 [攻击面清单](07_Security_Review.md#12-攻击向量分类) - 了解高风险攻击面

**关键输出**:
- N-API JS 接口攻击面（70+ 个入口点）
- IPC 接口攻击面（跨进程通信）
- 网络数据攻击面（DNS、HTTP、Netlink）
- 配置文件攻击面（路径遍历风险）

---

### 阶段 2：理解信任边界（1 小时）

**目标**: 识别安全域跨越点和权限检查点

1. 阅读 [信任边界图](07_Security_Review.md#5-信任边界) - 理解安全域划分
2. 阅读 [权限检查清单](07_Security_Review.md#21-权限检查) - 定位权限检查点
3. 阅读 [UID/GID 校验](07_Security_Review.md#22-uidgid-校验) - 理解身份验证

**关键输出**:
- 应用 → N-API → 服务 → 内核的安全域跨越
- 权限检查代码位置（`netmanager_base_permission.cpp`）
- IPC 接口权限映射（`net_policy_service_stub.cpp:40-97`）

---

### 阶段 3：深入分析具体风险（2-3 小时）

**目标**: 理解每类风险的可利用性和影响

根据关注点选择阅读：

#### 3.1 N-API 参数安全（重点关注）
- 阅读 [N-API 接口文档](03_NAPI_JS_API.md) - 了解所有 JS API
- 重点关注：参数类型转换、边界检查、空指针检查
- 高风险 API：`setPolicyByUid()`, `bindSocket()`, `setNetQuotaPolicies()`

#### 3.2 权限与鉴权
- 阅读 [权限检查详解](07_Security_Review.md#21-权限检查)
- 分析权限提升路径：普通应用 → 特权操作
- 重点检查：`NetManagerPermission::CheckPermission()` 调用点

#### 3.3 并发安全
- 阅读 [并发安全问题](07_Security_Review.md#3-并发安全) - 竞态条件分析
- 重点关注：网络状态变化、流量统计共享数据
- 检查点：`std::mutex` 使用、`std::shared_ptr` 引用计数

#### 3.4 内存安全
- 阅读 [内存安全问题](07_Security_Review.md#4-内存安全) - 缓冲区溢出、UAF
- 重点关注：N-API 缓冲区操作、IPC 消息处理
- 检查点：`memcpy/memset`、智能指针使用

#### 3.5 逻辑漏洞
- 阅读 [逻辑漏洞分析](07_Security_Review.md#5-逻辑漏洞)
- 重点关注：错误处理、资源耗尽、信息泄露
- 检查点：返回码处理、日志输出、异常流程

---

### 阶段 4：漏洞利用场景研究（1-2 小时）

**目标**: 理解具体攻击路径和利用条件

1. 阅读 [攻击面详细文档](09_Attack_Surfaces.md) - 深入分析每个攻击面
2. 阅读 [利用场景文档](10_Exploit_Scenarios.md) - 具体攻击路径示例
3. 分析每个场景的：
   - 攻击前提（需要什么条件）
   - 利用步骤（如何触发）
   - 代码证据链（从入口到漏洞点的调用链）
   - 影响评估（权限提升、信息泄露等）
   - 防护建议（如何修复）

---

### 阶段 5：安全关键代码快速定位（30 分钟）

**目标**: 快速找到安全相关代码

使用 [安全关键代码索引](07_Security_Review.md#安全关键代码索引):
- 按风险类别查找代码
- 快速跳转到具体漏洞点
- 查看已知安全补丁位置

---

### 阶段 6：实战演练（可选）

**目标**: 通过代码审计实践技能

1. 选择一个高风险 API（如 `setPolicyByUid()`）
2. 追踪调用链：JS API → N-API → IPC → 服务端 → 权限检查
3. 识别潜在漏洞点：
   - 参数验证是否充分？
   - 权限检查是否可绕过？
   - 是否存在竞态条件？
4. 编写漏洞报告：
   - CVE 编号（如已公开）
   - 漏洞描述
   - 触发路径
   - 影响评估
   - 修复建议

---

### 持续参考

- [安全风险总览](07_Security_Review.md) - 完整安全评审
- [攻击面详细文档](09_Attack_Surfaces.md) - 攻击面深度分析
- [利用场景文档](10_Exploit_Scenarios.md) - 具体攻击路径
- [08 常见问题](08_Troubleshooting.md) - 调试相关问题

---

## 主题索引

### 按主题查找

#### 网络连接
- [网络连接管理 - 概览](00_Overview.md#21-功能模块)
- [NetConnService 架构](02_Architecture.md#21-服务层组件)
- [connection N-API](03_NAPI_JS_API.md#4-netconnection-模块)
- [NetConnClient 接口](04_Inner_API.md#2-netconnclient-接口)
- [连接服务构建](05_GN_Build.md#213-连接管理服务)

#### 网络策略
- [网络策略管理 - 概览](00_Overview.md#21-功能模块)
- [NetPolicyService 架构](02_Architecture.md#21-服务层组件)
- [policy N-API](03_NAPI_JS_API.md#5-netpolicy-模块)
- [NetPolicyClient 接口](04_Inner_API.md#3-netpolicyclient-接口)
- [策略服务构建](05_GN_Build.md#214-策略管理服务)

#### 流量统计
- [流量统计管理 - 概览](00_Overview.md#21-功能模块)
- [NetStatsService 架构](02_Architecture.md#21-服务层组件)
- [statistics N-API](03_NAPI_JS_API.md#6-netstatistics-模块)
- [NetStatsClient 接口](04_Inner_API.md#4-netstatsclient-接口)
- [统计服务构建](05_Gn_Build.md#215-统计管理服务)

#### IPC 通信
- [SA 配置](00_Overview.md#42-system-ability-sa)
- [IPC 接口定义](04_Inner_API.md#23-ipc-接口定义)
- [IPC 命令码](04_Inner_API.md#24-ipc-命令码)
- [IPC 安全检查](07_Security_Review.md#22-安全检查点清单)

#### 安全相关
- [安全风险总览](07_Security_Review.md)
- [权限检查](07_Security_Review.md#21-权限检查)
- [输入验证](07_Security_Review.md#23-输入验证)
- [信任边界](07_Security_Review.md#5-信任边界)

---

## 关键代码路径速查

| 组件 | 头文件 | 实现文件 |
|------|--------|----------|
| NetConnService | `services/netconnmanager/include/net_conn_service.h` | `services/netconnmanager/src/net_conn_service.cpp` |
| NetPolicyService | `services/netpolicymanager/include/net_policy_service.h` | `services/netpolicymanager/src/net_policy_service.cpp` |
| NetStatsService | `services/netstatsmanager/include/net_stats_service.h` | `services/netstatsmanager/src/net_stats_service.cpp` |
| NetsysNativeService | `services/netmanagernative/include/netsys/netsys_native_service.h` | `services/netmanagernative/src/netsys_native_service.cpp` |
| NetsysController | `services/netsyscontroller/include/netsys_controller.h` | `services/netsyscontroller/src/netsys_controller.cpp` |
| NetConnClient | `interfaces/innerkits/netconnclient/include/net_conn_client.h` | `interfaces/innerkits/netconnclient/src/net_conn_client.cpp` |
| NetPolicyClient | `interfaces/innerkits/netpolicyclient/include/net_policy_client.h` | `interfaces/innerkits/netpolicyclient/src/net_policy_client.cpp` |
| NetStatsClient | `interfaces/innerkits/netstatsclient/include/net_stats_client.h` | `interfaces/innerkits/netstatsclient/src/net_stats_client.cpp` |

---

## 附录

### 外部参考
- [官方 README](../README.md)
- [官方 README (中文)](../README_zh.md)
- [Bundle 配置](../bundle.json)
- [构建配置](../netmanager_base_config.gni)
- [HiSysEvent 配置](../hisysevent.yaml)

### 相关仓库
- [communication_netmanager_ext](https://gitee.com/openharmony/communication_netmanager_ext) - 网络管理扩展
- [communication_netstack](https://gitee.com/openharmony/communication_netstack) - 网络协议栈

---

## 文档维护

### 更新记录

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-02-07 | v2.0 | 添加安全研究路线，增强安全评审深度 |
| 2025-02-06 | v1.0 | 初始版本，完整工程 Wiki |

### 生成信息
- **生成工具**: OpenHarmony Wiki Generator Agent
- **代码版本**: 基于仓库当前 HEAD
- **最后更新**: 2026-02-07

---

*本文档遵循 Apache 2.0 许可证*
