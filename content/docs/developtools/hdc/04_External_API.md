# 对外 API

## 目的

描述 hdc 项目对外的 API 接口，包括 JDWP 注册库、参数校验、错误码等。

## 适用范围

本文档适用于：
- 了解 hdc 对外提供的接口
- 集成 hdc 到其他应用
- JDWP 调试协议使用

## 相关跳转

- [项目概览](./00_Overview.md) - 项目定位和 JDWP 能力
- [架构说明](./03_Architecture.md) - Client-Server-Daemon 通信
- [内部 API](./05_Internal_API.md) - 模块间接口

---

## N-API 绑定状态

### 重要说明

**hdc 项目不提供 N-API 绑定**。

**证据**：
- `BUILD.gn:542-567` - hdc_register 是 `ohos_shared_library`
- `src/register/` - 纯 C++ 实现，无 `napi_module_register` 调用
- grep 搜索结果：整个项目中无 N-API 宏

**结论**：任何 JavaScript API 应在其他框架层（如 ArkUI、Ability 框架）实现，而非在 hdc 中。

---

## JDWP 注册库（libhdc_register.so）

### 概述

`hdc_register` 是一个**纯 C++ 原生库**，提供 JDWP（Java Debug Wire Protocol）连接能力，用于 ArkTS 应用的调试。

**构建目标**：`BUILD.gn:542-567`

```gn
ohos_shared_library("hdc_register") {
  defines = [
    "JS_JDWP_CONNECT",    # 启用 JDWP 连接
    "HDC_HILOG",
  ]
  sources = [
    "src/register/hdc_connect.cpp",
    "src/register/hdc_jdwp.cpp",
  ]
  external_deps = [
    "bounds_checking_function:libsec_shared",
    "c_utils:utils",
    "hilog:libhilog",
    "init:libbegetutil",
    "libuv:uv",
  ]
}
```

### 导出接口

**头文件**：`src/register/hdc_connect.h:22-33`

```cpp
#ifdef __cplusplus
extern "C" {
#endif
/**
 * 启动 JDWP 连接
 * @param processName 进程名称，格式 "ark:pid@com.xxx.xxxx"
 * @param pkgName 包名称，"com.xxx.xxxx"
 * @param isDebug 是否调试版本（1=debug, 0=release）
 * @param cb 回调函数，接收 JDWP 文件描述符和连接字符串
 */
void StartConnect(const std::string& processName, const std::string& pkgName, bool isDebug, Callback cb);

/**
 * 停止 JDWP 连接
 */
void StopConnect();

#ifdef __cplusplus
}
#endif
```

### 回调类型定义

**头文件**：`src/register/define_register.h:48`

```cpp
/**
 * JDWP 回调函数类型
 * @param fd JDWP Unix Domain Socket 文件描述符
 * @param str 连接字符串，格式 "ark:pid@com.xxx.xxxx, ark:pid@tid@Debugger" 或 "ark:pid@Debugger"
 */
using Callback = std::function<void(int fd, std::string str)>;
```

---

## JDWP 协议消息结构

### JsMsgHeader 结构

**头文件**：`src/register/hdc_jdwp.h:34-38`

```cpp
#pragma pack(push)
#pragma pack(1)
struct JsMsgHeader {
    uint32_t msgLen;    // 消息长度
    uint32_t pid;       // 进程 ID
    uint8_t isDebug;     // 1=debugApp, 0=releaseApp
};
#pragma pack(pop)
```

### JDWP Unix Domain Socket

**路径**：`src/daemon/jdwp.h`

- **设备端 Socket**：`/data/hdc/hdc_debug/hdc_server`
- **通信方式**：Unix Domain Socket (AF_UNIX, SOCK_STREAM)

**连接流程**：

```
ArkTS App
    │
    ▼
[Unix Socket] /data/hdc/hdc_debug/hdc_server
    │
    ▼
HDC Daemon
    │
    ▼
JDWP Simulator
    │
    ▼
[JVM TCP Socket] ohjpipid-control
```

---

## 参数校验

### 开发者模式检查

**实现**：`src/register/hdc_connect.cpp:141-160`

```cpp
void StartConnect(const std::string& processName, const std::string& pkgName, bool isDebug, Callback cb)
{
    if (!IsDeveloperMode()) {
        HILOG_INFO(LOG_CORE, "non developer mode not to start connect");
        return;  // 非开发者模式，拒绝启动
    }
    // ... 创建 pthread 启动 JDWP 连接
}
```

