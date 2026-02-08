# 文档导航

**文档版本**: v1.0  
**更新日期**: 2025-02-07  
**模块版本**: 3.2

---

## 文档概述

本文档为 OpenHarmony `distributed_input`（分布式输入）模块 Wiki 的全局导航索引。文档分为两条阅读路线，分别面向**新人学习者**和**安全研究员**。

### 模块信息

| 属性 | 值 |
|------|-----|
| **模块名称** | distributed_input |
| **模块描述** | 分布式输入部件 |
| **子系统** | distributedhardware |
| **版本** | 3.2 |
| **SA ID** | Source (4809), Sink (4810) |
| **进程名** | dinput |

---

## 快速索引

### 核心文档

| 文档 | 说明 | 优先级 |
|------|------|-------|
| [README](README.md) | 项目导航和快速链接 | P0 |
| [概览](00_Overview.md) | 项目介绍和核心概念 | P0 |
| [项目定位](01_Project_Positioning.md) | 功能边界和核心能力 | P1 |
| [目录结构](02_Directory_Structure.md) | 模块组织和代码地图 | P1 |
| [架构设计](03_Architecture.md) | 组件关系和数据流 | P1 |

### API 与接口

| 文档 | 说明 | 优先级 |
|------|------|-------|
| [公共 API](04_Public_API.md) | C++ Inner SDK 接口 | P0 |
| [内部 API](05_Inner_API.md) | 模块间接口和依赖 | P2 |
| [攻击面分析](05_Attack_Surface.md) | 输入入口和信任边界 | P0 |
| [安全ecurity_Review.md) | 安全机制评审](08_S和风险评估 | P0 |

### 构建与部署

| 文档 | 说明 | 优先级 |
|------|------|-------|
| [GN 目标](06_GN_Targets.md) | 编译目标和依赖关系 | P2 |
| [构建产物](07_Build_Artifacts.md) | 库文件和安装路径 | P2 |

---

## 新人学习路线

> 建议阅读顺序：按序号顺序阅读

### 第一阶段：基础认知 (1-3天)

| 顺序 | 文档 | 预计时间 | 关键内容 |
|------|------|---------|---------|
| 1 | [概览](00_Overview.md) | 30分钟 | 项目定位、Source/Sink 角色 |
| 2 | [项目定位](01_Project_Positioning.md) | 30分钟 | 功能边界、能力范围 |
| 3 | [目录结构](02_Directory_Structure.md) | 1小时 | 模块划分、代码导航 |

### 第二阶段：深入理解 (2-4天)

| 顺序 | 文档 | 预计时间 | 关键内容 |
|------|------|---------|---------|
| 4 | [架构设计](03_Architecture.md) | 2小时 | 组件关系、数据流 |
| 5 | [公共 API](04_Public_API.md) | 2小时 | API 使用方法 |
| 6 | [内部 API](05_Inner_API.md) | 1小时 | 模块间调用关系 |

### 第三阶段：实践应用 (1-2天)

| 顺序 | 文档 | 预计时间 | 关键内容 |
|------|------|---------|---------|
| 7 | [GN 目标](06_GN_Targets.md) | 1小时 | 构建配置 |
| 8 | [构建产物](07_Build_Artifacts.md) | 1小时 | 运行时部署 |

### 学习成果检验

完成学习后，你应该能够：

- [ ] 解释 Source 和 Sink 设备的区别
- [ ] 描述分布式输入事件的数据流
- [ ] 定位主要代码文件的位置
- [ ] 理解 InnerKit API 的使用方法
- [ ] 说明模块间的依赖关系

---

## 安全研究路线

> 建议阅读顺序：按需求选择性阅读

### 第一阶段：快速上手 (1-2小时)

| 顺序 | 文档 | 预计时间 | 关键内容 |
|------|------|---------|---------|
| 1 | [攻击面分析](05_Attack_Surface.md) | 1小时 | 外部输入、敏感操作 |
| 2 | [安全评审](08_Security_Review.md) | 1小时 | 安全机制、风险清单 |

### 第二阶段：深度分析 (2-4小时)

| 顺序 | 文档 | 预计时间 | 关键内容 |
|------|------|---------|---------|
| 3 | [架构设计](03_Architecture.md) | 2小时 | 数据流、信任边界 |
| 4 | [公共 API](04_Public_API.md) | 1小时 | 接口权限要求 |
| 5 | [目录结构](02_Directory_Structure.md) | 1小时 | 代码布局 |

### 第三阶段：代码审计 (按需)

| 场景 | 目标文档 | 关键文件 |
|------|---------|---------|
| IPC 权限校验审计 | [公共 API](04_Public_API.md) | `distributed_input_*_stub.cpp` |
| 传输层安全审计 | [攻击面分析](05_Attack_Surface.md) | `softbus_permission_check.cpp` |
| 输入验证审计 | [安全评审](08_Security_Review.md) | `input_check_param.cpp` |
| 虚拟设备审计 | [安全评审](08_Security_Review.md) | `virtual_device.cpp` |

### 安全审计检查清单

- [ ] 识别所有外部输入入口（IPC、跨设备、配置）
- [ ] 验证权限校验机制的有效性
- [ ] 检查输入验证的完整性
- [ ] 分析传输层安全措施
- [ ] 评估敏感操作的风险
- [ ] 确认日志监控的覆盖范围

---

## 文档交叉引用

### 概念关联

| 概念 | 相关文档 | 关联章节 |
|------|---------|---------|
| Source 设备 | [概览](00_Overview.md), [架构设计](03_Architecture.md) | 全部 |
| Sink 设备 | [概览](00_Overview.md), [架构设计](03_Architecture.md) | 全部 |
| 虚拟设备 | [架构设计](03_Architecture.md), [安全评审](08_Security_Review.md) | 架构:4.3, 安全:R3 |
| SoftBus 传输 | [架构设计](03_Architecture.md), [攻击面分析](05_Attack_Surface.md) | 架构:4.2, 攻击面:2.2 |
| AccessToken | [公共 API](04_Public_API.md), [安全评审](08_Security_Review.md) | API:2.3, 安全:4.1 |
| 白名单过滤 | [项目定位](01_Project_Positioning.md), [安全评审](08_Security_Review.md) | 定位:2.3, 安全:R2 |

### 文件定位

| 文件类型 | 路径模式 | 相关文档 |
|---------|---------|---------|
| IPC 存根 | `interfaces/ipc/src/*_stub.cpp` | [公共 API](04_Public_API.md) |
| IPC 代理 | `interfaces/ipc/src/*_proxy.cpp` | [公共 API](04_Public_API.md) |
| 服务实现 | `services/*/src/*.cpp` | [内部 API](05_Inner_API.md) |
| 头文件 | `*/include/*.h` | [目录结构](02_Directory_Structure.md) |
| 构建配置 | `*.gni`, `BUILD.gn` | [GN 目标](06_GN_Targets.md) |

---

## API 快速参考

### InnerKit 主要接口

| 接口 | 说明 | 头文件 |
|------|------|-------|
| `PrepareRemoteInput()` | 准备远程输入 | `distributed_input_kit.h` |
| `StartRemoteInput()` | 启动远程输入 | `distributed_input_kit.h` |
| `StopRemoteInput()` | 停止远程输入 | `distributed_input_kit.h` |
| `RegisterSimulationEventListener()` | 注册事件监听 | `distributed_input_kit.h` |

### SA 标识

| SA | ID | 库文件 | 说明 |
|----|-----|-------|------|
| Source | 4809 | `libdinput_source.z.so` | 主控端服务 |
| Sink | 4810 | `libdinput_sink.z.so` | 被控端服务 |

### 权限要求

| 权限 | 用途 | 保护的操作 |
|------|------|-----------|
| `ENABLE_DISTRIBUTED_HARDWARE` | 启用分布式硬件 | Init、Register |
| `ACCESS_DISTRIBUTED_HARDWARE` | 访问分布式硬件 | Prepare、Start、Stop |

---

## 版本信息

### 当前版本

| 属性 | 值 |
|------|-----|
| **文档版本** | v1.0 |
| **生成日期** | 2025-02-07 |
| **模块版本** | 3.2 |

### 更新日志

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2025-02-07 | 初始版本 |

---

## 反馈与贡献

### 文档问题反馈

如发现文档错误或遗漏，请：

1. 检查相关代码文件确认问题
2. 记录问题现象和证据
3. 提交 Issue 到代码仓库

### 贡献指南

欢迎贡献文档内容：

1. 遵循现有文档风格
2. 提供代码证据
3. 保持语言一致性

---

**文档版本**: v1.0  
**维护者**: OpenHarmony Wiki Generator
