# Patch 详细分析

> OpenHarmony musl Patch 分析文档
> 本文档说明 OpenHarmony 对 musl 库的适配方式

---

## 1. Patch 策略概述

### 1.1 musl 的特殊适配方式

与大多数 OpenHarmony 第三方库采用 `.patch` 文件不同，musl 库采用了**条件编译 + Porting 目录**的复合适配策略：

```
上游 musl 源码
        │
        ├── 条件编译隔离 ──► #ifdef OHOS_xxx 宏
        │
        └── Porting 目录 ──► 平台特定适配代码
```

**统计数据**：
- OH 特有条件编译：**257 处匹配**，分布在 **60 个文件**中
- Porting 适配目录：**6 个平台**（Linux, LiteOS-A, LiteOS-M, UniProton 等）
- Hook 机制实现：**5 个核心模块**

### 1.2 适配方式对比

| 方式 | 传统 Patch | musl 采用的方式 |
|------|-----------|----------------|
| **文件形式** | `.patch` 差异文件 | 条件编译 + Porting 目录 |
| **代码可读性** | 差，需应用后查看 | 好，源码中直接体现 |
| **版本升级** | 需重新解决冲突 | 条件隔离，冲突概率低 |
| **维护成本** | 高，需管理多个 patch | 中等，代码即文档 |
| **平台适配** | 单一策略 | 支持多平台统一源码 |

---

## 2. 条件编译适配

### 2.1 OHOS 相关宏总览

musl 库中使用了多种 OHOS 特有的条件编译宏，按功能分类如下：

#### 2.1.1 设备参数与系统特性

| 宏名称 | 定义位置 | 用途 | 涉及文件数 |
|--------|----------|------|-----------|
| `OHOS_ENABLE_PARAMETER` | `src/hook/linux/musl_preinit.c` | 参数设备 API 使能，控制设备参数读写 | ~25 处 |
| `SANITIZER_OHOS` | `src/env/__libc_start_main.c` | OHOS Sanitizer 支持 | 少量 |

#### 2.1.2 Hook 机制相关

| 宏名称 | 定义位置 | 用途 | 涉及文件数 |
|--------|----------|------|-----------|
| `OHOS_FDTRACK_HOOK_ENABLE` | `src/hook/linux/musl_fdtrack_load.c` | 文件描述符追踪 Hook | ~30 处 |
| `OHOS_SOCKET_HOOK_ENABLE` | `src/hook/linux/musl_socket_preinit_common.c` | Socket 操作 Hook | ~15 处 |

#### 2.1.3 网络权限与代理

| 宏名称 | 定义位置 | 用途 | 涉及文件数 |
|--------|----------|------|-----------|
| `OHOS_PERMISSION_INTERNET` | `src/network/socket.c` | 互联网权限检查 | ~10 处 |
| `OHOS_DNS_PROXY_BY_NETSYS` | `src/network/getaddrinfo.c` | DNS 代理功能 | ~20 处 |
| `OHOS_FWMARK_CLIENT_BY_NETSYS` | `src/network/resolvconf.c` | Fwmark 客户端功能 | ~5 处 |

### 2.2 关键代码示例

#### 2.2.1 文件描述符追踪 Hook

**文件**: `src/unistd/close.c`

```c
#include "fdtrack.h"

#ifdef OHOS_FDTRACK_HOOK_ENABLE
static void OHOS_fdtrack_before_close(int fd) {
    // 在 close 操作前记录 FD 信息
    __fdtrack_remove_fd(fd);
}
#endif

int close(int fd) {
#ifdef OHOS_FDTRACK_HOOK_ENABLE
    OHOS_fdtrack_before_close(fd);
#endif
    // 标准 close 操作
    return syscall(SYS_close, fd);
}
```

**涉及 Hook 的文件**：
- `src/unistd/close.c` - 文件关闭追踪
- `src/fcntl/open.c` - 文件打开追踪
- `src/fcntl/openat.c` - openat 追踪
- `src/unistd/dup.c` - dup/dup2 追踪
- `src/unistd/pipe.c` - 管道操作追踪
- `src/stdio/fopen.c` - 文件打开追踪
- `src/network/socketpair.c` - socketpair 追踪
- `src/network/accept.c` - accept 追踪
- `src/linux/epoll.c` - epoll 操作追踪
- `src/linux/eventfd.c` - eventfd 操作追踪
- `src/misc/ioctl.c` - ioctl 操作追踪
- `src/thread/linux/pthread_create.c` - 线程创建时 FD 追踪

