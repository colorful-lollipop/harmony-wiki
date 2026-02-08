# 安全风险评审

## 评审概述

### 评审范围

本安全评审覆盖 device_qemu 仓库的以下组件：

| 组件 | 范围 | 证据位置 |
|------|------|----------|
| **VirtIO 驱动** | virtblock, virtnet, virtgpu, virtinput, virtrng | `drivers/virtio/` |
| **UART 驱动** | uart_pl011, uart core | `drivers/uart/` |
| **字符设备驱动** | char, mmz | `drivers/char/` |
| **构建配置** | BUILD.gn, Kconfig, lite.mk | `drivers/` |

### 评审局限性

| 限制项 | 说明 |
|--------|------|
| **代码规模** | device_qemu 是设备模拟层，代码量相对较小 |
| **不涉及领域** | 不涉及用户态应用、IPC 框架、N-API 接口 |
| **验证方式** | 静态代码分析，未进行动态安全测试 |

## 攻击面分析

### 威胁模型

```
┌─────────────────────────────────────────────────────────────────┐
│                    Threat Model                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  External Attack Surfaces:                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │ Network     │  │ Block I/O   │  │ Serial      │              │
│  │ (virtio-net)│  │ (virtio-blk)│  │ (UART)      │              │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
│         ▼                ▼                ▼                      │
│  ┌─────────────────────────────────────────────────────┐        │
│  │              VirtIO Driver Layer                     │        │
│  │       (drivers/virtio/*.c, drivers/uart/*.c)        │        │
│  └─────────────────────────────────────────────────────┘        │
│         │                                                      │
│         ▼                                                      │
│  ┌─────────────────────────────────────────────────────┐        │
│  │              LiteOS Kernel                            │        │
│  │       (Memory, Scheduling, IPC)                      │        │
│  └─────────────────────────────────────────────────────┘        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 攻击面清单

### 外部输入入口

| 入口 | 组件 | 数据类型 | 信任级别 | 验证状态 |
|------|------|---------|---------|---------|
| **网络数据包** | virtio-net | 以太网帧 | 低信任 | 依赖 LWIP 栈 |
| **块设备请求** | virtio-blk | SCSI 命令块 | 中信任 | 描述符需验证 |
| **串口输入** | uart_pl011 | 字符流 | 低信任 | FIFO 边界检查 |
| **GPU 命令** | virtgpu | 显示命令 | 中信任 | 无用户态输入 |
| **输入事件** | virtinput | 按键/坐标 | 低信任 | 需验证坐标范围 |
| **随机数** | virtrng | 熵数据 | 高信任 | 硬件源可信 |

### 敏感操作

| 操作 | 组件 | 权限要求 | 风险说明 |
|------|------|---------|---------|
| **DMA 内存访问** | virtio 驱动 | 内核态 | 需验证地址范围 |
| **MMIO 寄存器访问** | virtmmio | 内核态 | QEMU 模拟可信 |
| **物理内存分配** | mmz | 内核态 | 需验证 size 参数 |
| **中断处理** | 所有驱动 | 内核态 | 需原子操作保护 |

### 信任边界跨越点

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界模型                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  QEMU 用户态空间 (低信任)                                 │   │
│  │  - 网络数据包来自 tap0 接口                               │   │
│  │  - 块设备镜像文件                                        │   │
│  │  - 串口输入来自 stdio/日志                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                   │
│                              │ VirtIO 协议                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  device_qemu 内核驱动 (高信任)                           │   │
│  │  - virtio-net: 网络数据处理                              │   │
│  │  - virtio-blk: 块设备请求处理                            │   │
│  │  - uart_pl011: 串口数据接收                              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**边界跨越说明**:
1. **QEMU → device_qemu**: VirtIO 描述符通道（需验证）
2. **device_qemu → LiteOS**: 系统调用接口（框架保护）
3. **device_qemu → 硬件**: MMIO 寄存器（QEMU 模拟）

## 信任边界

### 边界定义

```
┌─────────────────────────────────────────────────────────────────┐
│                    Trust Boundaries                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Boundary 1: QEMU Simulation                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Host OS: Linux/macOS                                   │   │
│  │  - QEMU provides hardware isolation                      │   │
│  │  - VM/sandbox boundary                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          ▲                                      │
│                          │ trusted                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  device_qemu: Kernel Drivers                             │   │
│  │  - Runs in kernel space                                  │   │
│  │  - Full access to kernel resources                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          ▲                                      │
│                          │ trusted                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  VirtIO Backend: QEMU Emulated Devices                   │   │
│  │  - Host file system access (block)                       │   │
│  │  - Network interface (tap)                               │   │
│  │  - Serial console                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 潜在安全风险

