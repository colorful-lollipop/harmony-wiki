# 01 - 项目概览 (Project Overview)

> 文档版本: 1.0  
> 更新日期: 2025-02-07  
> 适用版本: OpenHarmony tee_os_kernel (基于 ChCore)

---

## 1. 一句话定义

**OpenTrustee TEE OS Kernel** 是 OpenHarmony 可信执行环境（TEE）的微内核操作系统，基于上海交大 IPADS 的 ChCore 研究系统，运行在 ARM TrustZone 安全世界中，为安全敏感的应用（如指纹、支付、DRM）提供硬件隔离的执行环境。

---

## 2. 核心能力边界

### 2.1 能做什么

| 能力类别 | 具体功能 | 代码位置 |
|----------|----------|----------|
| **进程管理** | 创建/销毁进程、线程调度、Capability 权限管理 | `kernel/object/cap_group.c`, `kernel/sched/` |
| **内存管理** | 物理内存分配（Buddy/Slab）、虚拟内存映射、COW 支持 | `kernel/mm/`, `kernel/object/memory.c` |
| **IPC 通信** | 同步 RPC（Connection）、异步通知（Notification）、TEE 专用通道（Channel） | `kernel/ipc/` |
| **中断处理** | IRQ 注册/分发、定时器、IPI 核间中断 | `kernel/irq/`, `kernel/object/irq.c` |
| **TrustZone 集成** | SMC 调用处理、安全世界切换、与 Normal World 通信 | `kernel/arch/aarch64/trustzone/` |
| **系统调用** | 256 个系统调用接口，覆盖内存、IPC、调度、硬件访问 | `kernel/syscall/syscall_num.h` |

### 2.2 不能做什么

| 限制 | 说明 | 原因 |
|------|------|------|
| **无网络协议栈** | 不支持 TCP/IP、UDP 等网络通信 | TEE 安全设计，网络功能由 Rich OS 提供 |
| **无文件系统驱动** | 文件系统（tmpfs）运行在用户态 | 微内核架构，驱动移出内核 |
| **无 GUI 支持** | 无显示、输入等图形接口 | TEE 无直接访问显示硬件 |
| **有限的硬件支持** | 仅支持 ARM64 (RK3568/RK3399) | 专注于特定 TEE 平台 |
| **无动态模块加载** | 不支持运行时加载内核模块 | 安全考虑，减少攻击面 |

---

## 3. 运行环境要求

### 3.1 硬件要求

| 组件 | 要求 | 说明 |
|------|------|------|
| **处理器** | ARM64 (AArch64) | Cortex-A53/A72 等 |
| **TrustZone** | ARM TrustZone 支持 | EL3 Secure Monitor |
| **内存** | 最少 8MB RAM | 根据 bundle.json |
| **存储** | 最少 2MB ROM | 内核镜像大小 |
| **平台** | Rockchip RK3568/RK3399 | 当前支持的平台 |

### 3.2 软件依赖

| 依赖 | 版本/来源 | 用途 |
|------|-----------|------|
| **Secure Monitor** | OP-TEE SPD 或 TEED SPD | EL3 固件接口 |
| **tee_os_framework** | OpenHarmony 配套仓库 | TEE 框架层（CA/TA 接口） |
| **构建工具** | Clang + LLVM | 编译器 |
| **构建系统** | GN + Ninja | OpenHarmony 标准构建 |

### 3.3 权限要求

| 操作 | 所需权限 |
|------|----------|
| 编译内核 | 普通用户权限 |
| 烧录 TEE 镜像 | 需要设备烧录权限 |
| 调试 TEE | 需要 JTAG 或串口访问 |
| SMC 调用 | 仅在 Secure World 执行 |

---

## 4. 关键概念

### 4.1 Capability（能力）

Capability 是 ChCore 微内核的核心安全机制，替代传统的 UID/GID 权限模型：

```c
// kernel/include/object/object.h:40-53
enum object_type {
    TYPE_CAP_GROUP = 0,     // 进程（Capability 组）
    TYPE_THREAD,            // 线程
    TYPE_CONNECTION,        // IPC 连接
    TYPE_NOTIFICATION,      // 通知对象
    TYPE_IRQ,               // IRQ 对象
    TYPE_PMO,               // 物理内存对象
    TYPE_VMSPACE,           // 虚拟内存空间
    TYPE_CHANNEL,           // TEE 通道
    TYPE_MSG_HDL,           // TEE 消息句柄
    TYPE_NR,
};
```

**核心特性**:
- 每个内核对象通过 Capability 访问
- Capability 不可伪造，只能复制或传递
- 支持 Capability 的撤销和转移

### 4.2 PMO (Physical Memory Object)

PMO 是 ChCore 的内存管理抽象，将物理内存封装为可传递的对象：

```c
// PMO 类型定义
PMO_DATA = 0,       // 数据段
PMO_ANONYM,         // 匿名映射（延迟分配）
PMO_FILE,           // 文件映射
PMO_SHM,            // 共享内存
PMO_DEVICE,         // 设备寄存器
PMO_TEE_NS,         // TEE 非安全内存（OH-TEE）
PMO_TEE_SHM,        // TEE 共享内存（OH-TEE）
PMO_FORBID,         // 禁止访问区域（安全边界）
```

### 4.3 IPC 机制

TEE OS 提供三种 IPC 机制：

