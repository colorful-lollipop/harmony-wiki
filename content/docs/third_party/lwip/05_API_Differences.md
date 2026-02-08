# API/接口差异

## 概述

OpenHarmony 的 lwIP 在保持标准 API 兼容的同时，添加了三类特有接口：

1. **网络容器 API**: 支持网络命名空间隔离
2. **低功耗 API**: 支持定时器睡眠管理
3. **分布式网络 API**: 支持跨设备网络代理

## 标准 API 兼容性

### BSD Socket API

完全兼容，无需修改：

```c
#include "lwip/sockets.h"

int socket(int domain, int type, int protocol);
int bind(int s, const struct sockaddr *name, socklen_t namelen);
int listen(int s, int backlog);
int accept(int s, struct sockaddr *addr, socklen_t *addrlen);
int connect(int s, const struct sockaddr *name, socklen_t namelen);
ssize_t send(int s, const void *dataptr, size_t size, int flags);
ssize_t recv(int s, void *mem, size_t len, int flags);
int close(int s);
// ... 其他标准 Socket API
```

### Netconn API

完全兼容：

```c
#include "lwip/netconn.h"

struct netconn *netconn_new(enum netconn_type t);
err_t netconn_bind(struct netconn *conn, const ip_addr_t *addr, u16_t port);
err_t netconn_listen(struct netconn *conn);
err_t netconn_accept(struct netconn *conn, struct netconn **new_conn);
err_t netconn_connect(struct netconn *conn, const ip_addr_t *addr, u16_t port);
err_t netconn_send(struct netconn *conn, struct netbuf *buf);
err_t netconn_recv(struct netconn *conn, struct netbuf **new_buf);
err_t netconn_delete(struct netconn *conn);
```

### Raw API

完全兼容：

```c
#include "lwip/tcp.h"
#include "lwip/udp.h"
#include "lwip/raw.h"

// TCP Raw API
struct tcp_pcb *tcp_new(void);
err_t tcp_bind(struct tcp_pcb *pcb, const ip_addr_t *ipaddr, u16_t port);
err_t tcp_listen(struct tcp_pcb *pcb);
err_t tcp_connect(struct tcp_pcb *pcb, const ip_addr_t *ipaddr, u16_t port);
void tcp_sent(struct tcp_pcb *pcb, tcp_sent_fn sent);
void tcp_recv(struct tcp_pcb *pcb, tcp_recv_fn recv);
err_t tcp_write(struct tcp_pcb *pcb, const void *dataptr, u16_t len, u8_t apiflags);
err_t tcp_output(struct tcp_pcb *pcb);
void tcp_close(struct tcp_pcb *pcb);

// UDP Raw API
struct udp_pcb *udp_new(void);
err_t udp_bind(struct udp_pcb *pcb, const ip_addr_t *ipaddr, u16_t port);
err_t udp_connect(struct udp_pcb *pcb, const ip_addr_t *ipaddr, u16_t port);
err_t udp_send(struct udp_pcb *pcb, struct pbuf *p);
err_t udp_recv(struct udp_pcb *pcb, udp_recv_fn recv, void *recv_arg);
void udp_remove(struct udp_pcb *pcb);
```

---

## OH 特有 API

### 1. 网络容器 API

**头文件**: `lwip/net_group.h`

**条件编译**: `#ifdef LOSCFG_NET_CONTAINER`

#### 数据结构

```c
/**
 * 网络组结构
 * 代表一个独立的网络命名空间
 */
struct net_group {
    u8_t netif_num;              // 网卡数量
    struct netif *netif_default; // 默认网卡
    struct netif *netif_list;    // 网卡列表
    struct netif *loop_netif;    // 回环网卡
};

/**
 * 网络组操作函数表
 * 用于内核实现网络容器隔离
 */
struct net_group_ops {
    // 获取当前进程所属的网络组
    struct net_group *(*get_curr_process_net_group)(void);
    
    // 设置网卡所属的网络组
    void (*set_netif_net_group)(struct netif *, struct net_group *);
    
    // 从网卡获取所属网络组
    struct net_group *(*get_net_group_from_netif)(struct netif *);
    
    // 设置 IP PCB 所属的网络组
    void (*set_ippcb_net_group)(struct ip_pcb *, struct net_group *);
    
    // 从 IP PCB 获取所属网络组
    struct net_group *(*get_net_group_from_ippcb)(struct ip_pcb *);
};
```

#### API 函数

