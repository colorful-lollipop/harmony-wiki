# 03 - OH 构建适配

## 概述

OpenHarmony 使用 **GN (Generate Ninja)** 构建系统替代了 libwebsockets 原生的 CMake。本节详细说明 BUILD.gn 的配置和与上游构建系统的差异。

---

## BUILD.gn 结构

### 文件位置
```
third_party/libwebsockets/BUILD.gn
```

### 提供的 Target

| Target | 类型 | 用途 |
|--------|------|-----|
| `websockets` | `ohos_static_library` | 完整功能，系统组件使用 |
| `websockets_static` | `ohos_static_library` | 精简功能，IDE 工具使用 |
| `websocket_config` | `config` | 编译选项配置 |
| `websockets_public_config` | `config` | 公共头文件路径配置 |

---

## Target: websockets（主库）

### 源文件组织

**核心网络功能** (`lib/core-net/`):
```gn
sources = [
  "//third_party/libwebsockets/lib/core-net/adopt.c",
  "//third_party/libwebsockets/lib/core-net/client/client.c",
  "//third_party/libwebsockets/lib/core-net/client/conmon.c",
  "//third_party/libwebsockets/lib/core-net/client/connect.c",
  "//third_party/libwebsockets/lib/core-net/client/connect2.c",
  "//third_party/libwebsockets/lib/core-net/client/connect3.c",
  "//third_party/libwebsockets/lib/core-net/client/connect4.c",
  "//third_party/libwebsockets/lib/core-net/client/sort-dns.c",
  "//third_party/libwebsockets/lib/core-net/close.c",
  "//third_party/libwebsockets/lib/core-net/dummy-callback.c",
  "//third_party/libwebsockets/lib/core-net/network.c",
  "//third_party/libwebsockets/lib/core-net/output.c",
  "//third_party/libwebsockets/lib/core-net/pollfd.c",
  "//third_party/libwebsockets/lib/core-net/service.c",
  "//third_party/libwebsockets/lib/core-net/sorted-usec-list.c",
  "//third_party/libwebsockets/lib/core-net/state.c",
  "//third_party/libwebsockets/lib/core-net/wsi-timeout.c",
  "//third_party/libwebsockets/lib/core-net/wsi.c",
]
```

**平台适配** (`lib/plat/unix/`):
```gn
"//third_party/libwebsockets/lib/plat/unix/unix-caps.c",
"//third_party/libwebsockets/lib/plat/unix/unix-fds.c",
"//third_party/libwebsockets/lib/plat/unix/unix-file.c",
"//third_party/libwebsockets/lib/plat/unix/unix-init.c",
"//third_party/libwebsockets/lib/plat/unix/unix-misc.c",
"//third_party/libwebsockets/lib/plat/unix/unix-pipe.c",
"//third_party/libwebsockets/lib/plat/unix/unix-service.c",
"//third_party/libwebsockets/lib/plat/unix/unix-sockets.c",
```

**TLS/SSL 支持** (`lib/tls/openssl/`):
```gn
"//third_party/libwebsockets/lib/tls/openssl/openssl-client.c",
"//third_party/libwebsockets/lib/tls/openssl/openssl-server.c",
"//third_party/libwebsockets/lib/tls/openssl/openssl-session.c",
"//third_party/libwebsockets/lib/tls/openssl/openssl-ssl.c",
"//third_party/libwebsockets/lib/tls/openssl/openssl-tls.c",
"//third_party/libwebsockets/lib/tls/openssl/openssl-x509.c",
```

**协议实现**:
- `lib/roles/h1/` - HTTP/1.x
- `lib/roles/h2/` - HTTP/2
- `lib/roles/ws/` - WebSocket
- `lib/roles/http/` - HTTP 通用

### iOS 平台特殊处理

#### 动态 Patch 应用

```gn
if (target_os == "ios") {
  libwebsockets_path = rebase_path("//third_party/libwebsockets")
  exec_script("for_ios.sh", [ "$libwebsockets_path" ])
}
```

**说明**:
- 构建时执行 `for_ios.sh` 脚本
- 脚本复制 `vhost.c` → `vhost_ios.c` 并应用 patch
- 需要 GN 白名单授权：`//build/core/gn/ohos_exec_script_allowlist.gni`

#### iOS 源文件排除

```gn
if (target_os != "ios") {
  sources += [
    "//third_party/libwebsockets/lib/core-net/route.c",
    "//third_party/libwebsockets/lib/roles/netlink/ops-netlink.c",
  ]
}
```

**说明**:
- `route.c`: Linux 路由表操作（iOS 不支持）
- `ops-netlink.c`: Netlink 协议实现（Linux 特有）

#### iOS 源文件选择

