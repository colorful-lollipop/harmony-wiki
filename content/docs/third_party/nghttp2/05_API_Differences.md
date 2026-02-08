# API/接口差异

## 概述

nghttp2 在 OpenHarmony 中**没有**添加 OH 特定的 API，也没有修改现有 API 的行为。本文档说明 OH 配置下的 API 特性。

**状态**: ✅ 与上游完全一致

---

## OH 配置下的 API 可用性

### 标准 API

所有 nghttp2 公共 API 在 OH 中均可用：

```c
#include <nghttp2/nghttp2.h>

// 会话管理
nghttp2_session_client_new2();
nghttp2_session_server_new2();
nghttp2_session_del();

// 请求/响应提交
nghttp2_submit_request();
nghttp2_submit_response();
nghttp2_submit_headers();
nghttp2_submit_data();

// HPACK 编解码
nghttp2_hd_deflate_new();
nghttp2_hd_inflate_new();

// 其他...
```

### 特定于 OH 构建的配置

通过 `defines` 启用的功能：

| 宏 | 定义 | 影响 |
|----|------|------|
| `HAVE_ARPA_INET_H=1` | 是 | 使用 arpa/inet.h 的网络函数 |
| `HAVE_NETINET_IN_H=1` | 是 | 使用 netinet/in.h 的结构体 |
| `HAVE_TIME_H=1` | 是 | 使用时间相关函数 |

**说明**: 这些宏影响内部实现，不改变 API 签名。

---

## 与上游的差异

### 无差异

| 方面 | 状态 | 说明 |
|------|------|------|
| API 签名 | ✅ 一致 | 所有函数签名与上游相同 |
| 行为 | ✅ 一致 | 功能行为无修改 |
| 常量 | ✅ 一致 | 宏定义和枚举值相同 |
| 结构体 | ✅ 一致 | 数据结构布局相同 |

### 构建系统差异（非 API）

| 方面 | OH | 上游 |
|------|-----|------|
| 构建工具 | GN/Ninja | CMake/Autotools |
| 可选功能 | 固定配置 | configure 选项 |
| 应用程序 | 不构建 | nghttp, nghttpd, nghttpx |
| 示例代码 | 不构建 | examples/ |

---

## API 使用示例

### 创建 HTTP/2 客户端会话

```c
#include <nghttp2/nghttp2.h>
#include <assert.h>
#include <string.h>

// 回调函数：发送数据
static ssize_t send_callback(nghttp2_session *session,
                              const uint8_t *data, size_t length,
                              int flags, void *user_data) {
    // 实际发送数据到 socket
    // return write(fd, data, length);
    return length;
}

// 回调函数：接收帧
static int on_frame_recv_callback(nghttp2_session *session,
                                   const nghttp2_frame *frame,
                                   void *user_data) {
    switch (frame->hd.type) {
        case NGHTTP2_HEADERS:
            // 处理响应头
            break;
        case NGHTTP2_DATA:
            // 处理响应数据
            break;
    }
    return 0;
}

int main() {
    nghttp2_session *session;
    nghttp2_session_callbacks *callbacks;
    
    // 创建回调
    nghttp2_session_callbacks_new(&callbacks);
    nghttp2_session_callbacks_set_send_callback(callbacks, send_callback);
    nghttp2_session_callbacks_set_on_frame_recv_callback(callbacks, 
                                                          on_frame_recv_callback);
    
    // 创建客户端会话
    nghttp2_session_client_new2(&session, callbacks, NULL, NULL);
    
    // 使用会话...
    
    // 清理
    nghttp2_session_del(session);
    nghttp2_session_callbacks_del(callbacks);
    
    return 0;
}
```

### 提交 HTTP/2 请求

```c
// 准备请求头
nghttp2_nv headers[] = {
    MAKE_NV(":method", "GET"),
    MAKE_NV(":path", "/"),
    MAKE_NV(":scheme", "https"),
    MAKE_NV(":authority", "nghttp2.org"),
    MAKE_NV("accept", "*/*"),
    MAKE_NV("user-agent", "OpenHarmony/HTTP2-Client")
};

// 提交请求
int32_t stream_id = nghttp2_submit_request(
    session,           // 会话
    NULL,              // 优先级（可选）
    headers,           // 请求头数组
    sizeof(headers) / sizeof(headers[0]),  // 头数量
    NULL,              // 数据源（GET 请求无 body）
    NULL               // 用户数据
);

// 发送请求
nghttp2_session_send(session);
```

---

## 头文件路径

### OH 中的头文件位置

```
third_party/nghttp2/lib/includes/
├── nghttp2/
│   ├── nghttp2.h          # 主头文件
│   └── nghttp2ver.h       # 版本定义
```

### 包含方式

```c
// 推荐方式
#include <nghttp2/nghttp2.h>

// 获取版本
const char *version = NGHTTP2_VERSION;  // "1.66.0"
```

---

## 版本信息

### 版本宏

```c
// nghttp2ver.h
#define NGHTTP2_VERSION "1.66.0"
#define NGHTTP2_VERSION_NUM 0x014200
```

### 运行时版本

```c
// 获取版本信息
const nghttp2_info *info = nghttp2_version(0);
printf("nghttp2 version: %s\n", info->version_str);
```

---

## 功能完整性

### 完整支持的功能

| 功能 | 状态 | 说明 |
|------|------|------|
| HTTP/2 帧层 | ✅ | 完整实现 RFC 9113 |
| HPACK 编解码 | ✅ | 完整实现 RFC 7541 |
| 流管理 | ✅ | 多路复用、优先级、流控 |
| 服务器推送 | ✅ | 支持推送 Promise |
| ALPN | ✅ | 通过底层 TLS 库 |
| 流控制 | ✅ | 连接级和流级 |

### 不构建的功能

| 功能 | 状态 | 原因 |
|------|------|------|
| nghttp 客户端 | ❌ | 示例程序 |
| nghttpd 服务器 | ❌ | 示例程序 |
| nghttpx 代理 | ❌ | 示例程序 |
| h2load 工具 | ❌ | 测试工具 |

**说明**: 这些程序在 OH 中不构建，但 libnghttp2 库本身完整，可以自行开发类似功能。

---

## 兼容性

### 二进制兼容性

- **API 版本**: 1.66.0
- **ABI 稳定性**: nghttp2 承诺向后兼容
- **升级风险**: 低

### 源代码兼容性

- **头文件**: 与上游完全一致
- **代码迁移**: 无需修改即可从上游移植

---

## 总结

nghttp2 在 OpenHarmony 中：

1. **零 API 修改**: 没有添加或修改任何 API
2. **完整功能**: 核心库功能全部可用
3. **标准兼容**: 与上游 nghttp2 100% 兼容
4. **易于移植**: 基于 nghttp2 的上游代码可直接在 OH 运行

### 使用建议

- **直接参考上游文档**: https://nghttp2.org/documentation/
- **查看头文件**: `lib/includes/nghttp2/nghttp2.h`
- **示例代码**: 参考上游 `examples/` 目录（虽然 OH 不构建）
