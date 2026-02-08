# QoS Manager Wiki 导航

> OpenHarmony 线程服务质量（QoS）管理系统文档
> 
> 本文档提供 qos_manager Wiki 的完整导航，帮助快速定位所需信息。

---

## 📖 文档概览

本 Wiki 面向两类核心受众：

| 受众 | 关注重点 | 推荐路线 |
|-----|---------|---------|
| **新人学习者** | 项目定位、API 使用、架构理解 | 🟢 新人学习路线 |
| **安全研究员** | 攻击面、风险评估、利用路径 | 🔒 安全研究路线 |

---

## 🛠 新人学习路线

> 目标：30 分钟内掌握项目核心概念和使用方法

### 路线图

```
README.md        → 项目概览、文档导航
    ↓
00_Overview.md   → 理解 QoS 概念和项目定位
    ↓
02_NDK_API.md    → 掌握 API 使用方法
    ↓
01_Architecture.md → 理解整体架构和数据流
    ↓
04_Build_Targets.md → 了解构建配置和产物
```

### 路线说明

| 阶段 | 文档 | 预计时间 | 收获 |
|-----|-----|---------|-----|
| **入门** | README.md | 2 分钟 | 了解文档结构，找到所需信息 |
| **概念** | 00_Overview.md | 5 分钟 | 理解 QoS 是什么、能做什么 |
| **实践** | 02_NDK_API.md | 10 分钟 | 掌握 API 调用方法，能在应用中使用 |
| **深入** | 01_Architecture.md | 10 分钟 | 理解架构设计，便于调试和扩展 |
| **工程** | 04_Build_Targets.md | 3 分钟 | 了解构建配置，便于集成 |

### 快速上手示例

```c
// 1. 设置线程 QoS 级别为用户交互（最高优先级）
OH_QoS_SetThreadQoS(QOS_USER_INTERACTIVE);

// 2. 执行需要高响应的任务
do_responsive_work();

// 3. 完成后重置 QoS
OH_QoS_ResetThreadQoS();
```

---

## 🔒 安全研究路线

> 目标：快速识别攻击面、评估安全风险

### 路线图

```
README.md        → 文档导航
    ↓
05_Security.md   → 安全风险全景图
    ↓
04_AttackSurface.md → 外部输入和敏感操作清单
    ↓
02_NDK_API.md    → API 参数入口分析
    ↓
01_Architecture.md → 信任边界和数据流
```

### 路线说明

| 阶段 | 文档 | 预计时间 | 收获 |
|-----|-----|---------|-----|
| **导航** | README.md | 2 分钟 | 了解文档结构 |
| **风险** | 05_Security.md | 10 分钟 | 获取风险清单和评估 |
| **攻击面** | 04_AttackSurface.md | 10 分钟 | 识别所有外部输入点 |
| **API** | 02_NDK_API.md | 5 分钟 | 分析 API 参数验证 |
| **架构** | 01_Architecture.md | 8 分钟 | 理解信任边界和数据流 |

### 安全检查清单

- [ ] 识别所有外部输入入口（API 参数、IPC 数据）
- [ ] 定位敏感操作和权限检查点
- [ ] 评估每类风险的可利用性
- [ ] 查看修复建议

---

## 📚 全站导航

### 核心文档

| 编号 | 文档 | 说明 | 受众 | 状态 |
|-----|-----|-----|-----|-----|
| README.md | 文档导航 | 本文档，介绍文档结构和维护方式 | 全部 | ✅ |
| SUMMARY.md | 全站导航 | 双路线导航（本文档） | 全部 | ✅ |
| 00_Overview.md | 项目概览 | 项目定位、能力边界、运行环境 | 新人 | 🔲 |
| 01_Architecture.md | 架构说明 | 组件图、数据流、线程模型 | 全部 | 🔲 |
| 02_NDK_API.md | NDK 接口 | C 语言 API 清单和使用示例 | 新人 | 🔲 |
| 03_Inner_API.md | 内部 API | C++ Inner API 和 IPC 接口 | 开发者 | 🔲 |
| 04_AttackSurface.md | 攻击面分析 | 外部输入、敏感操作、信任边界 | 安全 | 🔲 |
| 05_Security.md | 安全评审 | 风险评估、修复建议 | 安全 | 🔲 |
| 04_Build_Targets.md | 构建配置 | GN targets、编译产物 | 开发者 | 🔲 |

### 附录文档

