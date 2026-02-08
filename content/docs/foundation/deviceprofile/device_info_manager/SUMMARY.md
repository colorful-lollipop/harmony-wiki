# SUMMARY - DeviceProfile Wiki 导航

**快速选择你的阅读路线：**

---

## 🎓 新人学习路线

适合：初次接触 DeviceProfile 的开发者
目标：理解项目、掌握 API、能够独立开发

| 序号 | 文档 | 预计时间 | 学习目标 |
|------|------|----------|----------|
| 1 | [01_Overview.md](01_Overview.md) | 5 min | 理解项目定位和核心能力 |
| 2 | [02_Architecture.md](02_Architecture.md) | 10 min | 理解架构和数据流向 |
| 3 | [03_CodeMap.md](03_CodeMap.md) | 5 min | 快速定位代码文件 |
| 4 | [04_Interface.md](04_Interface.md) | 15 min | 掌握 IPC 接口使用 |
| 5 | [07_Build.md](07_Build.md) | 5 min | 了解构建和产物 |
| 6 | [08_Internals.md](08_Internals.md) | 10 min | 深入内部实现（可选）|

**总计**: 30-50 分钟

---

## 🔒 安全研究路线

适合：安全研究员、审计人员
目标：识别攻击面、发现安全漏洞

| 序号 | 文档 | 预计时间 | 学习目标 |
|------|------|----------|----------|
| 1 | [05_AttackSurface.md](05_AttackSurface.md) | 10 min | 识别所有攻击入口 |
| 2 | [06_SecurityReview.md](06_SecurityReview.md) | 15 min | 分析具体安全风险 |
| 3 | [02_Architecture.md](02_Architecture.md) | 5 min | 理解信任边界 |
| 4 | [04_Interface.md](04_Interface.md) | 5 min | 分析接口参数 |
| 5 | [08_Internals.md](08_Internals.md) | 10 min | 检查内部实现细节 |

**总计**: 20-45 分钟

---

## 📚 完整文档列表

### 核心文档
- [README.md](README.md) - 文档中心首页
- [01_Overview.md](01_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 架构设计
- [03_CodeMap.md](03_CodeMap.md) - 代码地图
- [04_Interface.md](04_Interface.md) - 接口文档
- [05_AttackSurface.md](05_AttackSurface.md) - 攻击面分析
- [06_SecurityReview.md](06_SecurityReview.md) - 安全评估
- [07_Build.md](07_Build.md) - 构建配置
- [08_Internals.md](08_Internals.md) - 内部实现

### 工作文档
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估
- [_work/NOTES.md](_work/NOTES.md) - 代码证据汇总
- [_work/PLAN.md](_work/PLAN.md) - 任务计划

---

## 🔍 快速查找

### 按主题查找

| 主题 | 相关文档 |
|------|----------|
| **项目定位** | [01_Overview.md](01_Overview.md) |
| **架构图** | [02_Architecture.md](02_Architecture.md) |
| **代码位置** | [03_CodeMap.md](03_CodeMap.md) |
| **API 列表** | [04_Interface.md](04_Interface.md) |
| **安全风险** | [05_AttackSurface.md](05_AttackSurface.md), [06_SecurityReview.md](06_SecurityReview.md) |
| **编译构建** | [07_Build.md](07_Build.md) |
| **实现细节** | [08_Internals.md](08_Internals.md) |

### 关键代码位置速查

```
SA 服务入口: services/core/include/distributed_device_profile_service_new.h:42
IPC 客户端:   interfaces/innerkits/core/include/distributed_device_profile_client.h:46
权限检查:     services/core/src/permissionmanager/permission_manager.cpp:218
错误码定义:   common/include/constants/distributed_device_profile_errors.h:21
IPC 命令码:   common/include/interfaces/dp_ipc_interface_code.h
SA 配置:      sa_profile/6001.json
权限配置:     permission/permission.json
```

---

## 📝 术语表

| 术语 | 说明 |
|------|------|
| **SA** | System Ability，系统能力，OpenHarmony 的系统服务框架 |
| **SAID** | System Ability ID，系统能力标识符（DeviceProfile 为 6001）|
| **Innerkits** | 系统内部接口，供其他系统服务调用 |
| **IPC** | Inter-Process Communication，Binder 进程间通信 |
| **Profile** | 描述设备能力的数据结构 |
| **KV Store** | Key-Value 存储，分布式数据管理子系统 |
| **RDB** | Relational Database，关系型数据库 |
| **Softbus** | 分布式软总线，提供设备间通信能力 |

---

**导航提示**: 使用浏览器搜索功能（Ctrl+F）在当前页面快速查找关键词。