#### 2.2.2 Socket Hook

**文件**: `src/network/socket.c`

```c
#include "socket_common.h"

#ifdef OHOS_SOCKET_HOOK_ENABLE
#include "hilog_adapter.h"
#endif

#if OHOS_PERMISSION_INTERNET
#include "permission_check.h"
#endif

int socket(int domain, int type, int protocol) {
#if OHOS_PERMISSION_INTERNET
    // 检查互联网权限
    if (!check_internet_permission()) {
        return -EPERM;
    }
#endif

#ifdef OHOS_FDTRACK_HOOK_ENABLE
    // 追踪 socket 创建
    int fd = __socket(domain, type, protocol);
    if (fd >= 0) {
        __fdtrack_add_socket(fd, domain, type, protocol);
    }
    return fd;
#else
    return __socket(domain, type, protocol);
#endif
}
```

**涉及 Socket Hook 的文件**：
- `src/network/socket.c` - socket 创建
- `src/network/getaddrinfo.c` - DNS 查询（支持代理）
- `src/network/lookup_name.c` - 域名解析
- `src/network/resolvconf.c` - DNS 配置
- `src/network/res_msend.c` - DNS 响应处理.2.3 参数设备 API



#### 2**文件**: `src/hook/linux/musl_preinit.c`

```c
#ifdef OHOS_ENABLE_PARAMETER
#include "parameter_api.h"
#include "device_api_version.h"
#endif

void OHOS_preinit(void) {
#ifdef OHOS_ENABLE_PARAMETER
    // 初始化参数设备
    char hook_param_value[OHOS_PARAM_MAX_SIZE + 1] = {0};
    unsigned int len = OHOS_PARAM_MAX_SIZE;

    // 读取系统参数
    GetParameter("musl.hook.enable", "0", hook_param_value, len);
#endif
}

void InitDeviceApiVersion(void) {
#ifdef OHOS_ENABLE_PARAMETER
    // 获取设备 API 版本
    uint32_t api_version = GetDeviceApiVersion();
#endif
}
```

---

## 3. Porting 目录适配

### 3.1 Porting 目录结构

```
porting/
├── linux/                          # Linux 平台适配
│   └── user/                       # 用户态适配层
│       ├── src/                    # 适配源文件
│       │   ├── hook/              # Hook 机制
│       │   │   ├── musl_fdtrack_load.c
│       │   │   ├── musl_fdtrack.c
│       │   │   ├── musl_socket_preinit.c
│       │   │   ├── musl_socket_preinit_common.c
│       │   │   ├── musl_preinit.c
│       │   │   └── socket_common.c
│       │   ├── network/           # 网络适配
│       │   │   ├── socket.c
│       │   │   ├── getaddrinfo.c
│       │   │   ├── lookup_name.c
│       │   │   └── resolvconf.c
│       │   ├── hilog/            # 日志适配
│       │   │   └── hilog_adapter.c
│       │   ├── fdsan/            # 文件描述符安全
│       │   │   └── fdsan.c
│       │   ├── sigchain/          # 信号链
│       │   │   └── sigchain.c
│       │   ├── time/             # 时间
│       │   │   └── __tz.c
│       │   ├── trace/            # 跟踪
│       │   │   └── trace_marker.c
│       │   └── thread/           # 线程
│       │       └── pthread_sigmask.c
│       └── ldso/                 # 动态链接器适配
│           ├── dynlink.c
│           └── ld_log.c
├── liteos_a/                       # LiteOS-A 内核适配
│   ├── user/src/
│   │   └── linux/
│   │       └── cap.c              # 能力（Capability）管理
│   └── user/ldso/
│       └── dynlink.c
├── liteos_m/                       # LiteOS-M 内核适配
│   ├── user/
│   │   ├── hook/
│   │   ├── src/
│   │   └── ldso/
│   └── kernel/
├── liteos_m_iccarm/                # LiteOS-M IAR 编译器适配
│   └── kernel/
├── liteos_a_newlib/                # LiteOS-A Newlib 适配
│   └── kernel/
└── uniproton/                      # UniProton 内核适配
    ├── kernel/
    │   └── BUILD.gn
    └── kernel/
```