| 文档 | 说明 | 状态 |
|-----|-----|-----|
| appendix/Callgraphs.md | 关键调用链 | 🔲 |
| appendix/Config_Flags.md | 配置参数 | 🔲 |

**状态说明**：
- ✅ 完成 - 可供阅读
- 🔲 待创建 - 计划中

---

## 🔗 交叉引用

### 文档间关系

```
                    ┌─────────────────┐
                    │   README.md     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
     │00_Overview.md│ │02_NDK_API.md │ │05_Security.md│
     └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
            │                │                │
            │      ┌─────────┴─────────┐      │
            │      │                   │      │
            ▼      ▼                   ▼      ▼
     ┌──────────────┐         ┌──────────────┐
     │01_Architecture│        │04_AttackSurface│
     └──────────────┘         └──────────────┘
```

### 关键路径

- **新人上手路径**：README → 00_Overview → 02_NDK_API
- **安全分析路径**：README → 05_Security → 04_AttackSurface
- **开发集成路径**：README → 02_NDK_API → 04_Build_Targets
- **深度理解路径**：README → 01_Architecture → 03_Inner_API

---

## 📋 快速跳转

### 按功能分类

| 功能 | 相关文档 |
|-----|---------|
| **QoS 控制** | 02_NDK_API.md → OH_QoS_* 函数 |
| **RTG 调度** | 01_Architecture.md → RTG 权限管控 |
| **AI 推理** | 02_NDK_API.md → OH_QoS_Gewu_* 函数 |
| **系统服务** | 01_Architecture.md → SA 1912 |
| **客户端调用** | 03_Inner_API.md → ConcurrentTaskClient |
| **安全审计** | 04_AttackSurface.md, 05_Security.md |

### 按角色分类

| 角色 | 推荐阅读 |
|------|----------|
| **应用开发者** | 00_Overview.md + 02_NDK_API.md |
| **系统开发者** | 01_Architecture.md + 04_Build_Targets.md |
| **安全工程师** | 05_Security.md + 04_AttackSurface.md |
| **测试工程师** | 01_Architecture.md + appendix/Callgraphs.md |

---

## 🔍 关键信息速查

### 系统能力

| 属性 | 值 | 证据 |
|-----|-----|-----|
| **SA ID** | 1912 | `sa_profile/1912.json:5` |
| **进程名** | concurrent_task_service | `sa_profile/1912.json:2` |
| **系统能力** | SystemCapability.Resourceschedule.QoS.Core | `bundle.json:15` |
| **组件版本** | 3.1 | `bundle.json:4` |

### 编译产物

| 库名 | 路径 | 用途 |
|------|------|------|
| `libqos.so` | frameworks/native/ | NDK 接口 |
| `libqos.z.so` | qos/ | QoS 核心 |
| `libconcurrent_task_client.z.so` | frameworks/ | IPC 客户端 |
| `libconcurrentsvc.z.so` | services/ | 系统服务 |
| `libpi_mutex.a` | qos/ | PI 互斥锁静态库 |

### 内核接口

| 接口 | 用途 |
|-----|-----|
| `/proc/thread-self/sched_qos_ctrl` | QoS 控制 |
| `/proc/self/sched_rtg_ctrl` | RTG 调度 |

---

## 📝 更新日志

| 日期 | 版本 | 变更 |
|-----|-----|-----|
| 2026-02-06 | 1.0 | 初始版本，创建 SUMMARY.md |
| 2026-02-06 | 1.1 | 增强双路线导航，添加安全研究路线 |

---

## 💡 使用提示

1. **首次访问**：请先阅读 README.md 了解文档结构
2. **新人学习**：按照「新人学习路线」顺序阅读
3. **安全审计**：按照「安全研究路线」顺序阅读
4. **快速查找**：使用目录或搜索功能定位所需内容
5. **深入理解**：可交叉参考相关文档获取完整信息

---

## 📞 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [qos_manager 源码](https://gitee.com/openharmony/foundation/tree/master/resourceschedule/qos_manager)
- [FFRT 并发框架](https://gitee.com/openharmony/appexecfwk_framework/tree/master/ffrt)
- [frame_aware_sched 帧感知调度](https://gitee.com/openharmony/drivers_peripheral/tree/master/frame_aware_sched)

---

## 版本信息

- **本文档版本**: v1.1
- **适用版本**: qos_manager v3.1
- **最后更新**: 2026-02-06
- **维护者**: OpenHarmony QoS Team
