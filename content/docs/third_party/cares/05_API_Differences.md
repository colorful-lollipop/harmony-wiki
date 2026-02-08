# API/接口差异

**概述**: OpenHarmony 对 c-ares 的 API 扩展和行为变更说明。

---

## 新增公开 API

### ares_set_dns_netid()

**功能**: 为 DNS 查询通道绑定特定的网络 ID，支持多网络环境下的域名解析。

**声明位置**: `include/ares.h` 第 522-523 行

**函数原型**:
```c
/**
 * Set the network ID for DNS resolution.
 *
 * @param channel The channel to configure
 * @param netId   The network ID to bind (0 for default)
 */
CARES_EXTERN void ares_set_dns_netid(ares_channel_t *channel, int32_t netId);
```

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| **channel** | `ares_channel_t*` | 已初始化的 c-ares 通道 |
| **netId** | `int32_t` | 网络标识符，0 表示默认网络 |

**返回值**: 无（void）

**使用场景**:

1. **WiFi 网络**: 绑定到 WiFi 连接
   ```c
   ares_set_dns_netid(channel, wifi_netid);
   ```

2. **蜂窝数据**: 绑定到移动数据连接
   ```c
   ares_set_dns_netid(channel, cellular_netid);
   ```

3. **动态切换**: 网络状态变化时更新
   ```c
   void on_network_changed(int32_t new_netid) {
       ares_set_dns_netid(channel, new_netid);
   }
   ```

**注意事项**:
- ⚠️ **OH 特定 API**: 此 API 不在上游 c-ares 中，升级版本时需重新适配
- ⚠️ **线程安全**: 调用此函数时需确保通道已初始化且未被并发使用
- ⚠️ **生效时机**: netId 修改会影响后续所有查询，已发送的查询不受影响

**实现细节**:

netId 存储在 `ares_channeldata` 结构体中：

```c
// src/lib/ares_private.h
struct ares_channeldata {
  // ... 其他字段 ...
  int32_t netId;  // OH 新增字段
  // ... 其他字段 ...
};
```

在 NetSys DNS 配置时使用：

```c
// src/lib/ares_sysconfig.c
static ares_status_t ares_init_sysconfig_netsys(const ares_channel_t *channel,
                                                 ares_sysconfig_t *sysconfig)
{
#ifdef HAS_NETMANAGER_BASE
  int netid = channel->netId;
#else
  int netid = 0;
#endif
  // 使用 netid 调用 NetSysGetResolvConfExt()
  ...
}
```

---

## 内部接口变更

### 缓存接口

#### ares_get_dns_cache()

**功能**: 查询 OpenHarmony 系统级 DNS 缓存。

**声明**: 声明在 `src/lib/ares_getaddrinfo.c`，实现在 OH DNS 缓存模块中。

**函数原型**:
```c
int ares_get_dns_cache(const char *name, unsigned short port, int ai_family,
    struct ares_addrinfo_node **addr_info);
```

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| **name** | `const char*` | 要查询的域名 |
| **port** | `unsigned short` | 端口号（0 表示不限制） |
| **ai_family** | `int` | 地址族（AF_INET/AF_INET6） |
| **addr_info** | `struct ares_addrinfo_node**` | 输出参数，接收缓存结果 |

**返回值**:
| 值 | 说明 |
|-----|------|
| **0** | 缓存命中，addr_info 已填充 |
| **非 0** | 缓存未命中或错误 |

**调用位置**:
- 在 `ares_getaddrinfo_int()` 开始时调用
- 在 `OHOS_DNS_PROXY_BY_NETSYS` 条件块内

**伪代码**:
```c
#if OHOS_DNS_PROXY_BY_NETSYS
#ifdef HAS_NETMANAGER_BASE
  struct ares_addrinfo_node *cache_addr = NULL;
  if (ares_get_dns_cache(name, port, ai_family, &cache_addr) == 0) {
    // 缓存命中，直接返回
    ares_record_process(ARES_SUCCESS, name, start_time, cache_addr, NULL);
    // ... 返回缓存结果 ...
  }
#endif
#endif
```

#### ares_set_dns_cache()

**功能**: 将 DNS 查询结果存储到 OpenHarmony 系统级缓存。

**函数原型**:
```c
int ares_set_dns_cache(const char *name, unsigned short port, int ai_family,
    const struct ares_addrinfo_node *addr_info);
```

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| **name** | `const char*` | 域名 |
| **port** | `unsigned short` | 端口号 |
| **ai_family** | `int` | 地址族 |
| **addr_info** | `struct ares_addrinfo_node*` | 要缓存的地址信息 |

**返回值**:
| 值 | 说明 |
|-----|------|
| **0** | 成功缓存 |
| **非 0** | 缓存失败 |

**调用位置**:
- 在 `end_hquery()` 函数中，DNS 查询完成后调用

**伪代码**:
```c
#if OHOS_DNS_PROXY_BY_NETSYS
#ifdef HAS_NETMANAGER_BASE
  // 将查询结果写入缓存
  ares_set_dns_cache(hquery->name, hquery->port, hquery->ai_family,
                    hquery->ai);
#endif
#endif
```