### 3.2 各平台适配重点

#### 3.2.1 Linux 平台适配

**适配文件**: `porting/linux/user/src/`

| 模块 | 文件 | 适配内容 |
|------|------|---------|
| Hook 机制 | `hook/musl_preinit.c` | 启动时初始化 Hook 环境 |
| | `hook/musl_fdtrack.c` | FD 追踪实现 |
| | `hook/musl_socket_preinit.c` | Socket Hook 初始化 |
| 网络 | `network/socket.c` | Socket 权限检查 |
| | `network/getaddrinfo.c` | DNS 代理配置 |
| | `network/resolvconf.c` | DNS 配置管理 |
| 日志 | `hilog/hilog_adapter.c` | OHOS 日志系统集成 |
| 信号 | `sigchain/sigchain.c` | 信号链处理 |

#### 3.2.2 LiteOS-A 平台适配

**适配文件**: `porting/liteos_a/user/src/`

| 模块 | 文件 | 适配内容 |
|------|------|---------|
| 能力管理 | `linux/cap.c` | Linux 兼容的能力接口 |
| 动态链接 | `user/ldso/dynlink.c` | OHOS-vdso.so 支持 |

**LiteOS-A 特有配置**：
```c
// vdsO 命名
vdso.shortname = "OHOS-vdso.so";
```

#### 3.2.3 能力（Capability）管理

**文件**: `porting/liteos_a/user/src/linux/cap.c`

```c
typedef enum {
    OHOS_CAP_CHOWN = 0,
    OHOS_CAP_DAC_EXECUTE,
    OHOS_CAP_DAC_WRITE,
    OHOS_CAP_DAC_READ_SEARCH,
    OHOS_CAP_FOWNER,
    OHOS_CAP_KILL,
    OHOS_CAP_SETGID,
    OHOS_CAP_SETUID,
    OHOS_CAP_NET_BIND_SERVICE,
    OHOS_CAP_NET_BROADCAST,
    OHOS_CAP_NET_ADMIN,
    OHOS_CAP_NET_RAW,
    OHOS_CAP_FS_MOUNT,
    OHOS_CAP_FS_FORMAT,
    OHOS_CAP_SCHED_SETPRIORITY,
    OHOS_CAP_SET_TIMEOFDAY,
    OHOS_CAP_CLOCK_SETTIME,
    OHOS_CAP_CAPSET,
    OHOS_CAP_REBOOT,
    OHOS_CAP_SHELL_EXEC,
} OHOS_CAP_TYPE;
```

---

## 4. 动态链接器 OH 适配

### 4.1 OHOS 特有段类型

动态链接器支持以下 OHOS 特有的 Program Header 类型：

| 段类型 | 宏定义 | 用途 |
|--------|--------|------|
| `PT_OHOS_CFI_MODIFIER` | `ph->p_type == PT_OHOS_CFI_MODIFIER` | 控制流完整性修改器 |
| `PT_OHOS_RANDOMDATA` | `ph->p_type == PT_OHOS_RANDOMDATA` | 随机化数据段 |

**代码位置**: `ldso/linux/dynlink.c`

```c
// CFI 修改器处理
} else if (ph->p_type == PT_OHOS_CFI_MODIFIER) {
    // 处理 CFI 相关数据
    process_cfi_modifier(ph);
}

// 随机化数据处理
if (ph->p_type == PT_OHOS_RANDOMDATA) {
    // 读取随机化数据
    load_random_data(ph);
}
```

### 4.2 vDSO 适配

**文件**: `ldso/dynlink.c`

```c
// OHOS vDSO 配置
vdso.shortname = "OHOS-vdso.so";
vdso.path = "/system/lib64/OHOS-vdso.so";
```

### 4.3 参数设备初始化

**文件**: `ldso/linux/dynlink.c`

```c
#ifdef OHOS_ENABLE_PARAMETER
// 初始化设备 API 版本
InitDeviceApiVersion();  // 当定义了 OHOS_ENABLE_PARAMETER 时调用

// 获取设备版本信息
uint32_t api_version = GetDeviceApiVersion();
#endif
```

---

## 5. OH 特有功能清单

### 5.1 功能分类总表

