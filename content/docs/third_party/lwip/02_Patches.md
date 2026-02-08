# Patch 详细分析

## 概述

OpenHarmony 的 lwIP 没有采用传统的 `.patch` 文件方式来管理修改，而是将 OH 特有功能**直接集成**到代码库中，通过**条件编译宏**进行控制。

这种设计的好处：
- 代码与上游同步更容易
- OH 特有功能模块化，可独立开关
- 便于调试和问题定位

## Patch 清单

| 功能领域 | 条件宏 | 新增文件数 | 修改文件数 | 代码行数 |
|----------|--------|------------|------------|----------|
| 网络容器 | `LOSCFG_NET_CONTAINER` | 2 | 45+ | ~500+ |
| 低功耗模式 | `LWIP_LOWPOWER` | 2 | 35+ | ~800+ |
| 分布式网络 | `LWIP_ENABLE_DISTRIBUTED_NET` | 8 | 5+ | ~1000+ |
| **总计** | - | **12** | **68** | **~2300+** |

---

## 1. 网络容器支持 (LOSCFG_NET_CONTAINER)

### 1.1 原始问题

在标准 lwIP 中，所有网络资源（网卡、连接、协议控制块）都是全局的，无法支持：
- 多应用网络隔离
- 容器化部署
- 网络命名空间

### 1.2 OH 解决方案

引入 **Network Group** 概念，将网络资源按组隔离：

```
┌─────────────────────────────────────────────────────────┐
│                      系统全局                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Network     │  │  Network     │  │  Network     │  │
│  │  Group A     │  │  Group B     │  │  Group C     │  │
│  │              │  │              │  │              │  │
│  │ • netif_list │  │ • netif_list │  │ • netif_list │  │
│  │ • netif_default│ │ • netif_default│ │ • netif_default│ │
│  │ • loop_netif │  │ • loop_netif │  │ • loop_netif │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 1.3 新增文件

#### net_group.h
```c
// src/include/lwip/net_group.h
struct net_group {
    u8_t netif_num;
    struct netif *netif_default;
    struct netif *netif_list;
    struct netif *loop_netif;
};

struct net_group_ops {
    struct net_group *(*get_curr_process_net_group)(void);
    void (*set_netif_net_group)(struct netif *, struct net_group *);
    struct net_group *(*get_net_group_from_netif)(struct netif *);
    void (*set_ippcb_net_group)(struct ip_pcb *, struct net_group *);
    struct net_group *(*get_net_group_from_ippcb)(struct ip_pcb *);
};
```

#### net_group.c
- 提供默认的 network group 操作实现
- 支持运行时替换 ops（用于内核实现网络容器）

### 1.4 修改的核心文件

| 文件 | 修改内容 |
|------|----------|
| `src/include/lwip/netif.h` | 添加 `netif.net_group` 字段 |
| `src/include/lwip/ip.h` | IP_PCB 添加 `net_group` 字段 |
| `src/include/lwip/ip4.h` | IPv4 路由按 network group 隔离 |
| `src/include/lwip/ip6.h` | IPv6 路由按 network group 隔离 |
| `src/core/netif.c` | netif 操作关联到 network group |
| `src/core/ipv4/ip4.c` | 按 network group 查找路由 |
| `src/core/ipv6/ip6.c` | 按 network group 查找路由 |
| `src/core/tcp.c` | TCP PCB 关联 network group |
| `src/core/udp.c` | UDP PCB 关联 network group |
| `src/core/raw.c` | Raw PCB 关联 network group |
| `src/api/api_msg.c` | API 消息处理关联 network group |
| `src/core/tcp_in.c` | TCP 输入处理按 network group |

### 1.5 关键代码变更示例

**修改前 (标准 lwIP)**:
```c
// netif.c
struct netif *netif_list;
struct netif *netif_default;
```

**修改后 (OH lwIP)**:
```c
// netif.c
#ifdef LOSCFG_NET_CONTAINER
struct netif *netif_list;        // 移到 net_group 中
struct netif *netif_default;     // 移到 net_group 中
#else
struct netif *netif_list;
struct netif *netif_default;
#endif
```

### 1.6 OH 价值

- **网络隔离**: 不同应用/容器使用独立网络栈
- **安全增强**: 限制应用的网络访问范围
- **资源管理**: 按 network group 统计和限制网络资源

### 1.7 回归风险

- **高**: 核心数据结构 (`netif`, `ip_pcb`) 被修改
- **高**: 网络路由查找路径被修改
- **中**: 与上游同步时需要仔细合并

---

## 2. 低功耗模式 (LWIP_LOWPOWER)

### 2.1 原始问题

标准 lwIP 的定时器系统需要周期性轮询（通常 100ms 一次），导致：
- CPU 无法进入深度睡眠
- 功耗较高，不适合电池供电设备

### 2.2 OH 解决方案

重构定时器系统，支持**事件驱动**和**睡眠唤醒**：

```
┌─────────────────────────────────────────────────────────┐
│                   标准 lwIP 定时器                       │
│  CPU ──→ [定时中断 100ms] ──→ 检查所有定时器 ──→ 处理    │
│                              (CPU 无法睡眠)              │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                   OH 低功耗定时器                        │
│  CPU ──→ [计算下次唤醒时间] ──→ 睡眠 N ms ──→ [事件唤醒]  │
│         ↑                                    │         │
│         └────────── 处理定时器任务 ───────────┘         │
│                              (CPU 可深度睡眠)            │
└─────────────────────────────────────────────────────────┘
```

### 2.3 新增文件

#### lowpower.h
```c
// src/include/lwip/lowpower.h

