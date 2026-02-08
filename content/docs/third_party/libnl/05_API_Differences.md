# 05 - API/接口差异

本文档说明 OpenHarmony 版本 libnl 与上游版本的 API 差异。

## 概述

OpenHarmony 版本的 libnl 主要保留了上游的公共 API，但在以下方面存在差异：

1. **条件编译控制**: 某些功能通过宏控制，根据内核版本启用
2. **内部实现差异**: 部分内部函数实现经过安全加固
3. **禁用功能**: 动态库加载功能被禁用

## 条件编译控制的 API

### OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION 宏

该宏控制与新内核版本相关的功能。定义此宏时，启用以下特性：

#### 1. 邻居表扩展标志 (NDA_FLAGS_EXT)

**文件**: `lib/route/neigh.c`

```c
#ifdef OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION
    if (tb[NDA_FLAGS_EXT]) {
        neigh->n_ext_flags = nla_get_u32(tb[NDA_FLAGS_EXT]);
        neigh->ce_mask |= NEIGH_ATTR_EXT_FLAGS;
    }
#endif
```

**影响**:
- 启用后可获取/设置邻居表的扩展标志
- 包括 `NTF_EXT_MANAGED` 和 `NTF_EXT_LOCKED`
- 需要内核支持 `NDA_FLAGS_EXT`

#### 2. 网桥端口属性

**文件**: `lib/route/link/bridge.c`

```c
#ifdef OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION
    [IFLA_BRPORT_LOCKED]            = { .type = NLA_U8 },
    [IFLA_BRPORT_MAB]               = { .type = NLA_U8 },
    [IFLA_BRPORT_NEIGH_VLAN_SUPPRESS] = { .type = NLA_U8 },
#endif
```

**新增属性**:
| 属性 | 说明 | 内核版本要求 |
|------|------|-------------|
| `IFLA_BRPORT_LOCKED` | 锁定端口 | >= 5.10 |
| `IFLA_BRPORT_MAB` | MAC 认证绕过 | >= 5.10 |
| `IFLA_BRPORT_NEIGH_VLAN_SUPPRESS` | 抑制邻居 VLAN | >= 5.10 |

**使用示例**:
```c
#ifdef OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION
    rtnl_link_bridge_set_port_locked(link, 1);
#endif
```

## 行为差异

### 1. nl_socket_use_seq() 序列号生成

**差异**: 修复了序列号溢出处理

**上游版本**:
```c
unsigned int nl_socket_use_seq(struct nl_sock *sk) {
    return sk->s_seq_next++;
}
```

**OpenHarmony 版本**:
```c
unsigned int nl_socket_use_seq(struct nl_sock *sk) {
    if (sk->s_seq_next == UINT_MAX) {
        sk->s_seq_next = 0;
        return UINT_MAX;
    }
    return sk->s_seq_next++;
}
```

**影响**: 
- 行为更健壮，正确处理溢出情况
- 公共 API 签名不变，内部行为增强

### 2. 动态库加载

**差异**: 禁用了模块动态加载功能

**上游版本**:
```c
void *load_module(const char *prefix, const char *name) {
    char path[FILENAME_MAX+1];
    snprintf(path, sizeof(path), "%s/%s/%s.so",
             _NL_PKGLIBDIR, prefix, name);
    return dlopen(path, RTLD_LAZY);
}
```

**OpenHarmony 版本**:
```c
void *load_module(const char *prefix, const char *name) {
    // 动态库路径构造代码被注释掉
    // OpenHarmony 不支持此功能
    return NULL;
}
```

**影响**:
- `nl_cli_load_module()` 等函数不可用
- 所有功能静态链接到主库中

## 新增内部 API

### 1. _badrandom_from_time()

**文件**: `lib/socket.c`

```c
static uint32_t _badrandom_from_time(void)
{
    uint32_t result;
    uint64_t v64;
    time_t t;

    t = time(NULL);
    v64 = (uint64_t)t;
    result = (uint32_t)v64;
    result ^= (~(v64 >> 32));
    return result;
}
```

**说明**: 内部函数，用于改进端口随机化，不暴露为公共 API。

## 宏定义差异

### 1. ARRAY_SIZE 保护

**OpenHarmony 新增**:
```c
#ifndef ARRAY_SIZE
#define _NL_N_ELEMENTS(arr) (sizeof(arr) / sizeof((arr)[0]))
#define ARRAY_SIZE(arr) _NL_N_ELEMENTS(arr)
#endif
```

**原因**: 防止与系统中其他头文件的宏定义冲突。

### 2. NTF_EXT_* 定义

**OpenHarmony 新增**（当内核头文件缺失时）:
```c
#define NTF_EXT_MANAGED     (1 << 0)
#define NTF_EXT_LOCKED      (1 << 1)
```

**原因**: 补充旧版本内核头文件缺失的定义。

## 类型定义差异

### 1. restrict 关键字

**上游版本**:
```c
void *_nl_memcpy(void *restrict dest, const void *restrict src, size_t n)
```

**OpenHarmony 版本**:
```c
void *_nl_memcpy(void *__restrict dest, const void *__restrict src, size_t n)
```

**原因**: `__restrict` 兼容性更好，支持 C89/C99 混合编译环境。

## 函数签名不变但实现增强

以下函数签名与上游一致，但内部实现经过安全加固：

| 函数 | 增强内容 |
|------|---------|
| `nla_nest_end()` | 添加 NULL 参数检查 |
| `nl_object_diff()` | 修复有符号移位溢出 |
| `rtnl_tc_calc_cell_log()` | 修复有符号比较 |
| `nl_proto2str()` | 增强边界检查 |
| `xfrmnl_sa_parse()` | 添加所有内存分配检查 |
| `xfrmnl_sp_parse()` | 添加所有内存分配检查 |
| `xfrmnl_ae_parse()` | 添加所有内存分配检查 |

## 废弃或禁用的功能

### 1. CLI 模块加载

**状态**: 禁用

**影响函数**:
- `nl_cli_load_module()`
- `nl_cli_load_module_default()`

**替代方案**: 所有 CLI 功能静态编译到库中。

### 2. 配置文件加载

**状态**: 不可用

**说明**: 
- 配置路径被硬编码（`/etc/libnl` 等）
- 实际运行时这些路径不存在
- libnl 在 OH 中以编程方式配置

## 兼容性说明

### 向前兼容

- OpenHarmony 版本向上游版本兼容
- 使用标准 libnl API 的应用可在 OH 上运行

### 向后兼容

- 依赖 `OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION` 特性的代码
- 在非 OH 环境编译时需要定义该宏

## 迁移指南

### 从上游 libnl 迁移到 OH 版本

**步骤 1**: 检查动态库加载
```c
// 移除或注释掉
// nl_cli_load_module("route", "tc");
```

**步骤 2**: 验证内核特性依赖
```c
// 如果使用新内核特性，确保目标内核支持
#ifdef OPEN_HARMONY_UPDATE_ADAPT_KERNEL_VERSION
    // 使用新特性
#else
    // 回退方案
#endif
```

**步骤 3**: 测试序列号相关代码
```c
// 序列号溢出处理已修复，但需验证业务逻辑
uint32_t seq = nl_socket_use_seq(sock);
```

## 文档化建议

使用 libnl 时，建议：

1. **检查返回值**: 安全加固后更多函数可能返回错误
2. **避免动态模块**: 不要依赖 CLI 动态加载功能
3. **内核版本匹配**: 使用条件编译适配不同内核版本