### 风险 1: VirtIO 描述符验证不足

**风险等级**: 中

**证据位置**: `drivers/virtio/virtblock.c:142-170`

**问题描述**:
VirtIO 驱动在处理来自 QEMU 的描述符时，存在验证不充分的情况。在 `VirtblkIO()` 函数中，直接使用用户提供缓冲区地址和长度。

**证据代码**:
```c
// drivers/virtio/virtblock.c:142-170
static uint8_t VirtblkIO(struct Virtblk *blk, uint32_t cmd, uint64_t startSector,
                          uint8_t *buf, uint32_t sectors)
{
    // ...
    /* fill in and notify virt queue */
    blk->req.type = cmd;
    blk->req.startSector = startSector;
    q->desc[1].pAddr = VMM_TO_DMA_ADDR((VADDR_T)buf);  // 直接使用用户缓冲区地址
    q->desc[1].len = sectors * MMC_SEC_SIZE;          // 直接使用用户提供的扇区数
    // ...

**影响**:
- 内存损坏
- 内核崩溃 (DoS)
- 信息泄露

**修复建议**:
```c
// 在处理描述符前增加验证
static int VirtBlkValidateDesc(struct virtqueue_desc *desc) {
    // 验证物理地址范围
    if (desc->addr + desc->len > MAX_PHYS_ADDR) {
        return HDF_ERR_INVALID_ADDR;
    }
    // 验证长度范围
    if (desc->len == 0 || desc->len > MAX_BLK_SIZE) {
        return HDF_ERR_INVALID_SIZE;
    }
    return HDF_SUCCESS;
}
```

### 风险 2: UART 缓冲区溢出风险

**风险等级**: 低

**证据位置**: `drivers/uart/uart_pl011.c:31-67`

**问题描述**:
UART 驱动在中断处理函数中，使用固定大小的缓冲区接收数据。虽然有 `count < FIFO_SIZE` 检查，但在高频率数据输入场景下可能触发缓冲区压力。

**证据代码**:
```c
// drivers/uart/uart_pl011.c:31-67
static uint32_t Pl011Irq(uint32_t irq, void *data)
{
    // ...
    char buf[FIFO_SIZE];  // FIFO_SIZE = 128 (第27行)
    uint32_t count = 0;
    // ...
    do {
        // ...
        buf[count++] = OSAL_READB(port->physBase + UART_DR);  // 直接写入缓冲区
        // ...
    } while (count < FIFO_SIZE);  // 边界检查存在
    udd->recv(udd, buf, count);   // 传递给上层处理
}
```

**风险分析**:
| 检查项 | 状态 | 说明 |
|--------|------|------|
| 缓冲区大小固定 | ✅ 安全 | FIFO_SIZE 定义为 128 字节 |
| 边界检查 | ✅ 安全 | `count < FIFO_SIZE` 防止溢出 |
| 上层处理依赖 | ⚠️ 风险 | `udd->recv()` 回调函数需正确处理长度 |

**影响**:
- 内核栈溢出
- 数据 corruption
- 潜在的代码执行

**修复建议**:
```c
// UART 接收缓冲区处理
static int UartReceiveData(struct UartDevice *dev, char *data, size_t len) {
    size_t avail = dev->rx_buf_size - dev->rx_buf_len;
    size_t to_copy = MIN(len, avail);

    if (to_copy < len) {
        HDF_LOGW("UART: truncating data from %zu to %zu", len, to_copy);
    }

    memcpy(dev->rx_buf + dev->rx_buf_len, data, to_copy);
    dev->rx_buf_len += to_copy;
    return to_copy;
}
```

### 风险 3: MMZ 内存越界访问

**风险等级**: 低

**证据位置**: `drivers/char/mmz/mmz.c:54-80`

**问题描述**:
MMZ (Memory Management Zone) 在进行内存分配时，依赖调用者传入正确的参数。在 `MmzAlloc()` 函数中，对 `mmzm->size` 的验证较为简单。

**证据代码**:
```c
// drivers/char/mmz/mmz.c:54-80
static ssize_t MmzAlloc(int cmd, unsigned long arg)
{
    // ...
    MmzMemory *mmzm = (MmzMemory *)arg;
    UINT32 size = ROUNDUP(mmzm->size, PAGE_SIZE);  // 第66行: size 计算依赖输入
    // ...
    switch (cmd) {
        case MMZ_CACHE_TYPE:
        case MMZ_NOCACHE_TYPE:
            // ... 无 size 范围检查
        default:
            PRINT_ERR("%s %d: %d\n", __func__, __LINE__, cmd);
            return -EINVAL;  // 仅检查 cmd 有效性
    }
    vmRegion = LOS_RegionAlloc(curVmSpace, 0, size, vmFlags, 0);  // 使用计算后的 size
}
```

**风险分析**:
| 检查项 | 状态 | 说明 |
|--------|------|------|
| cmd 参数验证 | ✅ 安全 | default 分支返回错误 |
| size 参数验证 | ⚠️ 需注意 | 仅做 PAGE_SIZE 对齐，无上限检查 |
| 内存分配结果 | ✅ 安全 | `LOS_RegionAlloc()` 有内部保护机制 |

**修复建议**:
```c
// MMZ 地址范围检查
static int MmzValidateRange(phys_addr_t phys, size_t size) {
    if (phys < MMZ_START_ADDR) {
        return HDF_ERR_INVALID_ADDR;
    }
    if (phys + size > MMZ_END_ADDR) {
        return HDF_ERR_INVALID_ADDR;
    }
    return HDF_SUCCESS;
}
```

### 风险 4: VirtIO 中断处理竞争条件

**风险等级**: 低

**证据位置**: `drivers/virtio/virtblock.c:172-187`

**问题描述**:
VirtIO 设备的中断处理程序存在潜在的竞态条件风险。在高并发场景下，多个中断可能同时触发，导致 virtqueue 状态不一致。

**证据代码**:
```c
// drivers/virtio/virtblock.c:172-187
static uint32_t VirtblkIRQhandle(uint32_t swIrq, void *dev)
{
    (void)swIrq;
    struct Virtblk *blk = dev;
    struct Virtq *q = &blk->dev.vq[0];

    // 检查中断状态，无锁保护
    if (!(OSAL_READL(blk->dev.base + VIRTMMIO_REG_INTERRUPTSTATUS) & VIRTMMIO_IRQ_NOTIFY_USED)) {
        return 1;
    }

    (void)DmaEventSignal(&blk->event, 1);
    q->last++;  // 修改共享状态，无原子操作保护

    OSAL_WRITEL(VIRTMMIO_IRQ_NOTIFY_USED, blk->dev.base + VIRTMMIO_REG_INTERRUPTACK);
    return 0;
}
```

**风险分析**:
| 检查项 | 状态 | 说明 |
|--------|------|------|
| 中断状态检查 | ✅ 安全 | 先检查中断标志位 |
| 状态修改 | ⚠️ 潜在竞态 | `q->last++` 在多中断场景下可能竞争 |
| 同步机制 | ✅ 安全 | `DmaEventSignal()` 提供事件通知 |

**缓解因素**:
- 驱动设计为同步模式（注释第28-34行）："only have 4 descriptors... always in synchronous mode"
- 单请求队列设计减少并发风险

**修复建议**:
```c
// VirtIO 中断处理加锁
static irqreturn_t VirtIoIsr(int irq, void *dev_id) {
    unsigned long flags;
    struct VirtIoDevice *dev = (struct VirtIoDevice *)dev_id;

    spin_lock_irqsave(&dev->lock, flags);

    // 处理中断
    uint8_t status = VirtIoReadReg(dev, VIRTIO_MMIO_INTERRUPT_STATUS);
    if (status & VIRTQUEUE_IRQ) {
        VirtQueueNotify(dev->vqs);
    }

    spin_unlock_irqrestore(&dev->lock, flags);
    return IRQ_HANDLED;
}
```

### 风险 5: 配置依赖关系风险

**风险等级**: 低

**证据位置**: `drivers/Kconfig:22-42`

**问题描述**:
Kconfig 配置选项之间存在依赖关系，不当的配置组合可能导致安全边界模糊。

**证据代码**:
```kconfig
# drivers/Kconfig:22-42