```gn
if (target_os == "ios") {
  sources += [ "//third_party/libwebsockets/lib/core-net/vhost_ios.c" ]
} else {
  sources += [ "//third_party/libwebsockets/lib/core-net/vhost.c" ]
}
```

### ArkUI-X 特殊处理

```gn
if (is_arkui_x) {
  if (target_os == "ios") {
    deps += [
      "//third_party/openssl:libcrypto_static",
      "//third_party/openssl:libssl_static",
    ]
  } else {
    deps += [
      "//third_party/openssl:libcrypto_shared",
      "//third_party/openssl:libssl_shared",
    ]
  }
  include_dirs += [
    "//third_party/openssl/include/openssl",
    "//third_party/openssl/crypto/evp",
  ]
}
```

**说明**:
- iOS 使用静态 OpenSSL（App Store 要求）
- 其他平台使用共享库（节省内存）

### 编译定义（Defines）

```gn
defines = [
  "OHOS_LIBWEBSOCKETS=1",              # OH 构建标识
  "OPENSSL_SUPPRESS_DEPRECATED",        # 抑制 OpenSSL 弃用警告
]

if (target_os == "ios") {
  defines += [
    "LWS_DETECTED_PLAT_IOS=1",          # iOS 平台检测
    "CROSS_PLATFORM_IOS_LIBWEBSOCKETS=1", # 跨平台 iOS 支持
  ]
}
```

### 编译标志（CFlags）

```gn
cflags = [
  "-fPIC",                    # 位置无关代码
  "-Os",                      # 优化大小
  "-g",                       # 调试信息
  "-Wall",                    # 启用所有警告
  "-fno-strict-aliasing",     # 禁用严格别名优化
  "-fvisibility=hidden",      # 默认隐藏符号
  "-Wmissing-declarations",   # 缺失声明警告
  "-Waggregate-return",       # 结构体返回警告
  "-pipe",                    # 管道编译
]
```

### 外部依赖

```gn
external_deps = [
  "openssl:libcrypto_shared",
  "openssl:libssl_shared",
  "zlib:libz",
]
```

---

## Target: websockets_static（IDE 工具库）

### 用途
专为 IDE Previewer 工具提供的精简版本。

### 与主库的差异

1. **缺少的源文件**:
   - `lib/core-net/client/conmon.c` - 连接监控
   - `lib/core-net/route.c` - 路由表（本来就不在 iOS）
   - `lib/core-net/vhost_ios.c` vs `vhost.c`（统一使用 vhost.c）
   - `lib/misc/cache-ttl/*` - 缓存 TTL
   - `lib/misc/lws_map.c` - 映射表
   - `lib/misc/dir.c` - 目录操作
   - `lib/misc/prng.c` - 伪随机数
   - `lib/roles/http/cookie.c` - Cookie 处理
   - `lib/roles/http/server/lejp-conf.c` - 配置解析
   - `lib/roles/http/server/lws-spa.c` - SPA 支持
   - `lib/roles/netlink/ops-netlink.c` - Netlink
   - `lib/roles/pipe/ops-pipe.c` - 管道
   - `lib/roles/raw-file/ops-raw-file.c` - 原始文件
   - `lib/system/smd/smd.c` - 系统消息分发
   - `lib/tls/*` - 完整 TLS 实现

2. **平台适配**:
   - 支持 Windows (`plat/windows/`)
   - 支持 Unix (`plat/unix/`)

3. **编译选项**:
   ```gn
   configs = [ ":websocket_config" ]
   ```

### 为什么需要精简版本？

Previewer 工具的需求：
1. **减小体积**: IDE 工具包需要保持小巧
2. **功能有限**: Previewer 主要需要基础 WebSocket 客户端功能
3. **快速启动**: 减少不必要的初始化

---

## Config: websocket_config

### 编译警告配置

```gn
config("websocket_config") {
  cflags = [
    "-Wall",
    "-Wsign-compare",
    "-Wstrict-aliasing",
    "-Wuninitialized",
    "-fvisibility=hidden",
    "-Wtype-limits",
    "-Wignored-qualifiers",
    "-Wno-deprecated-declarations",    # 禁用弃用声明警告
    "-pthread",
    "-Wno-unused-command-line-argument",
    "-Wno-unused-parameter",
    "-Wno-implicit-function-declaration",
  ]
}
```

---

## Config: websockets_public_config

### 公共头文件路径

提供给依赖该库的其他模块使用的头文件搜索路径：

