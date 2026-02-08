# OpenHarmony NetStack GN 构建目标

## 1. 构建配置概览

### 1.1 配置文件

| 文件 | 说明 |
|------|------|
| `bundle.json` | 模块清单定义 |
| `netstack_config.gni` | 模块级 GN 配置 |
| `BUILD.gn` | 各模块构建定义 |

### 1.2 全局配置参数

**文件**: `netstack_config.gni`

```gn
declare_args() {
  netstack_http_boringssl = false  # 使用 BoringSSL 而非 OpenSSL
}
```

### 1.3 编译器安全标志

**文件**: `frameworks/js/napi/http/BUILD.gn:206-210`

```gn
cflags = [
  "-fstack-protector-strong",    # 堆栈保护
  "-D_FORTIFY_SOURCE=2",         # 缓冲区溢出检测
  "-O2",                         # 优化级别
]
```

---

## 2. 主要构建目标

### 2.1 Frameworks - JS/NAPI 模块

| Target | 类型 | 输出 | 位置 |
|--------|------|------|------|
| `http` | ohos_shared_library | libhttp.so | frameworks/js/napi/http |
| `socket` | ohos_shared_library | libsocket.so | frameworks/js/napi/socket |
| `websocket` | ohos_shared_library | libwebsocket.so | frameworks/js/napi/websocket |
| `fetch` | ohos_shared_library | libfetch.so | frameworks/js/napi/fetch |
| `networksecurity_napi` | ohos_shared_library | libnetworksecurity_napi.so | frameworks/js/napi/net_ssl |

#### HTTP 模块详细配置

**文件**: `frameworks/js/napi/http/BUILD.gn:78-215`

```gn
ohos_shared_library("http") {
  # CFI (Control Flow Integrity) 保护
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  
  # PACRET (Pointer Authentication) 保护
  branch_protector_ret = "pac_ret"
  
  # 源文件
  sources = [
    "async_context/src/request_context.cpp",
    "async_work/src/http_async_work.cpp",
    "cache/cache_proxy/src/cache_proxy.cpp",
    "cache/cache_strategy/src/http_cache_strategy.cpp",
    "http_exec/src/http_exec.cpp",
    "http_module/src/http_module.cpp",
    # ... 更多源文件
  ]
  
  # 包含目录
  include_dirs = [
    "async_context/include",
    "async_work/include",
    "cache/cache_strategy/include",
    "$NETSTACK_DIR/utils/common_utils/include",
  ]
  
  # 配置
  configs = [ ":http_config" ]
  
  # 外部依赖
  external_deps = [
    "curl:curl_shared",           # HTTP 客户端库
    "openssl:libcrypto_shared",    # 加密库
    "openssl:libssl_shared",       # SSL/TLS 库
    "napi:ace_napi",               # N-API 框架
    "ipc:ipc_single",             # IPC 框架
    "samgr:samgr_proxy",           # SA 管理器客户端
  ]
  
  # 条件依赖
  if (defined(global_parts_info) && 
      global_parts_info.communication_netmanager_base) {
    external_deps += [
      "netmanager_base:net_conn_manager_if",
      "netmanager_base:net_security_config_if",
    ]
    defines += [ "HAS_NETMANAGER_BASE=1" ]
  } else {
    defines += [ "HAS_NETMANAGER_BASE=0" ]
  }
  
  # 安装路径
  relative_install_dir = "module/net"
  part_name = "netstack"
  subsystem_name = "communication"
}
```

#### Socket 模块详细配置

**文件**: `frameworks/js/napi/socket/BUILD.gn:62-169`

```gn
ohos_shared_library("socket") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  branch_protector_ret = "pac_ret"
  
  # 包含 TLS 和 Proxy 子模块
  sources = [
    "async_context/src/bind_context.cpp",
    "async_context/src/connect_context.cpp",
    "socket_exec/src/socket_exec.cpp",
    "$NETSTACK_DIR/frameworks/js/napi/tls/src/tlssocket_module.cpp",
    "$NETSTACK_DIR/frameworks/js/napi/proxy/src/socks5_instance.cpp",
    # ... 更多源文件
  ]
  
  external_deps = [
    "hilog:libhilog",
    "napi:ace_napi",
    "openssl:libcrypto_shared",
    "openssl:libssl_shared",
    "samgr:samgr_proxy",
  ]
  
  defines = [ "OPENSSL_SUPPRESS_DEPRECATED" ]
}
```

### 2.2 Frameworks - Native 模块

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `http_client` | ohos_shared_library | libhttp_client.so | HTTP Native 客户端 |
| `net_ssl` | ohos_shared_library | libnet_ssl.so | SSL/TLS Native |
| `websocket_native` | ohos_shared_library | libwebsocket_native.so | WebSocket Native |
| `http_interceptor` | ohos_shared_library | libhttp_interceptor.so | HTTP 拦截器 |

### 2.3 Interfaces - NDK 模块

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `net_http_ndk` | ohos_shared_library | libnet_http_ndk.so | HTTP C API (NDK) |
| `net_ssl_ndk` | ohos_shared_library | libnet_ssl_ndk.so | SSL C API (NDK) |
| `net_websocket` | ohos_shared_library | libnet_websocket.so | WebSocket C API (NDK) |

