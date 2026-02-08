# 项目概览

## 项目定位

`drivers_liteos` 是 OpenHarmony 操作系统的 **LiteOS_A 内核驱动子系统**，负责提供内核态驱动程序，支持用户态进程通过标准文件系统接口访问硬件资源。

**核心职责**：
- 提供字符设备驱动框架
- 实现内核与用户空间的数据传输通道
- 封装硬件操作细节，向上层提供统一 API

## 核心能力

### 当前开源模块

| 模块 | 功能 | 状态 | 位置 |
|-----|------|------|------|
| **hievent** | 事件日志管理驱动 | ✅ 完整开源 | `hievent/` |

### 受限/未开源模块

| 模块 | 功能 | 状态 | 说明 |
|-----|------|------|------|
| **tzdriver** | REE/TEE 通信驱动 | ⚠️ 部分开源 | 需要合作获取支持 |
| **hievent 配套功能** | 事件上报能力 | 📍 未开放 | 待后续开源 |

### 外部依赖模块

| 模块 | 功能 | 位置 | 说明 |
|-----|------|------|------|
| **mem** | 物理 I/O 访问 | `kernel/liteos_a/drivers/char` | 使用 mmap |
| **random** | 随机数生成 | `kernel/liteos_a/drivers/char` | TRNG/PRNG |
| **video** | 帧缓冲驱动 | `third_party/NuttX` | NuttX 子模块 |
| **quickstart** | 快速启动 | `kernel/liteos_a/drivers/char` | - |

## 运行环境

### 适用平台

| 平台 | 支持状态 | 说明 |
|-----|---------|------|
| LiteOS_A | ✅ 原生支持 | 本仓库主要目标 |
| Linux | ⚠️ 兼容 | 部分驱动可移植 |

### 依赖条件

| 依赖项 | 版本要求 | 说明 |
|-------|---------|------|
| LiteOS_A Kernel | - | 内核基础框架 |
| build GN | - | 构建系统 |
| GCC / LLVM | - | 交叉编译工具链 |

## 关键概念

### 字符设备驱动模式

本仓库采用标准的 Linux 字符设备驱动模式：

```c
// 设备操作向量表
static struct file_operations_vfs g_hieventFops = {
    .open  = HieventOpen,
    .close = HieventClose,
    .read  = HieventRead,
    .write = HieventWrite,
    .poll  = HieventPoll,
};

// 注册设备节点
register_driver("/dev/hwlog_exception", &g_hieventFops, DRIVER_MODE, &g_hieventDev);
```

**交互方式**：用户态进程使用标准文件操作（`open`/`read`/`write`/`ioctl`/`mmap`）

### 环形缓冲区

hievent 模块使用 **环形缓冲区** 实现事件日志：

| 参数 | 值 | 说明 |
|-----|---|------|
| 缓冲区大小 | 1024 字节 | `HIEVENT_LOG_BUFFER` |
| 设备节点 | `/dev/hwlog_exception` | 事件日志入口 |
| 同步机制 | LosMux | 互斥锁保护 |

## 相关仓库

| 仓库 | 关系 | 说明 |
|-----|------|------|
| [kernel_liteos_a](https://gitee.com/openharmony/kernel_liteos_a) | 上游依赖 | LiteOS_A 内核 |
| [drivers_liteos](https://gitee.com/openharmony/drivers_liteos) | 主仓库 | 本仓库镜像 |
| [third_party/NuttX](https://gitee.com/openharmony/third_party_NuttX) | 外部依赖 | NuttX 子模块 |

## 快速开始

### 获取代码

```bash
# 克隆仓库
git clone https://gitee.com/openharmony/drivers_liteos.git
cd drivers_liteos
```

### 编译模块

```bash
# 启用 hievent 模块
cd hievent
make

# 或使用 GN 构建
hb build -f
```

### 使用示例

```c
#include "hiview_hievent.h"

// 创建事件
struct HiviewHievent *event = HiviewHieventCreate(event_id);

// 添加键值
HiviewHieventPutIntegral(event, "error_code", code);
HiviewHieventPutString(event, "message", msg);

// 上报事件
HiviewHieventReport(event);

// 释放资源
HiviewHieventDestroy(event);
```

## 下一步

| 目标 | 推荐阅读 |
|-----|---------|
| 理解架构 | [03_Architecture.md](03_Architecture.md) |
| 使用 API | [04_Hievent_API.md](04_Hievent_API.md) |
| 构建配置 | [05_Build_Configuration.md](05_Build_Configuration.md) |
| 安全考虑 | [06_Security_Analysis.md](06_Security_Analysis.md) |

---

*最后更新: 2024*