### 指标收集接口

#### ares_record_process()

**功能**: 收集 DNS 查询性能指标并发送到 OH 遥测系统。

**函数原型**:
```c
void ares_record_process(int status, const char *hostname, long long start_time,
    const struct ares_addrinfo *addr_info, struct host_query *hquery);
```

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| **status** | `int` | 查询状态码 |
| **hostname** | `const char*` | 查询的域名 |
| **start_time** | `long long` | 查询开始时间戳 |
| **addr_info** | `const struct ares_addrinfo*` | 查询结果（可能为 NULL） |
| **hquery** | `struct host_query*` | 查询上下文（可能为 NULL） |

**调用场景**:
1. **缓存命中**: 当从系统缓存获取结果时
2. **查询失败**: 当 DNS 查询失败时
3. **查询成功**: 当 DNS 查询成功完成时

**数据结构**:

指标通过以下结构体传递：

```c
// src/lib/ares_private.h
struct ares_family_query_info {
  int retCode;
  char *serverAddr;
  unsigned char isNoAnswer;
  unsigned char cname;
};

struct ares_process_info {
  long long queryTime;
  char *hostname;
  int retCode;
  int firstQueryEndDuration;
  int firstQueryEnd2AppDuration;
  unsigned short firstReturnType;
  unsigned char isFromCache;
  unsigned char sourceFrom;
  struct ares_family_query_info ipv4QueryInfo;
  struct ares_family_query_info ipv6QueryInfo;
};
```

---

## NetSys 接口

### NetSysGetResolvConfExt()

**功能**: 从 OpenHarmony NetSys 服务获取 DNS 服务器配置。

**声明**: 声明在 `src/lib/ares_sysconfig.c`，实现在 OH NetSys 模块中。

**函数原型**:
```c
int32_t NetSysGetResolvConfExt(uint16_t netid, struct resolv_config *config);
```

**参数说明**:
| 参数 | 类型 | 说明 |
|-----|------|------|
| **netid** | `uint16_t` | 网络标识符 |
| **config** | `struct resolv_config*` | 输出参数，接收 DNS 配置 |

**返回值**:
| 值 | 说明 |
|-----|------|
| **>= 0** | 成功获取配置 |
| **< 0** | 获取失败 |

**配置结构**:
```c
struct resolv_config {
  int32_t error;
  int32_t timeout_ms;
  uint32_t retry_count;
  uint32_t non_public;
  char nameservers[MAX_SERVER_NUM][MAX_SERVER_LENGTH + 1];
};
```

**使用位置**:
- 在 `ares_init_sysconfig_netsys()` 函数中调用

**调用链**:
```
ares_init()
  └─> ares_init_by_sysconfig()
       └─> ares_init_sysconfig_netsys()
             └─> NetSysGetResolvConfExt()
```

---

## 行为变更

### 1. EDNS 默认禁用

**标准行为**: EDNS (Extension Mechanisms for DNS) 默认启用

**OH 行为**: EDNS 默认禁用

**代码位置**: `src/lib/ares_init.c` 第 132-140 行

```c
#ifndef OHOS_DNS_PROXY_BY_NETSYS
  /* Enable EDNS by default */
  if (!(channel->optmask & ARES_OPT_FLAGS)) {
    channel->flags = ARES_FLAG_EDNS;
  }
  if (channel->ednspsz == 0) {
    channel->ednspsz = EDNSPACKETSZ;
  }
#endif
```

**影响**:
- ⚠️ **DNSSEC 不可用**: DNSSEC 需要依赖 EDNS
- ⚠️ **大响应截断**: EDNS 0 支持更大的 UDP 响应包

**启用方法**: 如需 EDNS 支持，可通过选项显式启用：
```c
struct ares_options options;
memset(&options, 0, sizeof(options));
options.flags = ARES_FLAG_EDNS;
ares_init_options(&channel, &options, ARES_OPT_FLAGS);
```

### 2. 重试次数调整

**标准行为**: 默认重试次数为 3 次

**OH 行为**: 默认重试次数为 2 次

**代码位置**: `src/lib/ares_init.c` 第 146-152 行

```c
if (channel->tries == 0) {
#if OHOS_DNS_PROXY_BY_NETSYS
    channel->tries = 2; // change default tries from 3 to 2
#else
    channel->tries = DEFAULT_TRIES;
#endif
}
```

**影响**:
- ✅ **减少延迟**: 少一次重试，快速失败或成功
- ⚠️ **成功率略降**: 网络不稳定时可能降低成功率

**自定义方法**:
```c
// 设置自定义重试次数
ares_set_maxtries(channel, 3);  // 恢复默认值
ares_set_maxtries(channel, 5);  // 增加重试
```

### 3. 超时计算

**标准行为**: 根据历史延迟动态计算超时时间

**OH 行为**: 使用通道配置的固定超时值

**代码位置**: `src/lib/ares_process.c` 第 1325-1327 行