```c
/**
 * 获取根网络组（默认网络组）
 * @return 根网络组指针
 */
struct net_group *get_root_net_group(void);

/**
 * 获取当前进程所属的网络组
 * 通过注册的 ops 回调实现
 * @return 当前进程的网络组指针
 */
struct net_group *get_curr_process_net_group(void);

/**
 * 获取默认的网络组操作函数表
 * @return 默认 ops 指针
 */
struct net_group_ops *get_default_net_group_ops(void);

/**
 * 设置默认的网络组操作函数表
 * 内核通过此函数注入自定义实现
 * @param ops 新的操作函数表
 */
void set_default_net_group_ops(struct net_group_ops *ops);
```

#### 使用示例

```c
// 内核实现网络容器
static struct net_group *my_get_curr_process_net_group(void)
{
    // 根据当前进程 ID 返回对应的网络组
    pid_t pid = get_current_pid();
    return get_net_group_by_pid(pid);
}

static struct net_group_ops my_ops = {
    .get_curr_process_net_group = my_get_curr_process_net_group,
    .set_netif_net_group = my_set_netif_net_group,
    .get_net_group_from_netif = my_get_net_group_from_netif,
    .set_ippcb_net_group = my_set_ippcb_net_group,
    .get_net_group_from_ippcb = my_get_net_group_from_ippcb,
};

// 注册自定义实现
void init_network_container(void)
{
    set_default_net_group_ops(&my_ops);
}
```

---

### 2. 低功耗 API

**头文件**: `lwip/lowpower.h`

**条件编译**: `#if LWIP_LOWPOWER`

#### 数据结构

```c
/**
 * 低功耗模式
 */
enum lowpower_mod {
    LOW_TMR_LOWPOWER_MOD = 0,  // 启用低功耗模式
    LOW_TMR_NORMAL_MOD = 1,    // 标准模式（不休眠）
};

/**
 * 定时器状态
 */
enum timer_state {
    LOW_TMR_GETING_TICKS = 0,   // 正在计算下次触发时间
    LOW_TMR_TIMER_WAITING,       // 等待触发
    LOW_TMR_TIMER_HANDLING,      // 处理定时器
};

/**
 * 定时器处理函数结构
 */
struct timer_handler {
    u32_t interval;              // 基础间隔 (ms)
    lwip_timer_handler handler;  // 处理函数
    get_next_timeout next_tick;  // 动态间隔计算函数
#if LOWPOWER_TIMER_DEBUG
    char *name;                  // 调试名称
#endif
};

/**
 * 定时器条目
 */
struct timer_entry {
    u32_t clock_max;             // 定时器周期（以 TIMEOUT_TICK 为单位）
    u32_t timeout;               // 上次触发时间
    sys_timeout_handler handler; // 超时处理函数
    void *args;                  // 参数
    get_next_timeout next_tick;  // 动态间隔计算函数
    struct timer_entry *next;    // 链表指针
    u8_t enable;                 // 是否启用
};
```

#### API 函数

```c
/**
 * 设置低功耗模式
 * @param sw 模式：LOW_TMR_LOWPOWER_MOD 或 LOW_TMR_NORMAL_MOD
 */
void set_lowpower_mod(enum lowpower_mod sw);

/**
 * 获取当前低功耗模式
 * @return 当前模式
 */
enum lowpower_mod get_lowpowper_mod(void);

/**
 * 设置唤醒时间（外部事件触发）
 * 用于外部模块通知 lwIP 需要提前唤醒
 * @param val 唤醒时间（毫秒，相对于当前时间）
 */
void sys_timeout_set_wake_time(u32_t val);

/**
 * 获取当前设置的唤醒时间
 * @return 唤醒时间
 */
u32_t sys_timeout_get_wake_time(void);

/**
 * 检查是否需要长时间等待
 * 用于外部模块判断是否可以让 CPU 睡眠
 * @return 1=可以长时间等待，0=需要立即处理
 */
u8_t sys_timeout_waiting_long(void);

/**
 * 初始化所有系统定时器
 * 替代标准 lwIP 的 sys_timeouts_init
 */
void sys_timeouts_init(void);

/**
 * 注册定时器
 * 低功耗模式下的定时器注册接口
 * @param msec 间隔（毫秒）
 * @param handler 处理函数
 * @param arg 参数
 * @param name 名称（调试时）
 * @param next_tick 动态间隔计算函数
 * @return 0=成功，其他=失败
 */
err_t sys_timeout_reg(
    u32_t msec,
    sys_timeout_handler handler,
    void *arg,
#if LOWPOWER_TIMER_DEBUG
    char *name,
#endif
    get_next_timeout next_tick);

/**
 * 移除定时器
 * @param handler 处理函数
 * @param arg 参数
 */
void sys_untimeout(sys_timeout_handler handler, void *arg);

/**
 * 带超时处理的邮箱获取
 * TCP/IP 线程主循环使用，支持睡眠唤醒
 * @param mbox 邮箱
 * @param msg 接收到的消息
 */
void tcpip_timeouts_mbox_fetch(sys_mbox_t *mbox, void **msg);
```