**文件**: `interfaces/kits/c/net_http/BUILD.gn`

```gn
ohos_shared_library("net_http_ndk") {
  sources = [ "src/net_http_c.cpp" ]
  
  include_dirs = [ "include" ]
  
  external_deps = [
    "napi:ace_napi",
    "curl:curl_shared",
    "openssl:libcrypto_shared",
    "openssl:libssl_shared",
  ]
  
  relative_install_dir = "ndk"
}
```

### 2.4 Utils 模块

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `stack_utils_common` | ohos_shared_library | libstack_utils_common.so | 公共工具库 |
| `napi_utils` | ohos_shared_library | libnapi_utils.so | N-API 工具 |

---

## 3. 条件编译配置

### 3.1 平台相关配置

**文件**: `frameworks/js/napi/http/BUILD.gn:42-64`

```gn
config("http_config") {
  if (is_mingw || is_mac) {
    cflags = [ "-std=c++17", "-stdlib=libc++" ]
  }
  
  if (is_mingw) {
    defines += [ "WINDOWS_PLATFORM" ]
  } else if (is_mac) {
    defines += [ "MAC_PLATFORM" ]
  } else {
    defines += [ "HTTP_PROXY_ENABLE" ]
    cflags_cc = [
      "-fstack-protector-strong",
      "-D_FORTIFY_SOURCE=2",
      "-O2",
    ]
  }
  
  if (product_name != "ohos-sdk") {
    defines += [
      "HTTP_MULTIPATH_CERT_ENABLE",   # 多路径证书
      "HTTP_ONLY_VERIFY_ROOT_CA_ENABLE",  # 仅验证根 CA
    ]
  }
}
```

### 3.2 Feature 配置

```gn
# bundle.json 中定义的功能开关
"features": [
  "netstack_feature_http3",      # HTTP/3 支持
  "netstack_http_boringssl",     # BoringSSL
  "netstack_feature_communication_http3"
]
```

---

## 4. 依赖关系

### 4.1 内部依赖

```
http ─────► http_client ─────► curl
          │                   ├── openssl
          │                   └── zlib
          │
          └──► napi_utils ───► hilog
                           ├── ipc
                           └── samgr_proxy
```

### 4.2 外部依赖

| 依赖组件 | 用途 | 关键库 |
|----------|------|--------|
| curl | HTTP 客户端 | libcurl.so |
| openssl | TLS/SSL | libcrypto.so, libssl.so |
| napi | Node-API | ace_napi |
| ipc | 进程间通信 | ipc_single |
| samgr | SA 管理 | samgr_proxy |
| hilog | 日志 | libhilog |
| zlib | 压缩 | libz.so |

---

## 5. 安装路径

### 5.1 标准系统

 | 安装路径 |
|------|----------|
| lib| 产物http.so | /system/lib/module/net/ |
| libsocket.so | /system/lib/module/net/ |
| libwebsocket.so | /system/lib/module/net/ |
| libnetworksecurity_napi.so | /system/lib/module/net/ |

### 5.2 NDK

| 产物 | 安装路径 |
|------|----------|
| libnet_http_ndk.so | /system/lib/ndk/ |
| libnet_ssl_ndk.so | /system/lib/ndk/ |
| libnet_websocket.so | /system/lib/ndk/ |

---

## 6. 安全构建标志

### 6.1 必需安全标志

```gn
# 堆栈保护
-fstack-protector-strong

# 缓冲区溢出检测
-D_FORTIFY_SOURCE=2

# CFI 保护 (生产构建)
cfi = true
cfi_cross_dso = true

# PAC 指针认证
branch_protector_ret = "pac_ret"

# 禁用不安全的警告
-Wno-unused-result
```

### 6.2 编译器优化

```gn
# 发布构建优化
if (optimize) {
  cflags_cc += [
    "-O2",
    "-fno-omit-frame-pointer",
  ]
}
```

---

## 7. 测试构建

### 7.1 单元测试

| Target | 类型 | 说明 |
|--------|------|------|
| `http_unittest` | ohos_unittest | HTTP 模块测试 |
| `socket_unittest` | ohos_unittest | Socket 模块测试 |
| `netssl_unittest` | ohos_unittest | SSL 模块测试 |
| `websocket_unittest` | ohos_unittest | WebSocket 模块测试 |

### 7.2 Fuzz 测试

| Target | 类型 | 说明 |
|--------|------|------|
| `HttpFuzzTest` | ohos_fuzztest | HTTP Fuzz 测试 |
| `SocketExecFuzzTest` | ohos_fuzztest | Socket Fuzz 测试 |
| `NetsslExecFuzzTest` | ohos_fuzztest | SSL Fuzz 测试 |

---

## 相关文档
- [Architecture](01_Architecture.md)
- [N-API Reference](02_N-API_Reference.md)
- [Native API](03_Inner_API.md)
- [Security Review](05_Security_Review.md)
