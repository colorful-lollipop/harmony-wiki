# 02 - Patch 详细分析

本文档详细分析 OpenHarmony 对 libnl 的所有 Patch 修改。

## Patch 清单

| Patch 文件 | 目标版本 | 修改类型 | 当前状态 |
|-----------|---------|---------|---------|
| `solve-oh-compile-problem.patch` | 3.7.0 | 编译适配 | 已废弃 |
| `slove-oh-update-sp3.patch` | 3.7.0 | 安全加固/Bug修复/测试 | 已废弃 |
| `slove-oh-bug-fix.patch` | 3.7.0 | Bug修复 | 已废弃 |
| `solve-oh-compile-problem3_11_0.patch` | **3.11.0** | 编译适配/内核适配 | **当前使用** |

---

## Patch 1: solve-oh-compile-problem.patch (3.7.0)

**状态**: 历史版本，已被 3.11.0 Patch 替代

### 修改文件
- `include/netlink-private/utils.h`
- `lib/route/link/vrf.c`
- `lib/route/mdb.c`
- `lib/utils.c`
- `src/lib/utils.c`

### 修改摘要

#### 1.1 移除 typeof 宏使用
**文件**: `include/netlink-private/utils.h`

```c
// 修改前
#define _nl_assert_addr_family(addr_family)                    \
    do {                                                       \
        typeof(addr_family) _addr_family = (addr_family);      \
        _nl_assert(_addr_family == AF_INET ||                  \
               _addr_family == AF_INET6);                      \
    } while (0)

// 修改后
#define _nl_assert_addr_family(addr_family)                    \
    do {                                                       \
        _nl_assert(addr_family == AF_INET ||                   \
               addr_family == AF_INET6);                       \
    } while (0)
```

**修改目的**: 移除 `typeof` GCC 扩展，适配严格标准 C 编译环境。

#### 1.2 头文件路径调整
**文件**: `lib/route/mdb.c`, `lib/route/link/vrf.c`

```c
// 修改前
#include <linux/if_bridge.h>
#include <linux-private/linux/rtnetlink.h>

// 修改后
#include <linux-private/linux/if_bridge.h>
// (移除 rtnetlink.h)
```

**修改目的**: 使用 OpenHarmony 提供的私有内核头文件。

#### 1.3 变量初始化
**文件**: `lib/utils.c`

```c
// 修改前
char *unit;
double frac;

// 修改后
char *unit = NULL;
double frac = 0.0;
```

**修改目的**: 修复未初始化变量警告。

---

## Patch 2: solve-oh-compile-problem3_11_0.patch (3.11.0) ⭐

**状态**: **当前活跃 Patch**（用于 libnl 3.11.0）

### 修改概览

| 类别 | 修改文件数 | 说明 |
|------|-----------|------|
| 编译适配 | 4 | typeof、restrict、类型转换等 |
| 内核版本适配 | 3 | 条件编译控制新内核特性 |
| 头文件路径 | 2 | 使用私有内核头文件 |
| 库加载适配 | 1 | 禁用动态库加载 |

### 详细修改分析

#### 2.1 编译适配 - typeof 移除
**文件**: `include/base/nl-base-utils.h`

```c
// 修改前
#define _nl_assert_addr_family(addr_family)                       \
    do {                                                      \
        typeof(addr_family) _addr_family = (addr_family); \
        _nl_assert(_addr_family == AF_INET ||             \
               _addr_family == AF_INET6);             \
    } while (0)

// 修改后
#define _nl_assert_addr_family(addr_family)                       \
    do {                                                      \
        _nl_assert(addr_family == AF_INET ||             \
               addr_family == AF_INET6);             \
    } while (0)
```

**OH 价值**: 适配 OH 编译环境，移除 GCC 扩展。

#### 2.2 编译适配 - restrict 关键字
**文件**: `include/base/nl-base-utils.h`

```c
// 修改前
static inline void *_nl_memcpy(void *restrict dest, const void *restrict src,
                               size_t n)

// 修改后  
static inline void *_nl_memcpy(void *__restrict dest, const void *__restrict src,
                               size_t n)
```

**OH 价值**: 使用双下划线形式的 `__restrict`，兼容性更好。

#### 2.3 编译适配 - malloc 类型转换
**文件**: `include/base/nl-base-utils.h`

