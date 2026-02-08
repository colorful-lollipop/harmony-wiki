# newip - 新 IP 协议

## 1. 概述

### 1.1 模块定位

New IP 在现有 IPv4/IPv6 能力基础上，以灵活轻量级报头和可变长多语义地址为基础，通过二三层协议融合，对协议去冗和压缩，减少冗余字节，实现高能效比、高净吞吐，提升通信效率。

**证据来源**: `newip/README_zh.md:1-8`

```
newip/README_zh.md:1-8
NewIP支持可变长多语义地址（最短1字节），可变长定制化报头封装（最短5字节）。
报头开销相比IPv4节省25.9%，相比IPv6节省44.9%。
载荷传输效率，相比IPv4提高最少1%，相比IPv6提高最少2.33%。
```

### 1.2 效率对比

| 场景 | 报头开销 | 载荷传输效率 |
|------|----------|--------------|
| IPv4 for WiFi | 30+8+20=58 B | 96.13% |
| IPv6 for WiFi | 30+8+40=78 B | 94.8% |
| New IP for WiFi | 30+8+5=43 B | 97.13% |

**证据来源**: `newip/README_zh.md:22-27`

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/newip/
├── examples/                   # 用户态示例代码
│   ├── nip_udp_server_demo.c
│   ├── nip_udp_client_demo.c
│   ├── nip_tcp_server_demo.c
│   ├── nip_tcp_client_demo.c
│   └── nip_lib.c/h
├── src/                        # 自研代码
│   ├── common/                 # 通用代码
│   │   ├── nip_addr.c/h        # 地址操作
│   │   ├── nip_hdr_encap.c     # 报头封装
│   │   ├── nip_hdr_decap.c     # 报头解封装
│   │   └── nip_checksum.c/h    # 校验和
│   └── linux-5.10/
│       ├── net/newip/
│       │   └── tcp_nip_parameter.c
│       └── drivers/net/bt/
├── third_party/               # 三方引用+增量开发
│   └── linux-5.10/
│       ├── include/
│       │   ├── linux/nip.h
│       │   ├── net/nip.h, nip_fib.h, nip_route.h
│       │   └── uapi/linux/nip.h, nip_icmp.h
│       └── net/newip/
│           ├── af_ninet.c     # Socket 族实现
│           ├── nip_input.c    # 报文输入处理
│           ├── nip_output.c   # 报文输出处理
│           ├── udp.c          # UDP 实现
│           ├── tcp_nip.c      # TCP 核心实现
│           ├── route.c        # 路由实现
│           └── nip_fib.c      # FIB 实现
├── figures/                    # 文档图例
└── tools/                      # 配套工具
```

**证据来源**: `newip/README_zh.md:34-59`

---

## 3. 协议设计

### 3.1 可变长报头

| 属性 | 值 |
|------|-----|
| 最小报头 | 5 字节 |
| 最大报头 | 24 字节 |
| 默认 TTL | 128 |

**位图字段**:
- Byte 1: valid bit, TTL, total_len, next_hdr, daddr, saddr, have_byte2
- Byte 2: hdr_len, reserved, have_byte3

### 3.2 可变长地址

| 属性 | 值 |
|------|-----|
| 地址长度 | 1-8 字节（最短 1 字节） |
| 最大地址长度 | 64 位 |

### 3.3 协议常量

| 常量 | 值 | 说明 |
|------|-------|------|
| `ETH_P_NEWIP` | - | NewIP 以太网类型 |
| `IPPROTO_NIP_ICMP` | 0xB1 | NewIP ICMP 协议号 |
| `AF_NINET` | - | NewIP 地址族 |
| `NIP_MIN_MTU` | 44 字节 | 最小 MTU |

---

## 4. 核心数据结构

### 4.1 协议处理结构

| 结构 | 文件 | 说明 |
|------|------|------|
| `struct ninet_protocol` | nip.h | 协议处理器 |
| `struct nip_hdr_decap` | nip_hdr.h | 解封装报头信息 |
| `struct nip_rt_info` | nip_fib.h | 路由信息 |
| `struct nip_fib_table` | nip_fib.h | FIB 表 |
| `struct ninet_dev` | if_ninet.h | 网络设备 |
| `struct ninet_ifaddr` | nip_addrconf.h | 接口地址 |

### 4.2 Socket 接口

| 操作 | 说明 |
|------|------|
| `ninet_create()` | 创建 Socket |
| `ninet_bind()` | 绑定地址 |
| `ninet_listen()` | 监听连接 |
| `__ninet_stream_connect()` | 连接 |
| `ninet_dgram_ops` | 数据报操作 |
| `ninet_stream_ops` | 流操作 |

---

## 5. 关键函数

### 5.1 报文处理

| 函数 | 文件 | 功能 |
|------|------|------|
| `nip_rcv()` | nip_input.c | 主报文接收 |
| `nip_input()` | nip_input.c | 输入处理 |
| `nip_output()` | nip_output.c | 输出处理 |
| `nip_hdr_parse()` | nip_hdr_decap.c | 报头解析 |

### 5.2 传输层

| 函数 | 文件 | 功能 |
|------|------|------|
| `nip_udp_input()` | udp.c | UDP 输入 |
| `nip_udp_output()` | udp.c | UDP 输出 |
| `tcp_nip_connect()` | tcp_nip.c | TCP 连接 |
| `tcp_nip_recvmsg()` | tcp_nip.c | TCP 接收 |
| `tcp_nip_sendmsg()` | tcp_nip.c | TCP 发送 |

### 5.3 路由

| 函数 | 文件 | 功能 |
|------|------|------|
| `nip_route_input()` | route.c | 路由输入查找 |
| `nip_route_output()` | route.c | 路由输出查找 |
| `nip_fib_add()` | nip_fib.c | 添加 FIB 条目 |
| `nip_fib_del()` | nip_fib.c | 删除 FIB 条目 |

---

## 6. 安全机制

### 6.1 地址验证

| 函数 | 功能 |
|------|------|
| `nip_addr_invalid()` | 检查地址有效性 |
| `nip_addr_public()` | 检查是否为公网地址 |
| `nip_bind_addr_check()` | 验证绑定地址 |

### 6.2 输入验证

- 报头位图有效性检查
- 报头长度验证
- 地址长度验证
- 报文长度与 skb->len 匹配检查
- UDP 校验和验证

### 6.3 Socket 安全

| 机制 | 说明 |
|------|------|
| 端口绑定限制 | < `PROT_SOCK` 需要 `CAP_NET_BIND_SERVICE` |
| Socket 限制 | `NIP_MAX_SOCKET_NUM = 1024` |
| Socket 选项验证 | 验证 sockopt level 和 name |

### 6.4 TCP 安全

| 机制 | 说明 |
|------|------|
| Challenge ACK | 速率限制的挑战 ACK 防止盲注 |
| PAWS | 防序列号回绕 |
| 窗口外报文处理 | 丢弃接收窗口外的报文 |

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [NewIP 开发手册](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/kernel/kernel-standard-newip.md) | 详细协议文档 |
| [04_Security_Review.md](../04_Security_Review.md) | 安全风险评审 |
