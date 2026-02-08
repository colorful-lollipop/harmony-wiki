# 05 - API/接口差异

## 概述

libwebsockets 在 OpenHarmony 中**未添加新的公共 API**，也未修改现有 API 的签名。本库的使用方式与上游完全一致。

本节主要说明：
1. OH 使用的 API 子集
2. 行为一致性说明
3. 与上层封装的关系

---

## API 使用子集

### 使用的 API 类别

libwebsockets 提供了丰富的 API，OH 主要使用以下类别：

| API 类别 | 使用程度 | 说明 |
|---------|---------|-----|
| **Context 管理** | ✅ 完整使用 | `lws_create_context()`, `lws_context_destroy()` |
| **客户端连接** | ✅ 完整使用 | `lws_client_connect_via_info()` |
| **事件循环** | ✅ 完整使用 | `lws_service()` |
| **数据读写** | ✅ 完整使用 | `lws_write()`, `lws_read()` |
| **回调处理** | ✅ 完整使用 | `lws_callback_function` |
| **TLS/SSL** | ✅ 完整使用 | OpenSSL 集成 API |
| **服务器功能** | ⚠️ 有限使用 | 主要用于内部测试 |
| **MQTT 协议** | ❌ 未使用 | 未启用 |
| **Secure Streams** | ❌ 未使用 | 未启用 |
| **插件系统** | ❌ 未使用 | 未启用 |

### 主要使用的头文件

```c
// 核心头文件
#include <libwebsockets.h>

// 常用结构体和函数
struct lws_context;
struct lws;
struct lws_client_connect_info;

// 核心函数
struct lws_context *lws_create_context(const struct lws_context_creation_info *info);
void lws_context_destroy(struct lws_context *context);
struct lws *lws_client_connect_via_info(const struct lws_client_connect_info *ccinfo);
int lws_service(struct lws_context *context, int timeout_ms);
int lws_write(struct lws *wsi, unsigned char *buf, size_t len, enum lws_write_protocol protocol);
```

---

## 行为一致性

### 与上游行为一致

OpenHarmony 使用的 libwebsockets API 行为与上游完全一致：

| 行为 | OH 实现 | 上游行为 | 一致性 |
|-----|---------|---------|-------|
| WebSocket 握手 | 标准 RFC 6455 | 标准 RFC 6455 | ✅ 一致 |
| 帧格式 | 标准 WebSocket 帧 | 标准 WebSocket 帧 | ✅ 一致 |
| 回调时机 | 事件驱动 | 事件驱动 | ✅ 一致 |
| TLS 握手 | OpenSSL | OpenSSL | ✅ 一致 |
| HTTP 升级 | 标准 Upgrade | 标准 Upgrade | ✅ 一致 |

### OH 特有的封装层

虽然 libwebsockets API 本身未修改，但 OH 在之上添加了多层封装：

```
应用层 API
    ↓
OH 封装层 (websocket_native)
    - 连接池管理
    - 错误码映射
    - 线程安全封装
    ↓
libwebsockets API (标准)
    - lws_create_context()
    - lws_client_connect_via_info()
    - lws_service()
    - ...
    ↓
系统调用
```

**封装层提供的额外功能**:
1. **连接池**: 管理多个 WebSocket 连接
2. **错误映射**: 将 lws 错误码转换为 OH 错误码
3. **线程安全**: 封装异步回调处理
4. **资源管理**: 自动清理和生命周期管理

---

## OH 特有的配置

### 编译时配置

通过 BUILD.gn 的 `defines` 传递配置：

```gn
defines = [
  "OHOS_LIBWEBSOCKETS=1",              # OH 构建标识
  "OPENSSL_SUPPRESS_DEPRECATED",        # 抑制 OpenSSL 弃用警告
  "LWS_DETECTED_PLAT_IOS=1",           # iOS 平台（条件）
  "CROSS_PLATFORM_IOS_LIBWEBSOCKETS=1", # iOS 跨平台（条件）
]
```

**OHOS_LIBWEBSOCKETS=1**:
- 允许代码中检测 OH 环境
- 目前未在 libwebsockets 内部使用
- 保留给未来可能的 OH 特定优化

### 运行时配置

通过 `lws_context_creation_info` 结构体配置：

```c
struct lws_context_creation_info info;
memset(&info, 0, sizeof(info));

// OH 常用配置
info.port = CONTEXT_PORT_NO_LISTEN;  // 客户端模式
info.protocols = protocols;
info.gid = -1;
info.uid = -1;
info.options = LWS_SERVER_OPTION_DO_SSL_GLOBAL_INIT;
```