```c
// 修改前
return (char *)_nl_inet_ntop(addr_family, addr,
                             malloc((addr_family == AF_INET) ?
                                    INET_ADDRSTRLEN :
                                    INET6_ADDRSTRLEN));

// 修改后
return (char *)_nl_inet_ntop(addr_family, addr,
                             (char *)malloc((addr_family == AF_INET) ?
                                    INET_ADDRSTRLEN :
                                    INET6_ADDRSTRLEN));
```

**OH 价值**: 显式类型转换避免警告。

#### 2.4 编译适配 - ARRAY_SIZE 保护
**文件**: `include/base/nl-base-utils.h`

```c
#ifndef ARRAY_SIZE
#define _NL_N_ELEMENTS(arr) (sizeof(arr) / sizeof((arr)[0]))
#define ARRAY_SIZE(arr) _NL_N_ELEMENTS(arr)
#endif
```

**OH 价值**: 防止宏重复定义。

#### 2.5 编译适配 - 函数声明禁用
**文件**: `include/config.h`

```c
// 修改前
#define HAVE_DECL_GETPROTOBYNAME_R 1
#define HAVE_DECL_GETPROTOBYNUMBER_R 1
#define HAVE_STRERROR_L 1

// 修改后
#define HAVE_DECL_GETPROTOBYNAME_R 0
#define HAVE_DECL_GETPROTOBYNUMBER_R 0
#define HAVE_STRERROR_L 0
```

**OH 价值**: 禁用某些平台不支持的函数声明。

#### 2.6 内核版本适配 - 邻居表扩展标志
**文件**: `lib/route/neigh.c`

```c
+#ifdef OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION
    if (tb[NDA_FLAGS_EXT]) {
        neigh->n_ext_flags = nla_get_u32(tb[NDA_FLAGS_EXT]);
        neigh->ce_mask |= NEIGH_ATTR_EXT_FLAGS;
    }
+#endif
```

**OH 价值**: 使用宏控制新内核特性（`NDA_FLAGS_EXT`），确保与旧内核兼容。

#### 2.7 内核版本适配 - 网桥端口属性
**文件**: `lib/route/link/bridge.c`

```c
+#ifdef OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION
    [IFLA_BRPORT_LOCKED]            = { .type = NLA_U8 },
    [IFLA_BRPORT_MAB]               = { .type = NLA_U8 },
    [IFLA_BRPORT_NEIGH_VLAN_SUPPRESS] = { .type = NLA_U8 },
+#endif
```

**OH 价值**: 条件编译控制新网桥特性，避免在旧内核上编译失败。

#### 2.8 内核版本适配 - 缺失定义补充
**文件**: `include/base/nl-base-utils.h`

```c
#define NTF_EXT_MANAGED     (1 << 0)
#define NTF_EXT_LOCKED      (1 << 1)
#define NDA_MAX (__NDA_MAX - 1)
```

**OH 价值**: 补充旧内核头文件中缺失的定义。

#### 2.9 变量初始化
**文件**: `lib/utils.c`

```c
// 修改前
char *unit;
double frac;

// 修改后
char *unit = NULL;
double frac = 0.0;
```

#### 2.10 Bug 修复 - BUG() 后返回
**文件**: `lib/utils.c`

```c
    BUG();
+   return buf;
```

**OH 价值**: 修复控制流分析警告，虽然 `BUG()` 不会返回，但显式返回更安全。

#### 2.11 库加载适配
**文件**: `src/lib/utils.c`

```c
// 注释掉动态库加载路径构造
-   snprintf(path, sizeof(path), "%s/%s/%s.so",
-        _NL_PKGLIBDIR, prefix, name);
```

**OH 价值**: OpenHarmony 不使用 libnl 的动态库加载功能，注释掉相关代码。

---

## Patch 3: slove-oh-update-sp3.patch (3.7.0)

**状态**: 历史版本，包含大量测试代码和安全修复

### 安全修复摘要