#define TIMEOUT_TICK 100  // 100ms 基础时间单位

struct timer_handler {
  u32_t interval;
  lwip_timer_handler handler;
  get_next_timeout next_tick;  // 动态计算下次触发时间
};

struct timer_entry {
  u32_t clock_max;
  u32_t timeout;
  sys_timeout_handler handler;
  get_next_timeout next_tick;  // 允许定时器动态调整间隔
  u8_t enable;
};

// 低功耗模式控制
enum lowpower_mod {
  LOW_TMR_LOWPOWER_MOD = 0,  // 启用低功耗
  LOW_TMR_NORMAL_MOD = 1,    // 标准模式
};
```

#### lowpower.c
主要功能：
1. **定时器注册系统** (`sys_timeout_reg`): 支持动态间隔计算
2. **睡眠时间管理** (`get_sleep_time`): 计算距离下次定时器的最短时间
3. **事件驱动循环** (`tcpip_timeouts_mbox_fetch`): 等待事件或超时
4. **定时器批量处理** (`check_timeout`): 唤醒后批量处理到期定时器

### 2.4 修改的核心文件

| 文件 | 修改内容 |
|------|----------|
| `src/api/tcpip.c` | 使用 `tcpip_timeouts_mbox_fetch` 替代标准循环 |
| `src/core/timeouts.c` | 条件编译排除标准定时器实现 |
| `src/core/tcp.c` | TCP 定时器支持动态间隔计算 |
| `src/include/lwip/timeouts.h` | 条件编译调整 |
| `src/include/lwip/priv/tcp_priv.h` | 添加 `tcp_fast_tmr_tick` 等函数 |
| `src/core/ipv4/etharp.c` | ARP 定时器支持动态间隔 |
| `src/core/ipv4/dhcp.c` | DHCP 定时器支持动态间隔 |
| `src/core/ipv6/nd6.c` | ND6 定时器支持动态间隔 |

### 2.5 关键代码变更示例

**TCP 定时器动态间隔计算**:
```c
// src/core/tcp.c
#if LWIP_LOWPOWER
u32_t tcp_fast_tmr_tick(void)
{
  // 根据实际需要计算下次触发间隔
  // 如果没有活跃连接，返回 0 (不触发)
  if (tcp_active_pcbs == NULL) {
    return 0;
  }
  return 1;  // 100ms 后触发
}
#endif
```

### 2.6 OH 价值

- **显著降低功耗**: CPU 可在网络空闲时深度睡眠
- **延长电池寿命**: 适合 IoT 传感器、可穿戴设备
- **兼容标准 API**: 应用层无需修改

### 2.7 回归风险

- **中**: 定时器系统是核心机制，需充分测试
- **中**: 某些实时性要求高的场景可能受影响
- **低**: 可通过 `LWIP_LOWPOWER=0` 完全禁用

---

## 3. 分布式网络 (LWIP_ENABLE_DISTRIBUTED_NET)

### 3.1 原始问题

标准 lwIP 是单设备网络栈，无法支持：
- 跨设备网络代理
- 分布式软总线
- 设备间透明网络访问

### 3.2 OH 解决方案

添加分布式网络层，实现跨设备 Socket 代理：

```
┌──────────────┐                    ┌──────────────┐
│   应用进程   │                    │   应用进程   │
│  (Device A)  │                    │  (Device B)  │
└──────┬───────┘                    └──────┬───────┘
       │ Socket API                       │ Socket API
       │                                  │