| 机制 | 类型 | 用途 | 代码 |
|------|------|------|------|
| **Connection** | 同步 RPC | Client-Server 调用 | `kernel/ipc/connection.c` |
| **Notification** | 异步信号 | 线程同步、事件通知 | `kernel/ipc/notification.c` |
| **Channel** | TEE 消息 | OH-TEE 专用消息传递 | `kernel/ipc/channel.c` |

### 4.4 TrustZone 边界

```
┌─────────────────────────────────────────────────────────────┐
│                    Normal World (EL0/EL1)                   │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────┐  │
│  │  Android App │      │  Linux App   │      │  Driver  │  │
│  └──────┬───────┘      └──────┬───────┘      └────┬─────┘  │
│         │                     │                    │        │
│  ┌──────▼─────────────────────▼────────────────────▼──────┐ │
│  │                     Rich OS (Linux)                    │ │
│  └──────┬─────────────────────┬────────────────────┬──────┘ │
└─────────┼─────────────────────┼────────────────────┼────────┘
          │                     │                    │
          │              SMC Interface                │
          │                     │                    │
┌─────────┼─────────────────────┼────────────────────┼────────┐
│         │                     │                    │        │
│  ┌──────▼─────────────────────▼────────────────────▼──────┐ │
│  │                   Secure Monitor (EL3)                 │ │
│  │              (OP-TEE SPD / TEED SPD)                   │ │
│  └──────┬─────────────────────┬────────────────────┬──────┘ │
│         │                     │                    │        │
│  ┌──────▼─────────────────────▼────────────────────▼──────┐ │
│  │                   Secure World (EL1/EL0)               │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │ │
│  │  │ TEE OS Kernel│  │   System     │  │ Trusted App  │  │ │
│  │  │  (本仓库)     │  │   Servers    │  │   (TA)       │  │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │ │
│  │                                                         │ │
│  │  ┌─────────────────────────────────────────────────────┐│ │
│  │  │              Trusted Hardware (Crypto, etc.)        ││ │
│  │  └─────────────────────────────────────────────────────┘│ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. 快速开始

### 5.1 代码获取

```bash
# 克隆仓库
git clone https://gitcode.com/openharmony/tee_tee_os_kernel.git

# 或从 OpenHarmony 主仓库同步
repo sync -c base/tee/tee_os_kernel
```

### 5.2 构建命令

```bash
# 完整构建（推荐）
./build.sh --product-name rk3568 --build-target tee --ccache

# GN 构建（独立构建）
cd //base/tee/tee_os_kernel/build
gn gen out
ninja -C out

# 构建产物
# 输出位置: base/tee/tee_os_kernel/kernel/bl32.bin
```

### 5.3 目录导航

```bash
# 内核核心代码
cd kernel/

# 系统调用
cd kernel/syscall/

# 内存管理
cd kernel/mm/

# IPC 实现
cd kernel/ipc/

# 用户态服务
cd user/system-services/system-servers/
```

---

## 6. 典型使用场景

### 6.1 新人学习路线

1. **了解 TEE 基础** → 阅读 ARM TrustZone 文档
2. **理解微内核架构** → 阅读本文档 02_Architecture.md
3. **学习 Capability 机制** → 阅读 `kernel/object/capability.c`
4. **掌握系统调用** → 阅读 `kernel/syscall/syscall_num.h`
5. **研究 IPC 机制** → 阅读 `kernel/ipc/connection.c`

### 6.2 安全研究路线

1. **识别攻击面** → 阅读本文档 05_AttackSurface.md
2. **分析系统调用** → 检查 `check_user_addr_range()` 使用
3. **审计内存管理** → 检查 PMO 边界验证
4. **研究 IPC 安全** → 检查 Capability 传递验证
5. **TrustZone 边界** → 检查 SMC 调用验证

---

## 7. 相关资源

### 7.1 项目内文档

| 文档 | 内容 |
|------|------|
| [02_Architecture.md](02_Architecture.md) | 微内核架构详解 |
| [03_CodeMap.md](03_CodeMap.md) | 代码导航地图 |
| [04_Interface.md](04_Interface.md) | 系统调用和 IPC 接口 |
| [05_AttackSurface.md](05_AttackSurface.md) | 攻击面分析 |
| [06_SecurityReview.md](06_SecurityReview.md) | 安全风险评估 |
| [07_Build.md](07_Build.md) | 构建系统详解 |
| [08_Internals.md](08_Internals.md) | 内部实现细节 |

### 7.2 外部资源

| 资源 | 链接 |
|------|------|
| 相关仓库 | [tee_os_framework](https://gitcode.com/openharmony/tee_tee_os_framework) |
| ChCore 论文 | https://ipads.se.sjtu.edu.cn/pub/projects/chcore |
| ARM TrustZone | https://developer.arm.com/documentation/100720/0300 |
| OP-TEE 文档 | https://optee.readthedocs.io/ |
| seL4 微内核 | https://sel4.systems/ |

---

## 8. 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| TEE | Trusted Execution Environment | 可信执行环境 |
| TA | Trusted Application | 可信应用 |
| CA | Client Application | 客户端应用（Normal World） |
| SMC | Secure Monitor Call | 安全监视器调用 |
| PMO | Physical Memory Object | 物理内存对象 |
| Capability | Capability | 能力/权限令牌 |
| IPC | Inter-Process Communication | 进程间通信 |
| SPD | Secure Payload Dispatcher | 安全载荷分发器 |
| REE | Rich Execution Environment | 丰富执行环境（Normal World） |
| COW | Copy-On-Write | 写时复制 |
| ELx | Exception Level x | ARM 异常级别 |

---

*本文档基于代码证据生成，所有技术结论均可追溯至具体文件和行号。*
