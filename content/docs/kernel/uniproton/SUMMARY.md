# 文档导航

## 快速入口

- [首页](../README.md) - 项目主入口
- [GitHub 仓库](https://gitee.com/openharmony/kernel_uniproton)

## 新手阅读路线

```
1️⃣ 概览 → 2️⃣ 目录结构 → 3️⃣ API 参考 → 4️⃣ 架构设计 → 5️⃣ 构建系统 → 6️⃣ 安全评审 → 7️⃣ 故障排查
```

## 完整文档列表

### 01-基础信息

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [01_Overview.md](./01_Overview.md) | 项目概览 | 定位、能力、特性、运行环境 |
| [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构 | 模块职责、代码组织 |

### 02-接口与架构

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [03_API_Reference.md](./03_API_Reference.md) | API 参考 | C 原生函数、参数、返回值 |
| [04_Architecture.md](./04_Architecture.md) | 架构设计 | 组件图、数据流、时序图 |

### 03-构建与部署

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [05_Build_System.md](./05_Build_System.md) | 构建系统 | GN targets、编译产物、配置 |
| [07_Troubleshooting.md](./07_Troubleshooting.md) | 故障排查 | 常见问题、调试方法 |

### 04-安全与评审

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面分析 | 外部输入、敏感操作、信任边界 |
| [06_Security_Review.md](./06_Security_Review.md) | 安全评审 | 风险分析、修复建议、威胁模型 |

---

## 快速索引

### 按功能查找

| 功能 | API 头文件 | 实现文件 |
|------|-----------|----------|
| 任务管理 | `prt_task.h` | `src/core/kernel/task/` |
| 中断处理 | `prt_hwi.h` | `src/core/kernel/irq/` |
| 信号量 | `prt_sem.h` | `src/core/ipc/sem/` |
| 消息队列 | `prt_queue.h` | `src/core/ipc/queue/` |
| 事件标志 | `prt_event.h` | `src/core/ipc/event/` |
| 定时器 | `prt_timer.h` | `src/core/kernel/timer/` |
| 内存管理 | `prt_mem.h` | `src/mem/` |
| 文件系统 | `prt_fs.h` | `src/fs/` |
| 网络 | - | `src/net/lwip-2.1/` |

### 按架构查找

| 架构 | 代码位置 |
|------|----------|
| ARMv7-M | `src/arch/cpu/armv7-m/` |
| Cortex-M4 | `src/arch/cpu/armv7-m/cortex-m4/` |
| ARMv8 | `src/arch/cpu/armv8/` |
| GIC 驱动 | `src/arch/drv/gic/` |

---

*使用 Ctrl+F (Cmd+F) 快速搜索*