| 类别 | 功能名称 | 宏定义 | 状态 |
|------|---------|--------|------|
| **Hook 机制** | 文件描述符追踪 | `OHOS_FDTRACK_HOOK_ENABLE` | 已实现 |
| | Socket Hook | `OHOS_SOCKET_HOOK_ENABLE` | 已实现 |
| | 内存 Hook | - | 已实现 |
| **系统特性** | 参数设备 API | `OHOS_ENABLE_PARAMETER` | 已实现 |
| | 设备 API 版本 | `OHOS_ENABLE_PARAMETER` | 已实现 |
| **网络** | 互联网权限 | `OHOS_PERMISSION_INTERNET` | 已实现 |
| | DNS 代理 | `OHOS_DNS_PROXY_BY_NETSYS` | 已实现 |
| | Fwmark 客户端 | `OHOS_FWMARK_CLIENT_BY_NETSYS` | 已实现 |
| **安全** | Sanitizer | `SANITIZER_OHOS` | 已实现 |
| **日志** | Hilog 集成 | - | 已实现 |

### 5.2 Hook 机制详解

#### 5.2.1 文件描述符追踪 (FD Track)

**用途**: 追踪应用中所有文件描述符的操作，用于调试和性能分析

**实现文件**：
- `src/hook/linux/musl_fdtrack.c` - 核心追踪逻辑
- `src/hook/linux/musl_fdtrack_load.c` - 加载逻辑
- `src/unistd/close.c` - close 操作追踪
- `src/fcntl/open.c` - open 操作追踪

**追踪的事件**：
| 事件 | 函数 | 说明 |
|------|------|------|
| 创建 FD | `open()`, `socket()`, `pipe()` | 记录新 FD |
| 关闭 FD | `close()` | 移除 FD 记录 |
| 复制 FD | `dup()`, `dup2()`, `dup3()` | 复制记录 |
| 修改 FD | `fcntl()` | 更新 FD 属性 |

#### 5.2.2 Socket Hook

**用途**: 拦截所有 Socket 操作，支持权限检查和网络监控

**实现文件**：
- `src/network/socket.c` - socket() Hook
- `src/network/getaddrinfo.c` - DNS 查询 Hook
- `src/network/lookup_name.c` - 域名解析 Hook
- `src/network/resolvconf.c` - DNS 配置 Hook

**Socket Hook 流程**：
```
socket() 调用
     │
     ├── 权限检查 ──► OHOS_PERMISSION_INTERNET
     │
     ├── 创建追踪 ──► OHOS_FDTRACK_HOOK_ENABLE
     │
     └── DNS 代理 ──► OHOS_DNS_PROXY_BY_NETSYS
```

### 5.3 网络权限系统

#### 5.3.1 互联网权限检查

**宏**: `OHOS_PERMISSION_INTERNET`

**检查点**：
| 函数 | 检查位置 | 失败返回值 |
|------|---------|-----------|
| `socket()` | 调用入口 | `-EPERM` |
| `connect()` | 调用入口 | `-EPERM` |
| `bind()` | 调用入口 | `-EPERM` |

**权限检查流程**：
```c
#if OHOS_PERMISSION_INTERNET
int check_internet_permission(void) {
    // 获取调用进程权限
    uint64_t caps = GetProcessCaps();

    // 检查是否具有网络权限
    return (caps & (1 << OHOS_CAP_NET_RAW)) ||
           (caps & (1 << OHOS_CAP_NET_BIND_SERVICE));
}
#endif
```

#### 5.3.2 DNS 代理

**宏**: `OHOS_DNS_PROXY_BY_NETSYS`

**功能**：
- 将 DNS 查询重定向到系统配置的 DNS 代理
- 支持 Netsys 服务进行 DNS 查询

**实现文件**：
- `src/network/getaddrinfo.c`
- `src/network/res_msend.c`

---

## 6. 安全特性

### 6.1 ASLR (地址空间布局随机化)

**实现位置**: `ldso/linux/dynlink.c`

**功能**：
- 动态链接器加载地址随机化
- 共享库映射地址随机化

### 6.2 RELRO 共享机制

**配置**：
- 配置文件: `config/ld-musl-namespace-{arch}.ini`
- 支持完全 RELRO (Full RELRO)

### 6.3 Namespace 隔离