#### TCP 定时器动态间隔函数

```c
// 获取 TCP 快速定时器下次触发间隔
u32_t tcp_fast_tmr_tick(void);

// 获取 TCP 慢速定时器下次触发间隔
u32_t tcp_slow_tmr_tick(void);
```

#### 使用示例

```c
// 启用低功耗模式
set_lowpower_mod(LOW_TMR_LOWPOWER_MOD);

// TCP/IP 线程主循环使用低功耗接口
void tcpip_thread(void *arg)
{
    while (1) {
        void *msg;
        // 此函数会让线程在消息到达或定时器到期前睡眠
        tcpip_timeouts_mbox_fetch(mbox, &msg);
        
        if (msg != NULL) {
            // 处理消息
        }
    }
}

// 外部事件（如中断）需要提前唤醒时
void external_event_handler(void)
{
    // 设置 10ms 后唤醒处理
    sys_timeout_set_wake_time(10);
}
```

---

### 3. 分布式网络 API

**头文件**: `lwip/distributed_net/distributed_net.h`

**条件编译**: `#if LWIP_ENABLE_DISTRIBUTED_NET`

#### API 函数

```c
/**
 * 设置 Socket 为分布式模式
 * 此后该 Socket 的数据会被代理到远程设备
 * @param sock Socket 文件描述符
 */
void set_distributed_net_socket(int sock);

/**
 * 重置 Socket 的分布式模式
 * @param sock Socket 文件描述符
 */
void reset_distributed_net_socket(int sock);

/**
 * 获取本地 TCP 代理服务器端口
 * @return TCP 代理端口
 */
u16_t get_local_tcp_server_port(void);

/**
 * 获取本地 UDP 代理服务器端口
 * @return UDP 代理端口
 */
u16_t get_local_udp_server_port(void);

/**
 * 检查分布式网络是否已启用
 * @return 1=启用，0=未启用
 */
u8_t is_distributed_net_enabled(void);

/**
 * 启用分布式网络
 * 启动本地代理服务器，准备接收远程设备连接
 * @param tcp_port TCP 代理端口
 * @param udp_port UDP 代理端口
 * @return 0=成功，其他=失败
 */
int enable_distributed_net(u16_t tcp_port, u16_t udp_port);

/**
 * 禁用分布式网络
 * 关闭代理服务器，断开所有远程连接
 * @return 0=成功，其他=失败
 */
int disable_distributed_net(void);
```

#### 使用示例

```c
#include "lwip/distributed_net/distributed_net.h"

// 启用分布式网络
int ret = enable_distributed_net(8080, 8081);
if (ret != 0) {
    printf("Failed to enable distributed net\n");
    return -1;
}

// 创建 Socket 并设置为分布式模式
int sock = socket(AF_INET, SOCK_STREAM, 0);
set_distributed_net_socket(sock);

// 连接到远程服务（实际会被代理到远程设备）
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(80);
addr.sin_addr.s_addr = inet_addr("192.168.1.100");
connect(sock, (struct sockaddr *)&addr, sizeof(addr));

// 后续的数据收发都是透明的
send(sock, data, len, 0);
recv(sock, buffer, sizeof(buffer), 0);

// 清理
close(sock);
disable_distributed_net();
```

---

## 配置宏

### 启用 OH 特有功能

```c
// lwipopts.h

// 启用低功耗模式
#define LWIP_LOWPOWER 1

// 启用分布式网络
#define LWIP_ENABLE_DISTRIBUTED_NET 1

// 网络容器由 kernel 定义 LOSCFG_NET_CONTAINER，无需在 lwipopts.h 中定义
```

### 调试配置

```c
// 启用低功耗定时器调试
#define LOWPOWER_TIMER_DEBUG 1

// 低功耗调试级别
#define LOWPOWER_DEBUG LWIP_DBG_ON
```

---

## 版本变更记录

| 版本 | API 变更 |
|------|----------|
| 2.1.3 | 基础版本，仅支持 LOSCFG_NET_CONTAINER |
| 2.2.1 | 新增 LWIP_LOWPOWER 低功耗 API |
| 2.2.1+ | 新增 LWIP_ENABLE_DISTRIBUTED_NET 分布式网络 API |