**检查方式**：读取系统参数 `const.developer_mode` 或类似参数

### 进程名称格式

**预期格式**：
- `"ark:pid@com.xxx.xxxx"` - 指定进程
- `"ark:pid@tid@Debugger"` - 指定线程
- `"ark:pid@Debugger"` - 调试器连接

**实现**：`src/register/hdc_jdwp.cpp` 解析字符串格式

---

## 同步/异步模式

### 模式分析

**JDWP 连接是同步的**（阻塞启动）：

**证据**：`src/register/hdc_connect.cpp:147-151`

```cpp
pthread_t tid;
// ... 初始化 ConnectManagement
if (pthread_create(&tid, nullptr, &HdcConnectRun, static_cast<void*>(g_connectManagement.get())) != 0) {
    HILOG_FATAL(LOG_CORE, "pthread_create fail!");
    return;  // 同步创建线程失败直接返回
}
```

### 回调通知

**模式**：事件驱动回调

- **触发时机**：JDWP Socket 连接建立/断开
- **回调参数**：
  - `fd` - JDWP Unix Domain Socket 文件描述符
  - `str` - 连接字符串（格式如 "ark:pid@com.xxx.xxxx, ark:pid@tid@Debugger"）

---

## 错误码与异常封装

### 系统日志级别

**宏定义**：`src/common/log.h`

| 级别 | 宏 | 用途 |
|--------|------|------|
| FATAL | `WRITE_LOG` 或 `HILOG_FATAL` | 致命错误，终止程序 |
| ERROR | `WRITE_ERR` 或 `HILOG_ERROR` | 错误 |
| WARN | `WRITE_WARN` 或 `HILOG_WARN` | 警告 |
| INFO | `WRITE_LOG` 或 `HILOG_INFO` | 信息 |
| DEBUG | `WRITE_LOG` 或 `HILOG_DEBUG` | 调试信息 |

### 安全函数使用

**证据**：`src/register/hdc_connect.cpp:66-79`

```cpp
static void GetDevItem(const char *key, std::string &out, const char *preDefine = nullptr)
{
    constexpr int len = 512;
    char buf[len] = "";
    if (memset_s(buf, len, 0, len) != EOK) {  // 安全的内存清零
        HILOG_WARN(LOG_CORE, "memset_s failed");
        return;
    }
    // ... 读取参数
}
```

**说明**：使用 OpenHarmony 的 `securec` 安全函数（`memset_s`, `memcpy_s` 等）

---

## 使用示例

### C++ 应用使用

```cpp
#include "hdc_connect.h"

// 回调函数
void MyJdwpCallback(int fd, std::string str) {
    printf("JDWP connected: fd=%d, str=%s\n", fd, str.c_str());
    // fd 可用于与 JDWP 协议通信
}

// 启动 JDWP 连接
void StartMyAppDebug() {
    std::string processName = "ark:pid@com.example.app";
    std::string pkgName = "com.example.app";
    bool isDebug = true;

    StartConnect(processName, pkgName, isDebug, MyJdwpCallback);
}

// 停止连接
void StopMyAppDebug() {
    StopConnect();
}
```

---

## 权限/前置条件

### 开发者模式

**要求**：系统必须处于开发者模式
- 参数：`const.developer_mode` = "1"
- 检查位置：`src/register/hdc_connect.cpp:IsDeveloperMode()`

### 权限要求

**Daemon 端权限**：
- `/data/hdc/hdc_debug/` 目录访问
- Unix Domain Socket 创建权限
- JDWP Socket 访问权限

---

## 关键结论

1. **无 N-API 绑定**：hdc_register 是纯 C++ 库，不使用 N-API 框架
2. **JDWP 协议集成**：提供 ArkTS 应用的 Java 调试能力
3. **Unix Domain Socket 通信**：通过 `/data/hdc/hdc_debug/hdc_server` 与 App 通信
4. **C 接口导出**：使用 `extern "C"` 导出 C 函数，方便其他语言绑定
5. **开发者模式检查**：非开发者模式拒绝启动 JDWP 连接
6. **同步启动**：使用 pthread 创建连接线程
7. **安全编码**：使用 securec 安全函数

---

## 待确认事项

**TODO(需确认)**：
1. JDWP 协议的完整消息格式定义
2. 与标准 JDWP 协议的兼容性
3. 多个 JDWP 连接的管理机制
4. JDWP 连接超时和重试策略