```gn
config("websockets_public_config") {
  include_dirs = [
    "//third_party/libwebsockets/plugins",
    "//third_party/libwebsockets/lib/core",
    "//third_party/libwebsockets/lib/core-net",
    "//third_party/libwebsockets/lib/event-libs",
    "//third_party/libwebsockets/lib/abstract",
    "//third_party/libwebsockets/lib/tls",
    "//third_party/libwebsockets/lib/roles",
    "//third_party/libwebsockets/lib/event-libs/libuv",
    "//third_party/libwebsockets/lib/event-libs/poll",
    "//third_party/libwebsockets/lib/event-libs/libevent",
    "//third_party/libwebsockets/lib/event-libs/libev",
    "//third_party/libwebsockets/lib/jose/jwe",
    "//third_party/libwebsockets/lib/jose/jws",
    "//third_party/libwebsockets/lib/jose",
    "//third_party/libwebsockets/lib/misc",
    "//third_party/libwebsockets/lib/roles/http",
    "//third_party/libwebsockets/lib/roles/http/compression",
    "//third_party/libwebsockets/lib/roles/h1",
    "//third_party/libwebsockets/lib/roles/h2",
    "//third_party/libwebsockets/lib/roles/ws",
    "//third_party/libwebsockets/lib/roles/cgi",
    "//third_party/libwebsockets/lib/roles/dbus",
    "//third_party/libwebsockets/lib/roles/raw-proxy",
    "//third_party/libwebsockets/lib/abstract",
    "//third_party/libwebsockets/lib/system/async-dns",
    "//third_party/libwebsockets/lib/roles/mqtt",
    "//third_party/libwebsockets/lib/system/metrics",
    "//third_party/libwebsockets/lib",
    "//third_party/libwebsockets/win32port/win32helpers",
    "//third_party/libwebsockets/include",
  ]
}
```

### 平台特定路径

```gn
if (platform == "mingw_x86_64") {
  include_dirs += [ "//third_party/libwebsockets/lib/plat/windows" ]
} else if (platform == "mac_arm64" || platform == "mac_x64" ||
           platform == "linux_x64" || platform == "linux_arm64") {
  include_dirs += [ "//third_party/libwebsockets/lib/plat/unix" ]
}

cflags = [ "-Wno-error=#warnings" ]
```

---

## 与上游 CMake 的差异

### 功能选择

| 功能 | 上游 CMake | OH BUILD.gn |
|-----|-----------|-------------|
| WebSocket 客户端 | ✅ | ✅ |
| WebSocket 服务器 | ✅ | ⚠️ 客户端为主 |
| HTTP/1 客户端 | ✅ | ✅ |
| HTTP/1 服务器 | ✅ | ✅ |
| HTTP/2 客户端 | ✅ | ✅ |
| TLS/OpenSSL | ✅ | ✅ |
| MQTT | 可选 | ❌ 未启用 |
| libuv 事件循环 | 可选 | ❌ 使用 poll |
| 服务器配置 | 丰富 | 基础 |

### 平台支持

| 平台 | 上游 | OH |
|-----|------|-----|
| Linux | ✅ | ✅ |
| Windows | ✅ | ✅ (IDE 工具) |
| macOS | ✅ | ✅ (IDE 工具) |
| iOS | ⚠️ 部分 | ✅ (ArkUI-X) |
| Android | ✅ | ✅ |
| FreeRTOS | ✅ | ❌ |
| ESP32 | ✅ | ❌ |

### 构建特点对比

| 特性 | CMake | GN |
|-----|-------|-----|
| 配置方式 | 选项开关 | 硬编码源文件列表 |
| 条件编译 | `option()` | GN 条件 + defines |
| 跨平台 | CMake 处理 | GN 显式条件 |
| 动态 patch | 不支持 | `exec_script` 支持 |
| 模块化 | 模块化选项 | 显式源文件选择 |

---

## Inner Kits 配置

在 `bundle.json` 中定义了公共 API：

```json
"inner_kits": [
  {
    "name": "//third_party/libwebsockets:websockets",
    "header": {
      "header_files": [],
      "header_base": [
        "//third_party/libwebsockets/plugins",
        "//third_party/libwebsockets/lib/core",
        "//third_party/libwebsockets/lib/core-net",
        // ... 更多路径
        "//third_party/libwebsockets/include"
      ]
    }
  }
]
```

---

## 构建使用示例

### 依赖声明

```gn
# 在其他模块的 BUILD.gn 中
deps = [
  "//third_party/libwebsockets:websockets",
]
```

### IDE 工具使用

```gn
# IDE Previewer
deps = [
  "//third_party/libwebsockets:websockets_static",
]
```

---

## 总结

BUILD.gn 的关键 OH 定制化：

1. **iOS 动态 Patch**: 构建时应用平台特定修复
2. **双 Target 设计**: 系统和工具使用不同版本
3. **显式源文件**: 精确控制编译内容
4. **平台条件**: GN 条件处理跨平台差异
5. **ArkUI-X 适配**: iOS 静态 OpenSSL 链接

---

*BUILD.gn 分析完成。下一节将分析依赖关系和使用场景。*
