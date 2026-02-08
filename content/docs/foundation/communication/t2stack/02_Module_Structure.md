# T2Stack 模块结构

## 目录结构概览

```
t2stack/
├── fillp/                      # 流传输协议模块
│   ├── include/               # 对外接口头文件
│   ├── src/                   # 核心实现代码
│   │   ├── app_lib/          # 应用层接口
│   │   ├── fillp_lib/        # Fillp 协议实现
│   │   │   └── fillp/        # Fillp 核心协议
│   │   └── public/           # 公共基础组件
│   └── BUILD.gn              # 构建配置
├── nstackx_congestion/        # 拥塞控制模块
│   ├── interface/            # 对外接口
│   ├── core/                 # 核心算法
│   ├── platform/             # 平台适配
│   └── BUILD.gn
├── nstackx_core/             # 核心模块
│   ├── dfile/               # DFile 文件传输
│   │   ├── interface/       # 对外接口头文件
│   │   ├── core/            # 核心协议实现
│   │   └── platform/        # 平台适配
│   ├── platform/            # 公共平台适配
│   └── BUILD.gn
├── nstackx_ctrl/            # 设备发现模块
│   ├── interface/           # 对外接口头文件
│   ├── core/                # 核心协议实现
│   │   ├── coap_discover/   # CoAP 发现协议
│   │   └── mini_discover/   # 轻量发现 (LiteOS-M)
│   └── BUILD.gn
├── nstackx_util/             # 公共基础模块
│   ├── interface/           # 对外接口头文件
│   ├── core/                # 核心功能实现
│   ├── platform/            # 平台适配
│   │   ├── liteos/          # LiteOS 适配
│   │   └── unix/            # Linux/Unix 适配
│   └── BUILD.gn
├── figures/                  # 架构图等资源
├── BUILD.gn                  # 根构建配置
├── bundle.json              # Bundle 配置
└── t2stack.gni              # 全局配置
```

## 模块职责

### 1. Fillp 模块 (流传输)

**职责**：提供可靠的流传输能力，主要用于音视频数据传输。

**目录**：`fillp/`

**核心文件**：

| 文件 | 职责 | 代码证据 |
|------|------|----------|
| `include/fillpinc.h` | 对外 API 接口定义 | `fillpinc.h:1-22` (头部保护) |
| `src/app_lib/src/api.c` | Socket API 实现 | `BUILD.gn:53-60` |
| `src/fillp_lib/src/fillp/fillp.c` | Fillp 主逻辑 | `BUILD.gn:61-62` |
| `src/fillp_lib/src/fillp/fillp_conn.c` | 连接管理 | `BUILD.gn:63` |
| `src/fillp_lib/src/fillp/fillp_flow_control.c` | 流量控制 | `BUILD.gn:64` |
| `src/fillp_lib/src/fillp/fillp_output.c` | 数据发送 | `BUILD.gn:69` |
| `src/fillp_lib/src/fillp/fillp_input.c` | 数据接收 | `BUILD.gn:67` |

**对外依赖**：
- `nstackx_util` - 公共模块
- `bounds_checking_function` - 安全函数

### 2. NStackX Congestion 模块 (拥塞控制)

**职责**：提供拥塞控制算法，供 Fillp 和 DFile 使用。

**目录**：`nstackx_congestion/`

**核心文件**：

| 文件 | 职责 |
|------|------|
| `interface/` | 拥塞算法接口定义 |
| `core/` | 拥塞控制算法实现 |
| `platform/` | 平台适配 |

**对外依赖**：无（基础算法模块）

### 3. NStackX Core / DFile 模块 (文件传输)

**职责**：提供设备间文件传输能力，支持大文件、分片、加密、多路径。

**目录**：`nstackx_core/dfile/`

**核心文件**：

| 文件 | 职责 | 代码证据 |
|------|------|----------|
| `interface/nstackx_dfile.h` | 对外 API 接口定义 | `nstackx_dfile.h:16-17` |
| `core/nstackx_dfile.c` | DFile 主逻辑 | `BUILD.gn:64` |
| `core/nstackx_dfile_session.c` | 会话管理 | `BUILD.gn:73` |
| `core/nstackx_dfile_send.c` | 发送逻辑 | `BUILD.gn:72` |
| `core/nstackx_dfile_transfer.c` | 传输控制 | `BUILD.gn:74` |
| `core/nstackx_dfile_retransmission.c` | 重传机制 | `BUILD.gn:71` |
| `core/nstackx_dfile_mp.c` | 多路径支持 | `BUILD.gn:70` |
| `core/nstackx_file_manager.c` | 文件管理 | `BUILD.gn:76` |

**对外依赖**：
- `nstackx_congestion` - 拥塞控制
- `nstackx_util` - 公共模块
- `mbedtls` / `openssl` - 加密库
- `bounds_checking_function` - 安全函数

### 4. NStackX Ctrl 模块 (设备发现)