# MMZ 依赖字符设备驱动
config DRIVERS_MMZ_CHAR_DEVICE
    bool "Enable MMZ Platform Char Device Drivers"
    default y
    depends on DRIVERS_PLATFORM_CHAR_DEVICE && FS_VFS   # 依赖检查存在
    help
      Enable MMZ Platform Char Device Drivers.

# 字符设备驱动依赖文件系统
config DRIVERS_PLATFORM_CHAR_DEVICE
    bool "Enable Platform Char Device Drivers"
    default y
    depends on FS_VFS   # 依赖检查存在
    help
      Enable Platform Char Device Drivers.
```

**配置风险分析**:
| 配置项 | 依赖关系 | 风险等级 | 说明 |
|--------|---------|---------|------|
| `DRIVERS_MMZ_CHAR_DEVICE` | → `DRIVERS_PLATFORM_CHAR_DEVICE` → `FS_VFS` | 中 | MMZ 依赖链完整 |
| `DRIVERS_HDF_PLATFORM_UART` | → `DRIVERS_HDF_PLATFORM` | 低 | UART 驱动依赖 HDF |
| `DRIVERS_NETDEV` | → `DRIVERS` && `NET_LWIP_SACK` | 低 | 网络设备依赖网络栈 |

**安全建议**:
1. 确保 `FS_VFS` 和 `NET_LWIP_SACK` 在启用网络/MMZ 前已正确配置
2. 避免在生产环境使用 `default y` 的调试相关配置

## 安全最佳实践

### 已采用的安全措施

| 措施 | 实现位置 | 状态 | 说明 |
|------|---------|------|------|
| **HDF 框架** | 全局 | ✅ 已采用 | 统一的驱动框架，提供标准化接口 |
| **静态库链接** | BUILD.gn | ✅ 已采用 | 避免运行时动态加载风险 |
| **条件编译** | BUILD.gn | ✅ 已采用 | 可选功能通过编译开关控制 |
| **依赖检查** | Kconfig | ✅ 已采用 | 配置间依赖关系明确 |
| **FIFO 边界检查** | uart_pl011.c:60 | ✅ 已采用 | `count < FIFO_SIZE` 防止溢出 |
| **同步模式设计** | virtblock.c:28-34 | ✅ 已采用 | 单请求队列减少并发风险 |
| **中断状态检查** | virtblock.c:178 | ✅ 已采用 | 先检查标志位再处理 |

### 建议的安全增强

| 建议 | 优先级 | 风险关联 | 说明 |
|------|--------|---------|------|
| VirtIO 描述符验证 | 高 | 风险 1 | 增加地址和长度检查 |
| MMZ size 参数验证 | 中 | 风险 3 | 增加 size 上限检查 |
| VirtIO 状态原子操作 | 低 | 风险 4 | 使用原子操作保护共享状态 |

### 安全评估总结

| 评估维度 | 评分 | 说明 |
|---------|------|------|
| **输入验证** | ⭐⭐⭐☆☆ | UART 有边界检查，VirtIO 描述符需增强 |
| **内存安全** | ⭐⭐⭐⭐☆ | MMZ 有 PAGE_SIZE 对齐，缓冲区使用安全 |
| **并发安全** | ⭐⭐⭐⭐☆ | 同步模式设计减少竞态风险 |
| **配置安全** | ⭐⭐⭐⭐⭐ | Kconfig 依赖关系清晰 |
| **整体风险** | ⭐⭐⭐☆☆ | 中低风险，设备模拟层暴露面有限 |

## 相关文档

| 文档 | 说明 |
|------|------|
| [架构设计](03_Architecture.md) | 驱动架构与数据流 |
| [GN 构建](04_GN_Build.md) | 构建配置与安全相关编译选项 |
| [常见问题](08_Troubleshooting.md) | 安全相关问题排查 |