---

## 上层 API 映射

### C API 到 libwebsockets

| OH C API | libwebsockets API | 说明 |
|---------|------------------|-----|
| `OH_WebSocket_Create()` | `lws_create_context()` | 创建 WebSocket 实例 |
| `OH_WebSocket_Connect()` | `lws_client_connect_via_info()` | 建立连接 |
| `OH_WebSocket_Send()` | `lws_write()` | 发送数据 |
| `OH_WebSocket_Close()` | `lws_context_destroy()` | 关闭连接 |
| `OH_WebSocket_SetOnMessage()` | 回调注册 | 设置消息回调 |

### JS/ArkTS API 到 Native

```javascript
// ArkTS 应用代码
let ws = webSocket.createWebSocket();
ws.connect('wss://example.com');
ws.send('Hello');
```

```cpp
// Native 层转换
// 1. createWebSocket() → lws_create_context()
// 2. connect() → lws_client_connect_via_info()
// 3. send() → lws_write()
// 4. on('message') → lws_callback_function (LWS_CALLBACK_CLIENT_RECEIVE)
```

---

## 兼容性说明

### 版本兼容性

当前 OH 使用的版本：4.3.3

**升级兼容性考虑**:
1. **API 兼容性**: libwebsockets 遵循语义化版本，4.x 版本 API 稳定
2. **行为兼容性**: 需要验证 WebSocket 握手、重连等行为
3. **回调兼容性**: 确保回调顺序和参数一致

### 平台兼容性

| 平台 | 兼容性 | 说明 |
|-----|-------|-----|
| Android | ✅ 完全兼容 | 标准 Linux 平台 |
| iOS | ✅ 兼容 | 通过 vhost_ios.c patch |
| Linux | ✅ 完全兼容 | 原生支持 |
| Windows | ✅ 兼容 | IDE Previewer 使用 |

---

## 未启用的功能

以下 libwebsockets 功能在 OH 中未启用：

### 协议支持

- **MQTT**: 未编译相关源文件
- **Raw Socket**: 未暴露给应用层
- **Raw File**: 未使用

### 角色支持

- **Server Mode**: 主要用于客户端，服务器功能有限
- **CGI**: 未启用
- **DBus**: 未启用

### 扩展功能

- **Secure Streams**: 高级 API，未启用
- **插件系统**: 未启用
- **LEJP JSON 解析**: 使用系统 JSON 库
- **线程池**: 未启用

**原因**:
1. **精简体积**: 减少不必要的代码
2. **安全考虑**: 减少攻击面
3. **需求匹配**: 当前应用主要需要客户端 WebSocket

---

## 开发注意事项

### 直接使用 libwebsockets

如果需要在 OH 中直接使用 libwebsockets（不通过封装层）：

```gn
# 在 BUILD.gn 中添加依赖
deps = [ "//third_party/libwebsockets:websockets" ]
```

```c
#include <libwebsockets.h>

// 标准 libwebsockets API 使用
```

**注意**:
- 直接使用时需自行处理线程安全和资源管理
- 建议优先使用系统提供的封装 API

### 调试支持

libwebsockets 提供详细的日志：

```c
// 启用调试日志
lws_set_log_level(LLL_ERR | LLL_WARN | LLL_NOTICE | LLL_INFO | LLL_DEBUG, 
                  NULL);
```

---

## 总结

### API 差异总结

| 方面 | OH 状态 | 说明 |
|-----|--------|-----|
| **新增 API** | ❌ 无 | 未添加 OH 特有 API |
| **修改 API** | ❌ 无 | 未修改现有 API 签名 |
| **禁用 API** | ⚠️ 部分 | 通过不编译相关源文件实现 |
| **行为变更** | ❌ 无 | 核心行为与上游一致 |
| **封装层** | ✅ 有 | websocket_native 提供上层封装 |

### 关键结论

libwebsockets 在 OpenHarmony 中以**标准方式**使用：

1. **零 API 差异**: 未修改原始库的任何 API
2. **纯封装模式**: 通过上层封装提供 OH 特定功能
3. **子集使用**: 仅使用客户端 WebSocket 相关功能
4. **行为一致**: 协议实现与上游完全一致

---

*API 差异分析完成。下一节将分析安全风险。*