**职责**：提供局域网设备发现能力，支持广播、单播、设备列表管理。

**目录**：`nstackx_ctrl/`

**核心文件**：

| 文件 | 职责 | 代码证据 |
|------|------|----------|
| `interface/nstackx.h` | 对外 API 接口定义 | `nstackx.h:16-17` |
| `core/nstackx_device.c` | 设备管理 | `BUILD.gn:40` |
| `core/nstackx_device_local.c` | 本地设备 | `BUILD.gn:41` |
| `core/nstackx_device_remote.c` | 远程设备 | `BUILD.gn:42` |
| `core/coap_discover/coap_discover.c` | CoAP 发现协议 | `BUILD.gn:81` |
| `core/coap_discover/coap_client.c` | CoAP 客户端 | `BUILD.gn:80` |

**对外依赖**：
- `nstackx_util` - 公共模块
- `libcoap` - CoAP 协议栈
- `cJSON` - JSON 解析
- `bounds_checking_function` - 安全函数

### 5. NStackX Util 模块 (公共模块)

**职责**：提供公共基础功能，包括事件、Socket、定时器、加密、日志等。

**目录**：`nstackx_util/`

**核心文件**：

| 文件 | 职责 | 代码证据 |
|------|------|----------|
| `interface/nstackx_error.h` | 错误码定义 | `bundle.json:88-91` |
| `core/nstackx_event.c` | 事件机制 | `BUILD.gn:59` |
| `core/nstackx_socket.c` | Socket 封装 | `BUILD.gn:61` |
| `core/nstackx_timer.c` | 定时器管理 | `BUILD.gn:62` |
| `core/nstackx_mbedtls.c` | mbedtls 封装 | `BUILD.gn:104` (LiteOS) |
| `core/nstackx_openssl.c` | OpenSSL 封装 | `BUILD.gn:226` (Standard) |
| `core/nstackx_log.c` | 日志模块 | `BUILD.gn:60` |

**对外依赖**：
- `hilog` - 日志系统
- `bounds_checking_function` - 安全函数
- `mbedtls` / `openssl` - 加密库 (通过 Fillp/DFile 传递)

## 依赖关系图

```
                              ┌─────────────────┐
                              │   上层业务调用   │
                              └────────┬────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              ↓                        ↓                        ↓
     ┌───────────────┐       ┌───────────────┐       ┌───────────────────┐
     │    Fillp      │       │   DFile       │       │   NStackX Ctrl    │
     │  (流传输)      │       │  (文件传输)    │       │   (设备发现)       │
     └───────┬───────┘       └───────┬───────┘       └─────────┬─────────┘
             │                       │                          │
             │                       │                          │
             └───────────┬───────────┘                          │
                         ↓                                      │
              ┌─────────────────────┐                           │
              │   NStackX Util      │ ←────────────────────────┘
              │    (公共模块)        │      依赖
              └──────────┬──────────┘
                         │
                         ↓
              ┌─────────────────────┐
              │   系统适配层         │
              │ LiteOS / Linux      │
              └─────────────────────┘
```

## 模块间接口

| 调用方 | 被调用方 | 接口类型 | 主要接口 |
|--------|----------|----------|----------|
| Fillp | NStackX Util | 同步调用 | Socket、定时器、事件 |
| Fillp | NStackX Congestion | 同步调用 | 拥塞算法查询 |
| DFile | NStackX Util | 同步调用 | Socket、加密、日志 |
| DFile | NStackX Congestion | 同步调用 | 拥塞窗口查询 |
| DFile | NStackX Ctrl | 同步调用 | 设备发现结果 |
| NStackX Ctrl | NStackX Util | 同步调用 | Socket、CoAP 封装 |

## 平台适配层

### 适配策略

T2Stack 采用**条件编译**的方式支持多平台：

```c
// 代码中的平台判断示例
#ifdef NSTACKX_WITH_LITEOS
    // LiteOS 特有代码
#elif defined(NSTACKX_WITH_LINUX)
    // Linux 特有代码
#endif
```

### 各模块平台适配

| 模块 | LiteOS-A | LiteOS-M | Linux | Standard |
|------|----------|----------|-------|----------|
| **fillp** | ✅ | ❌ | ✅ | ✅ |
| **nstackx_congestion** | ✅ | ✅ | ✅ | ✅ |
| **nstackx_dfile** | ✅ | ❌ | ✅ | ✅ |
| **nstackx_ctrl** | ✅ | ✅ | ✅ | ✅ |
| **nstackx_util** | ✅ | ✅ | ✅ | ✅ |

## 相关文档

- [项目概览](./00_Overview.md) - 核心能力介绍
- [架构说明](./01_Architecture.md) - 组件交互和线程模型
- [API 参考](./03_CAPI_Reference.md) - 完整 API 接口
- [构建系统](./05_Build_System.md) - GN 构建配置

---

*文档版本：1.0.0*
*最后更新：2026-02-06*