**配置文件**：
| 架构 | 配置文件 |
|------|---------|
| arm | `ld-musl-namespace-arm.ini` |
| aarch64 | `ld-musl-namespace-aarch64.ini` |
| x86_64 | `ld-musl-namespace-x86_64.ini` |

**功能**：
- 默认 Namespace (default)
- NDK Namespace (ndk)
- 库搜索路径隔离

### 6.4 CFI (控制流完整性)

**段类型**: `PT_OHOS_CFI_MODIFIER`

**功能**：
- 控制流完整性检查
- 防止控制流劫持攻击

---

## 7. 升级注意事项

### 7.1 升级上游 musl 时的注意事项

由于 musl 采用条件编译方式进行 OH 适配，升级上游版本时需要关注：

#### 7.1.1 必须保留的 OH 适配

| 类型 | 原因 | 检查方式 |
|------|------|---------|
| `OHOS_xxx` 宏 | OH 特有功能 | 搜索 `#ifdef OHOS` |
| `porting/` 目录 | 平台适配 | 对比目录结构 |
| `src/hook/` | Hook 机制 | 检查 Hook 实现 |
| `ldso/` 修改 | 动态链接增强 | 对比 diff |

#### 7.1.2 可合并到上游的修改

| 修改类型 | 说明 | 建议 |
|---------|------|------|
| 通用安全增强 | ASLR、RELRO 等 | 评估后尝试推向上游 |
| 通用 Bugfix | 与平台无关的 Bug | 直接推向上游 |
| OH 特有功能 | Hook、权限检查等 | 保留在 OH 分支 |

### 7.2 Patch 维护建议

#### 7.2.1 当前方案评估

**优点**：
- 代码可维护性好
- 版本升级冲突概率低
- 平台隔离清晰

**缺点**：
- 代码复杂度增加
- 需要维护多套条件编译
- 测试复杂度增加

#### 7.2.2 改进建议

1. **标准化 OHOS 宏命名**: 统一使用 `__OHOS__` 前缀
2. **文档化所有适配**: 为每个条件编译块添加详细注释
3. **自动化测试**: Hook 机制需要完整测试覆盖
4. **模块化适配**: 将 OH 适配移到独立的 `ohos/` 目录

---

## 8. 相关文件索引

### 8.1 条件编译涉及文件

| 分类 | 文件数 | 主要用途 |
|------|-------|---------|
| Hook 机制 | ~10 | FD 追踪、Socket Hook |
| 网络相关 | ~8 | DNS、权限检查 |
| 平台适配 | ~15 | 各平台差异 |
| 动态链接 | ~5 | ASLR、CFI |
| 其他 | ~22 | 日志、信号等 |

### 8.2 Porting 适配文件

| 平台 | 适配文件数 | 主要适配点 |
|------|----------|-----------|
| Linux | ~25 | Hook、网络、权限 |
| LiteOS-A | ~10 | 能力管理、vDSO |
| LiteOS-M | ~15 | 内核差异 |
| UniProton | ~10 | 内核差异 |

---

## 附录 A: 条件编译宏速查表

| 宏名称 | 用途 | 定义文件 |
|--------|------|---------|
| `OHOS_ENABLE_PARAMETER` | 参数设备 API | `musl_preinit.c` |
| `OHOS_FDTRACK_HOOK_ENABLE` | FD 追踪 | `musl_fdtrack_load.c` |
| `OHOS_SOCKET_HOOK_ENABLE` | Socket Hook | `musl_socket_preinit_common.c` |
| `OHOS_PERMISSION_INTERNET` | 网络权限 | `socket.c` |
| `OHOS_DNS_PROXY_BY_NETSYS` | DNS 代理 | `getaddrinfo.c` |
| `OHOS_FWMARK_CLIENT_BY_NETSYS` | Fwmark | `resolvconf.c` |
| `SANITIZER_OHOS` | Sanitizer | `__libc_start_main.c` |

---

## 附录 B: Hook 机制文件清单

```
src/hook/linux/
├── musl_preinit.c              # 预初始化
├── musl_fdtrack.c             # FD 追踪核心
├── musl_fdtrack_load.c        # FD 追踪加载
├── musl_socket_preinit.c      # Socket 预初始化
├── musl_socket_preinit_common.c # Socket 公共预初始化
└── socket_common.c            # Socket 公共接口
```

---

*文档版本: 1.0*
*最后更新: 2025-02-08*
