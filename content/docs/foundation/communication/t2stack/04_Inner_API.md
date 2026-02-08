# T2Stack 内部 API 与模块接口

> 本文档记录 t2stack 模块间的内部接口（Inner API），供模块开发者参考。

## 目录

- [1. 模块间依赖概览](#1-模块间依赖概览)
- [2. NStackX Util 接口](#2-nstackx-util-接口)
- [3. NStackX Congestion 接口](#3-nstackx-congestion-接口)
- [4. 稳定性标注](#4-稳定性标注)

---

## 1. 模块间依赖概览

### 依赖矩阵

| 调用方 → | Fillp | DFile | NStackX Ctrl |
|----------|-------|-------|--------------|
| **NStackX Util** | ✅ 被调用 | ✅ 被调用 | ✅ 被调用 |
| **NStackX Congestion** | ✅ 调用 | ✅ 调用 | ❌ |

### 接口风格

所有内部接口遵循以下规范：

| 规范 | 说明 |
|------|------|
| **导出宏** | `NSTACKX_EXPORT` / `DFINDER_EXPORT` |
| **C++ 兼容** | 使用 `extern "C"` 包裹 |
| **错误处理** | 返回 int32_t，负值表示错误 |
| **头文件保护** | `#ifndef XXX_H` / `#define XXX_H` |

---

## 2. NStackX Util 接口

### 2.1 错误码定义

**头文件**：`nstackx_util/interface/nstackx_error.h`

```c
/* 错误码定义示例 */
#define NSTACKX_EOK            0   /* 成功 */
#define NSTACKX_EFAILED       -1   /* 通用失败 */
#define NSTACKX_EPARAM        -2   /* 参数错误 */
#define NSTACKX_EMEMORY       -3   /* 内存错误 */
#define NSTACKX_ETIMEOUT      -4   /* 超时 */
/* ... 更多错误码 */
```

### 2.2 事件机制

**头文件**：`nstackx_util/interface/nstackx_event.h`（推测）

```c
/**
 * @brief 创建事件
 *
 * @return 事件句柄
 */
NSTACKX_Handle NSTACKX_EventCreate(void);

/**
 * @brief 等待事件
 *
 * @param [in] handle 事件句柄
 * @param [in] timeout 超时时间（毫秒）
 *
 * @return int32_t
 *         0: 事件触发
 *         负值: 失败或超时
 */
int32_t NSTACKX_EventWait(NSTACKX_Handle handle, uint32_t timeout);

/**
 * @brief 触发事件
 *
 * @param [in] handle 事件句柄
 */
int32_t NSTACKX_EventSet(NSTACKX_Handle handle);

/**
 * @brief 销毁事件
 */
void NSTACKX_EventDestroy(NSTACKX_Handle handle);
```

### 2.3 Socket 封装

**头文件**：`nstackx_util/interface/nstackx_socket.h`（推测）

```c
/**
 * @brief 创建 UDP socket
 *
 * @return socket 描述符
 */
int32_t NSTACKX_SocketUdpCreate(void);

/**
 * @brief 发送数据
 *
 * @param [in] fd       socket 描述符
 * @param [in] data    数据指针
 * @param [in] len     数据长度
 * @param [in] to      目标地址
 * @param [in] toLen   目标地址长度
 *
 * @return int32_t
 *         非负值: 发送的字节数
 *         负值: 失败
 */
int32_t NSTACKX_SocketSendTo(int32_t fd, const uint8_t *data, int32_t len,
                              struct sockaddr *to, socklen_t toLen);

/**
 * @brief 接收数据
 *
 * @param [in] fd       socket 描述符
 * @param [out] data   接收缓冲区
 * @param [in] len     缓冲区长度
 * @param [out] from   源地址（可选）
 * @param [out] fromLen 源地址长度（可选）
 *
 * @return int32_t
 *         非负值: 接收的字节数
 *         负值: 失败
 */
int32_t NSTACKX_SocketRecvFrom(int32_t fd, uint8_t *data, int32_t len,
                                struct sockaddr *from, socklen_t *fromLen);
```

### 2.4 定时器

**头文件**：`nstackx_util/interface/nstackx_timer.h`（推测）

```c
/**
 * @brief 定时器回调类型
 */
typedef void (*NSTACKX_TimerCallback)(void *arg);

/**
 * @brief 创建定时器
 *
 * @param [in] callback 回调函数
 * @param [in] arg     回调参数
 *
 * @return 定时器句柄
 */
NSTACKX_Handle NSTACKX_TimerCreate(NSTACKX_TimerCallback callback, void *arg);

/**
 * @brief 启动定时器
 *
 * @param [in] handle 定时器句柄
 * @param [in] ms     定时时间（毫秒）
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 */
int32_t NSTACKX_TimerStart(NSTACKX_Handle handle, uint32_t ms);

/**
 * @brief 停止定时器
 */
int32_t NSTACKX_TimerStop(NSTACKX_Handle handle);

/**
 * @brief 销毁定时器
 */
void NSTACKX_TimerDestroy(NSTACKX_Handle handle);
```

### 2.5 加密接口

**头文件**：`nstackx_util/interface/nstackx_crypto.h`（推测）

```c
/**
 * @brief AES-ECB 加密
 *
 * @param [in] key        密钥（16/24/32 字节）
 * @param [in] keyLen     密钥长度
 * @param [in] in         明文输入
 * @param [in] inLen      输入长度（必须是 16 的倍数）
 * @param [out] out       密文输出
 * @param [in,out] outLen 输出缓冲区长度/实际输出长度
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 */
int32_t NSTACKX_AesEncrypt(const uint8_t *key, uint32_t keyLen,
                           const uint8_t *in, uint32_t inLen,
                           uint8_t *out, uint32_t *outLen);

/**
 * @brief AES-ECB 解密
 */
int32_t NSTACKX_AesDecrypt(const uint8_t *key, uint32_t keyLen,
                           const uint8_t *in, uint32_t inLen,
                           uint8_t *out, uint32_t *outLen);

/**
 * @brief HMAC-SHA256
 */
int32_t NSTACKX_HmacSha256(const uint8_t *key, uint32_t keyLen,
                           const uint8_t *data, uint32_t dataLen,
                           uint8_t *hash, uint32_t *hashLen);
```

---

## 3. NStackX Congestion 接口

### 3.1 拥塞控制算法

**头文件**：`nstackx_congestion/interface/nstackx_congestion.h`（推测）

```c
/**
 * @brief 拥塞控制参数
 */
typedef struct {
    uint32_t cwnd;          /* 拥塞窗口 */
    uint32_t ssthresh;      /* 慢启动阈值 */
    uint32_t rtt;           /* 往返时间 */
    uint32_t packetLoss;    /* 丢包率 */
} NSTACKX_CongestionParam;

/**
 * @brief 创建拥塞控制器
 *
 * @return 控制器句柄
 */
NSTACKX_CongHandle NSTACKX_CongestionCreate(void);

/**
 * @brief 更新拥塞状态
 *
 * @param [in] handle 控制器句柄
 * @param [in] param  新的拥塞参数
 *
 * @return int32_t
 *         0: 成功
 *         负值: 失败
 */
int32_t NSTACKX_CongestionUpdate(NSTACKX_CongHandle handle, NSTACKX_CongestionParam *param);

/**
 * @brief 获取发送窗口
 *
 * @param [in] handle 控制器句柄
 *
 * @return 发送窗口大小
 */
uint32_t NSTACKX_CongestionGetSendWindow(NSTACKX_CongHandle handle);

/**
 * @brief 销毁拥塞控制器
 */
void NSTACKX_CongestionDestroy(NSTACKX_CongHandle handle);
```

---

## 4. 稳定性标注

### 稳定性等级

| 等级 | 标注 | 说明 |
|------|------|------|
| **Stable** | ✅ | 接口稳定，兼容性好 |
| **Unstable** | ⚠️ | 接口可能变化，不建议直接使用 |
| **Internal** | 🔒 | 内部接口，请勿直接调用 |

### 接口稳定性列表

| 接口模块 | 接口数量 | 稳定 | 不稳定 | 内部 |
|----------|----------|------|--------|------|
| **Fillp API** | ~50 | 40 | 10 | 0 |
| **DFile API** | ~30 | 25 | 5 | 0 |
| **NStackX API** | ~40 | 30 | 8 | 2 |
| **Util 内部** | ~20 | 5 | 5 | 10 |
| **Congestion** | ~10 | 3 | 2 | 5 |

### 稳定性判断依据

1. **Stable 接口标准**：
   - 有完整的注释文档
   - 在多个版本中保持兼容
   - 有对应的测试用例

2. **Unstable 接口特征**：
   - 注释中标注 `TODO` 或 `XXX`
   - 参数或返回值可能变化
   - 主要用于内部测试或临时功能

3. **内部接口特征**：
   - 位于 `*_internal.h` 或 `*_private.h`
   - 以 `_NSTACKX_` 或 `__` 开头命名
   - 无官方文档

---

## 相关文档

- [API 参考](./03_CAPI_Reference.md) - 对外 C API
- [模块结构](./02_Module_Structure.md) - 模块职责
- [架构说明](./01_Architecture.md)

---

*文档版本：1.0 - 组件交互.0*
*最后更新：2026-02-06*