┌──────v───────┐                    ┌──────v───────┐
│ 分布式网络层 │◄────── 网络 ──────►│ 分布式网络层 │
│  (代理服务)  │      传输          │  (代理服务)  │
└──────┬───────┘                    └──────┬───────┘
       │                                  │
┌──────v───────┐                    ┌──────v───────┐
│   lwIP 栈   │                    │   lwIP 栈   │
└──────────────┘                    └──────────────┘
```

### 3.3 新增文件

| 文件 | 功能 |
|------|------|
| `distributed_net.h` | 分布式网络 API 头文件 |
| `distributed_net.c` | 主实现：Socket 状态管理、代理逻辑 |
| `distributed_net_core.h` | 核心数据结构定义 |
| `distributed_net_core.c` | 核心实现：连接管理、协议处理 |
| `distributed_net_utils.h` | 工具函数头文件 |
| `distributed_net_utils.c` | 工具函数实现 |
| `udp_transmit.h` | UDP 传输层头文件 |
| `udp_transmit.c` | UDP 传输实现 |

### 3.4 修改的文件

| 文件 | 修改内容 |
|------|----------|
| `src/api/sockets.c` | Socket API 中集成分布式网络钩子 |
| `src/core/dns.c` | DNS 请求支持分布式解析 |

### 3.5 关键 API

```c
// 启用分布式网络
int enable_distributed_net(u16_t tcp_port, u16_t udp_port);

// 禁用分布式网络
int disable_distributed_net(void);

// 设置 Socket 为分布式模式
void set_distributed_net_socket(int sock);

// 获取本地 TCP/UDP 代理端口
u16_t get_local_tcp_server_port(void);
u16_t get_local_udp_server_port(void);

// 检查分布式网络是否启用
u8_t is_distributed_net_enabled(void);
```

### 3.6 OH 价值

- **分布式软总线**: 为 DSoftBus 提供底层网络支持
- **透明跨设备访问**: 应用无需感知设备位置
- **灵活组网**: 支持星型、网状等多种拓扑

### 3.7 回归风险

- **低**: 功能相对独立，通过宏控制
- **低**: 不影响标准 Socket API 行为
- **中**: 与 DSoftBus 紧密耦合，需同步升级

---

## 4. 升级建议

### 4.1 推向上游的可行性

| 功能 | 可行性 | 原因 |
|------|--------|------|
| **低功耗模式** | ⭐⭐⭐ 高 | 通用功能，社区有需求 |
| **网络容器** | ⭐⭐ 中 | 概念通用，但实现依赖 LiteOS |
| **分布式网络** | ⭐ 低 | OH 特有功能，耦合度高 |

### 4.2 版本升级策略

1. **同步上游代码**: 使用 git merge 或 cherry-pick
2. **检查冲突文件**: 重点关注标记 `LOSCFG_NET_CONTAINER` 和 `LWIP_LOWPOWER` 的文件
3. **功能验证**: 每个 OH 特有功能需独立测试
4. **回归测试**: 标准 lwIP 功能需完整测试

### 4.3 文件变更统计

```bash
# OH 特有文件（无 upstream 对应）
src/core/net_group.c
src/core/lowpower.c
src/core/distributed_net/*.c
src/include/lwip/net_group.h
src/include/lwip/lowpower.h
src/include/lwip/distributed_net/*.h

# 修改的上游文件（需 merge 时关注）
src/core/netif.c      (+45 处 LOSCFG_NET_CONTAINER)
src/core/tcp.c        (+20 处条件编译)
src/core/udp.c        (+15 处条件编译)
src/core/ipv4/ip4.c   (+12 处条件编译)
src/core/ipv6/ip6.c   (+20 处条件编译)
```