```c
#if OHOS_DNS_PROXY_BY_NETSYS
  timeout = channel->timeout;  // 直接使用通道配置的超时
#else
  // 根据历史数据动态计算超时
  timeout = ares_metrics_server_timeout(server, now);
#endif
```

**影响**:
- ⚠️ **失去自适应**: 不能根据网络质量动态调整
- ✅ **简化逻辑**: 避免与 NetSys 超时策略冲突

---

## 废弃或禁用的功能

### 未实现的上游功能

| 功能 | 上游支持 | OH 状态 | 原因 |
|-----|----------|---------|------|
| **CLI 工具** | adig, ahost, acountry | ❌ 不构建 | OH 应用不使用 CLI |
| **符号隐藏** | CARES_SYMBOL_HIDING | ❌ 未启用 | 未配置 |
| **编译覆盖率** | CARES_COVERAGE | ❌ 未支持 | 无测试需求 |

### 条件编译的功能

| 功能 | 编译条件 | OH 配置 |
|-----|----------|---------|
| **NetSys DNS 代理** | `OHOS_DNS_PROXY_BY_NETSYS` | 需定义此宏 |
| **NetManager 集成** | `HAS_NETMANAGER_BASE` | 需定义此宏 |
| **DNS 缓存** | 两者都定义 | 需同时定义 |
| **指标收集** | 两者都定义 | 需同时定义 |

---

## 数据结构扩展

### ares_channeldata 扩展

**新增字段**: `netId`

```c
// src/lib/ares_private.h
struct ares_channeldata {
  // ... 原有字段 ...

  int32_t netId;  // ← OH 新增

  // ... 其他字段 ...
};
```

**用途**: 存储当前通道绑定的网络 ID。

### 新增内部结构体

#### ares_family_query_info

```c
struct ares_family_query_info {
  int retCode;                // 返回码
  char *serverAddr;           // DNS 服务器地址
  unsigned char isNoAnswer;   // 是否无应答
  unsigned char cname;          // CNAME 信息
};
```

#### ares_process_info

```c
struct ares_process_info {
  long long queryTime;                     // 查询耗时
  char *hostname;                         // 查询的域名
  int retCode;                            // 返回码
  int firstQueryEndDuration;                // 首次查询完成时间
  int firstQueryEnd2AppDuration;             // 首次查询到应用耗时
  unsigned short firstReturnType;              // 首次返回类型（A/AAAA）
  unsigned char isFromCache;                // 是否来自缓存
  unsigned char sourceFrom;                 // 来源标识
  struct ares_family_query_info ipv4QueryInfo;   // IPv4 查询信息
  struct ares_family_query_info ipv6QueryInfo;   // IPv6 查询信息
};
```

---

## 宏定义

### OH 特定宏

| 宏名 | 作用 | 定义位置 |
|-----|------|---------|
| `OHOS_DNS_PROXY_BY_NETSYS` | 启用 NetSys DNS 代理功能 | 编译选项 |
| `HAS_NETMANAGER_BASE` | 启用 NetManager 集成 | 编译选项 |

### 宏使用统计

**`OHOS_DNS_PROXY_BY_NETSYS` 使用位置**:
- `src/lib/ares_sysconfig.c`: 2 处
- `src/lib/ares_init.c`: 2 处
- `src/lib/ares_getaddrinfo.c`: 11 处
- `src/lib/ares_process.c`: 1 处
- `src/lib/ares_private.h`: 1 处

**总计**: 17 处条件编译

---

## 升级兼容性

### 向后兼容性

| API | 兼容性 | 说明 |
|-----|--------|------|
| 标准 c-ares API | ✅ 完全兼容 | 所有上游 API 可用 |
| `ares_set_dns_netid()` | ⚠️ OH 特定 | 升级时需重新适配 |
| 内部缓存接口 | ⚠️ OH 特定 | 升级时需重新适配 |
| NetSys 接口 | ⚠️ OH 特定 | 升级时需重新适配 |

### 升级检查清单

升级到新版本时，需检查：

- [ ] OH 特定 Patch 是否已合并
- [ ] `ares_set_dns_netid()` API 是否保留
- [ ] NetSys 集成代码是否冲突
- [ ] `OHOS_DNS_PROXY_BY_NETSYS` 宏是否仍有效
- [ ] 缓存接口签名是否变化
- [ ] 指标结构体是否兼容

---

## 总结

### API 差异总结

| 类别 | 数量 | 主要影响 |
|-----|------|---------|
| **新增公开 API** | 1 | `ares_set_dns_netid()` |
| **新增内部接口** | 3 | 缓存和指标接口 |
| **行为变更** | 3 | EDNS、重试、超时 |
| **数据结构扩展** | 2 | channel 和指标结构体 |
| **条件编译块** | 17 | OH 特定功能 |

### 使用建议

1. **优先使用 getaddrinfo**: `ares_getaddrinfo()` 是现代 API，推荐优先使用
2. **合理使用 netId**: 仅在多网络环境需要时使用
3. **注意 EDNS 限制**: OH 默认禁用 EDNS，需显式启用
4. **监控指标**: 利用 `ares_record_process()` 收集的性能指标进行优化

---

**最后更新**: 2026-02-08
