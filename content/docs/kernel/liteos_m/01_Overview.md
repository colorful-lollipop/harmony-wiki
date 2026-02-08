# 项目概览 - 01_Overview

## 项目定位

**OpenHarmony LiteOS-M** 是 OpenHarmony 操作系统的轻量级内核组件，专为物联网（IoT）设备设计。

> 引用: "OpenHarmony LiteOS-M is a lightweight operating system kernel designed for the Internet of Things (IoT) field." (README.md:12)

## 核心特性

| 特性 | 说明 |
|------|------|
| **小体积** | 最小内核功能集，代码结构简洁 |
| **低功耗** | 针对能耗敏感场景优化 |
| **高性能** | 高效的调度与内存管理 |
| **可配置** | 支持模块化裁剪与定制 |

## 架构边界

```
+-------------------+
|   应用层 (用户态)  |  ← 不在本项目范围内
+-------------------+
         ↓
+-------------------+
|   系统服务层      |  ← 不在本项目范围内
+-------------------+
         ↓
+-------------------+
|   LiteOS-M 内核   |  ← 本项目
+-------------------+
         ↓
+-------------------+
|   硬件层 (HAL)    |  ← 芯片厂商实现
+-------------------+
```

## 核心能力

### 1. 任务管理 (`kernel/src/los_task.c`)
- 任务创建、删除、调度
- 优先级管理
- 任务状态机

### 2. 进程间通信
- 消息队列 (`los_queue.c`)
- 信号量 (`los_sem.c`)
- 互斥锁 (`los_mux.c`)
- 事件 (`los_event.c`)

### 3. 内存管理 (`kernel/src/mm/`)
- 动态内存分配
- 静态内存池

### 4. 时间管理
- 系统节拍 (`los_tick.c`)
- 软件定时器 (`los_swtmr.c`)

### 5. 中断与异常处理
- 架构相关中断处理 (`arch/`)

## 支持的芯片架构

| 架构 | 路径 | 说明 |
|------|------|------|
| ARM32 | `arch/arm/` | ARM9, Cortex-M3/M4/M33/M7 |
| ARM64 | `arch/aarch64/` | 64位 ARM |
| RISC-V | `arch/risc-v/` | nuclei, riscv32 |
| C-Sky | `arch/csky/` | v2 |
| Xtensa | `arch/xtensa/` | lx6 |

> 详见: [arch_spec.md](../arch_spec.md)

## 语言限制

**仅支持 C 和 C++**

> 引用: "OpenHarmony LiteOS-M supports only C and C++." (README.md:69)

## 安全要求

动态加载模块必须进行签名验证或来源限制：

> 引用: "As for dynamic loading module, the shared library to be loaded needs signature verification or source restriction to ensure security." (README.md:73)

## 相关文档

- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [内核 API](04_Kernel_API.md)