| 类别 | 修改文件 | 修复内容 |
|------|---------|---------|
| 空指针检查 | `lib/attr.c` | `nla_nest_end()` 添加 NULL 检查 |
| 类型安全 | `lib/object.c` | 修复 `1 << 31` 有符号溢出 |
| 线程安全 | `lib/xfrm/*.c` | `gmtime()` → `gmtime_r()` |
| 内存安全 | `lib/xfrm/*.c` | 所有 `nl_addr_build()` 添加空检查 |
| 函数指针检查 | `lib/route/link.c` | 添加 `ao_override_rtm` 空检查 |
| 随机数安全 | `lib/socket.c` | 改进时间戳随机化 |
| 边界检查 | `lib/utils.c` | 协议名解析边界检查 |

### 关键安全修复详解

#### 3.1 线程安全 - gmtime_r
**文件**: `lib/xfrm/ae.c`, `sa.c`, `sp.c`

```c
// 修改前
add_time_tm = gmtime(&add_time);

// 修改后
struct tm tm_buf;
add_time_tm = gmtime_r(&add_time, &tm_buf);
```

**安全价值**: `gmtime()` 返回静态缓冲区指针，非线程安全；`gmtime_r()` 使用调用者提供的缓冲区，线程安全。

#### 3.2 内存安全 - nl_addr_build 空检查
**文件**: `lib/xfrm/sa.c`

```c
// 修改前
addr = nl_addr_build(sa_info->sel.family, &sa_info->sel.daddr.a4, 
                     sizeof(sa_info->sel.daddr.a4));

// 修改后
if (!(addr = nl_addr_build(sa_info->sel.family, &sa_info->sel.daddr.a4,
                           sizeof(sa_info->sel.daddr.a4)))) {
    err = -NLE_NOMEM;
    goto errout;
}
```

**安全价值**: 防止内存分配失败导致的空指针解引用。

#### 3.3 类型安全 - 移位操作
**文件**: `lib/object.c`

```c
// 修改前
? (uint32_t) diff | (1 << 31)

// 修改后  
? (uint32_t) diff | (((uint32_t) 1u) << 31)
```

**安全价值**: 避免有符号整数溢出未定义行为。

---

## Patch 4: slove-oh-bug-fix.patch (3.7.0)

**状态**: Bug 修复 Patch

### 修改内容

#### 4.1 序列号溢出保护
**文件**: `lib/socket.c`

```c
unsigned int nl_socket_use_seq(struct nl_sock *sk)
{
+   if (sk->s_seq_next == UINT_MAX) {
+       sk->s_seq_next = 0;
+       return UINT_MAX;
+   }
    return sk->s_seq_next++;
}
```

**Bug 描述**: 当序列号达到 `UINT_MAX` 时，`++` 操作会导致溢出回绕到 0，可能导致序列号冲突。

**修复价值**: 显式处理溢出情况，确保序列号单调递增（模 `UINT_MAX`）。

#### 4.2 序列号获取统一
**文件**: `lib/nl.c`

```c
// 修改前
nlh->nlmsg_seq = sk->s_seq_next++;

// 修改后
nlh->nlmsg_seq = nl_socket_use_seq(sk);
```

**修复价值**: 统一使用 `nl_socket_use_seq()` 获取序列号，确保溢出保护生效。

---

## 升级建议

### 推向上游可行性评估

| Patch | 推向上游可行性 | 说明 |
|-------|--------------|------|
| `solve-oh-compile-problem3_11_0.patch` 编译适配部分 | ✅ 推荐 | typeof 移除、restrict 修改等可提高可移植性 |
| `solve-oh-compile-problem3_11_0.patch` 内核适配部分 | ❌ OH 特有 | `OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION` 宏为 OH 特有 |
| `slove-oh-bug-fix.patch` 序列号溢出 | ✅ 强烈推荐 | 通用 Bug 修复，建议推向上游 |
| `slove-oh-update-sp3.patch` 安全修复 | ✅ 推荐 | gmtime_r、空指针检查等安全修复有价值 |
| `slove-oh-update-sp3.patch` 测试代码 | ❌ OH 特有 | 为 OH 测试框架编写 |

### 版本升级注意事项

1. **保留当前 Patch**: 升级上游版本时，`solve-oh-compile-problem3_11_0.patch` 需要重新适配
2. **内核版本匹配**: 检查 `OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION` 宏控制的特性是否与目标内核匹配
3. **API 兼容性**: 验证依赖模块（wpa_supplicant、WLAN 驱动）的兼容性
4. **安全修复同步**: 确保 SP3 中的安全修复在新版本中已包含或重新应用
