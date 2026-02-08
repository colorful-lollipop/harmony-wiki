# Call Manager 文档导航

**项目**: OpenHarmony telephony_call_manager
**系统能力**: SystemCapability.Telephony.CallManager
**SA ID**: 4005

---

## 快速开始

**我是新人，想快速了解项目**：→ [新人学习路线](#新人学习路线)

**我是安全研究员，想进行安全审计**：→ [安全研究路线](#安全研究路线)

**我是开发者，想开始集成开发**：→ [开发者路线](#开发者路线)

---

## 文档索引

### 基础文档
- [README](README.md) - 项目概览、文档导航、更新说明
- [项目评估](wiki/_work/ASSESSMENT.md) - 项目类型、受众分析、文档策略

### 新人学习必备
1. [项目概览](wiki/01_Overview.md) ⏳ 待编写
   - 一句话定义、能力边界、运行环境
   - 快速开始示例、通话类型支持

2. [架构与数据流](wiki/02_Architecture.md) ⏳ 待编写
   - 组件图、数据流图
   - 线程模型、关键时序

3. [目录结构与代码地图](wiki/03_CodeMap.md) ⏳ 待编写
   - 目录职责说明、核心文件定位
   - 代码导航图

4. [对外接口文档](wiki/04_Interface.md) ⏳ 待编写
   - N-API 清单表、IPC 接口、配置文件

### 安全研究必备
5. [攻击面分析](wiki/05_AttackSurface.md) ⏳ 待编写
   - 外部输入清单、敏感操作清单
   - 信任边界图、权限检查点

6. [安全风险评估](wiki/06_SecurityReview.md) ⏳ 待编写
   - 5+ 类常见风险分析
   - 证据、触发路径、影响评估、修复建议

### 工程参考
7. [构建与产物](wiki/07_Build.md) ⏳ 待编写
   - GN 目标清单、编译产物
   - Feature 开关、条件编译宏

8. [内部实现细节](wiki/08_Internals.md) ⏳ 待编写
   - 核心类职责、内部 API 契约
   - 资源生命周期

### 工作文档
- [项目评估](wiki/_work/ASSESSMENT.md) ✅ 已完成
- [代码证据汇总](wiki/_work/NOTES.md) ✅ 已完成
- [任务进度追踪](wiki/_work/PLAN.md) ✅ 已完成

---

## 推荐阅读路径

### 新人学习路线

**目标**: 在 30 分钟内理解 Call Manager 的基本架构和使用方式

| 步骤 | 文档 | 预计时间 | 关键收获 |
|------|------|----------|----------|
| 1. 了解项目定位 | [01_Overview.md](wiki/01_Overview.md) | 5 分钟 | Call Manager 解决什么问题、支持哪些通话类型 |
| 2. 理解整体架构 | [02_Architecture.md](wiki/02_Architecture.md) | 15 分钟 | 六大子系统如何协作、数据如何流转 |
| 3. 定位核心代码 | [03_CodeMap.md](wiki/03_CodeMap.md) | 5 分钟 | 快速找到拨号/接听等功能的代码位置 |
| 4. 学习 API 使用 | [04_Interface.md](wiki/04_Interface.md) | 15 分钟 | 如何调用 JS API 进行拨号、接听、挂断 |
| 5. 了解构建配置 | [07_Build.md](wiki/07_Build.md) | 10 分钟 | 如何编译、Feature 开关如何配置 |
| **总计** | - | **50 分钟** | 快速入门 |

### 安全研究路线

**目标**: 在 60 分钟内识别所有攻击面和潜在风险

| 步骤 | 文档 | 预计时间 | 关键收获 |
|------|------|----------|----------|
| 1. 了解项目架构 | [02_Architecture.md](wiki/02_Architecture.md) | 10 分钟 | 理解系统边界和信任关系 |
| 2. 识别攻击面 | [05_AttackSurface.md](wiki/05_AttackSurface.md) | 10 分钟 | 所有外部输入入口、敏感操作点 |
| 3. 分析输入验证 | [06_SecurityReview.md](wiki/06_SecurityReview.md) | 15 分钟 | 电话号码、callId、IPC 参数的验证漏洞 |
| 4. 分析内存安全 | [06_SecurityReview.md](wiki/06_SecurityReview.md) | 10 分钟 | 缓冲区溢出、UAF、双重释放风险 |
| 5. 分析权限控制 | [05_AttackSurface.md](wiki/05_AttackSurface.md) + [06_SecurityReview.md](wiki/06_SecurityReview.md) | 10 分钟 | 权限检查绕过可能性 |
| 6. 分析并发安全 | [06_SecurityReview.md](wiki/06_SecurityReview.md) | 5 分钟 | 竞态条件、TOCTOU 风险 |
| **总计** | - | **60 分钟** | 全面安全审计 |

### 开发者路线

**目标**: 在 90 分钟内掌握集成开发和问题排查

| 步骤 | 文档 | 预计时间 | 关键收获 |
|------|------|----------|----------|
| 1. 快速了解项目 | [01_Overview.md](wiki/01_Overview.md) | 10 分钟 | 项目定位、能力边界 |
| 2. 理解架构 | [02_Architecture.md](wiki/02_Architecture.md) | 20 分钟 | 数据流、状态机、线程模型 |
| 3. 学习 API | [04_Interface.md](wiki/04_Interface.md) | 20 分钟 | 如何正确调用 N-API |
| 4. 查看内部实现 | [08_Internals.md](wiki/08_Internals.md) | 15 分钟 | CallBase 虚函数、资源生命周期 |
| 5. 了解构建 | [07_Build.md](wiki/07_Build.md) | 15 分钟 | GN targets、Feature 开关、依赖关系 |
| 6. 安全注意事项 | [05_AttackSurface.md](wiki/05_AttackSurface.md) + [06_SecurityReview.md](wiki/06_SecurityReview.md) | 10 分钟 | 常见陷阱和最佳实践 |
| **总计** | - | **90 分钟** | 开发就绪 |

---

## 核心概念速查表

### 通话类型
| 类型 | 全称 | 说明 | 实现文件 |
|------|------|------|----------|
| CS | Circuit Switched | 电路交换（传统蜂窝通话） | services/call/src/cs_call.cpp |
| IMS | IP Multimedia Subsystem | IP多媒体（VoLTE/VoNR 高清通话） | services/call/src/ims_call.cpp |
| OTT | Over-The-Top | 网络应用（如微信、钉钉） | services/call/src/ott_call.cpp |
| VoIP | Voice over IP | 内置 VoIP 支持 | services/call/src/voip_call.cpp |

### 通话状态
| 状态 | 说明 |
|------|------|
| IDLE | 空闲，无通话 |
| INCOMING | 来电中 |
| DIALING | 拨号中 |
| ALERTING | 响铃中 |
| ACTIVE | 通话中 |
| HOLDING | 保持中 |
| DISCONNECTING | 挂断中 |
| DISCONNECTED | 已挂断 |

### 核心类
| 类 | 职责 |
|------|------|
| CallManagerService | 系统能力服务入口（SA 4005），处理 IPC 请求 |
| CallControlManager | 通话控制核心，处理下行操作（拨号/接听/挂断） |
| CallStatusManager | 通话状态管理，处理上行状态上报 |
| CallObjectManager | 通话对象管理器，负责 callId 分配和生命周期 |
| CallBase | 通话基类，抽象所有通话类型的通用接口 |
| AudioControlManager | 音频资源管理，处理铃声、音频设备切换 |
| VideoControlManager | 视频资源管理，处理摄像头、视频窗口 |
| BluetoothCallManager | 蓝牙通话管理，处理 HFP 协议交互 |

### 权限
| 权限 | 用途 |
|------|------|
| ohos.permission.PLACE_CALL | 拨打电话 |
| ohos.permission.ANSWER_CALL | 接听/挂断电话 |
| ohos.permission.GET_TELEPHONY_STATE | 查询通话状态 |
| ohos.permission.SET_TELEPHONY_STATE | 设置通话状态 |
| ohos.permission.READ_CALL_LOG | 读取通话记录 |
| ohos.permission.WRITE_CALL_LOG | 写入通话记录 |

---

## 快速链接

### 新人快速定位
- [项目定位](wiki/01_Overview.md#项目定位)
- [能做什么](wiki/01_Overview.md#能力边界)
- [最小示例](wiki/01_Overview.md#快速开始)
- [拨号入口代码](wiki/03_CodeMap.md#拨号入口)
- [接听入口代码](wiki/03_CodeMap.md#接听入口)

### 安全快速定位
- [外部输入入口](wiki/05_AttackSurface.md#外部输入入口)
- [敏感操作清单](wiki/05_AttackSurface.md#敏感操作清单)
- [输入验证风险](wiki/06_SecurityReview.md#输入验证缺陷)
- [权限检查点](wiki/05_AttackSurface.md#权限检查点)
- [内存安全风险](wiki/06_SecurityReview.md#内存安全问题)

### 开发者快速定位
- [API 清单](wiki/04_Interface.md#n-api-清单表)
- [GN targets](wiki/07_Build.md#gn-目标清单)
- [Feature 开关](wiki/07_Build.md#feature-开关)
- [错误码说明](wiki/04_Interface.md#错误码映射表)
- [CallBase 虚函数](wiki/08_Internals.md#callbase-虚函数)

---

## 文档维护

### 更新触发
- 新增 N-API 接口 → 更新 [04_Interface.md](wiki/04_Interface.md)
- 新增 GN target → 更新 [07_Build.md](wiki/07_Build.md)
- 安全相关修改 → 更新 [05_AttackSurface.md](wiki/05_AttackSurface.md) 和 [06_SecurityReview.md](wiki/06_SecurityReview.md)
- 架构重构 → 更新 [02_Architecture.md](wiki/02_Architecture.md)

### 贡献指南
所有文档均通过以下方式维护：
1. 阅读现有文档，理解结构和风格
2. 根据代码变化更新内容
3. 确保每个技术点都有代码证据
4. 更新 `wiki/_work/NOTES.md` 记录新的证据
5. 更新本文档的文档索引和快速链接

---

**最后更新**: 2026-02-07
**文档版本**: 1.0
**维护者**: OpenHarmony Wiki 生成 Agent
