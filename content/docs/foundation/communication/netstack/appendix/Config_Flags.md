# OpenHarmony NetStack 配置开关与宏

## 1. 编译时宏定义

### 1.1 平台相关宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `WINDOWS_PLATFORM` | `http/BUILD.gn:49` | Windows 平台构建 |
| `MAC_PLATFORM` | `http/BUILD.gn:52` | macOS 平台构建 |
| `HTTP_PROXY_ENABLE` | `http/BUILD.gn:58` | HTTP 代理功能启用 |

### 1.2 功能开关宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `HTTP_MULTIPATH_CERT_ENABLE` | `http/BUILD.gn:68` | 多路径证书功能 |
| `HTTP_ONLY_VERIFY_ROOT_CA_ENABLE` | `http/BUILD.gn:69` | 仅验证根 CA |
| `HTTP_CACHE_FILE_PATH_USE_BASE` | `netstack_config.gni:27` | BoringSSL 缓存路径 |

### 1.3 网络管理器集成宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `HAS_NETMANAGER_BASE` | `http/BUILD.gn:169` | 集成 netmanager_base |
| `HAS_NETSTACK_CHR` | `http/BUILD.gn:170` | 启用云端健康报告 |
| `ENABLE_HTTP_INTERCEPT` | `http/BUILD.gn:171` | HTTP 拦截功能 |
| `HTTP_HANDOVER_FEATURE` | `http/BUILD.gn:182` | HTTP 切换功能 |

### 1.4 安全相关宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `OH_CORE_NETSTACK_PERMISSION_CHECK` | `common_utils.cpp:148` | 启用权限检查 |
| `FUZZ_TEST` | `common_utils.cpp:150` | Fuzz 测试模式 |
| `DT_TEST` | `common_utils.cpp:153` | DT 测试模式 |

---

## 2. 构建配置参数

### 2.1 GN 参数

**文件**: `netstack_config.gni`

```gn
declare_args() {
  netstack_http_boringssl = false  # 使用 BoringSSL
}
```

**Bundle 配置** (`bundle.json`)

```json
"features": [
  "netstack_feature_http3",
  "netstack_http_boringssl",
  "netstack_feature_communication_http3"
]
```

### 2.2 编译器配置

```gn
# 必需安全标志
cflags_cc = [
  "-fstack-protector-strong",    # 堆栈保护
  "-D_FORTIFY_SOURCE=2",         # 缓冲区溢出检测
  "-O2",                         # 优化级别
]

# CFI 保护 (生产构建)
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}

# PAC 指针认证
branch_protector_ret = "pac_ret"
```

---

## 3. 运行时配置

### 3.1 SSL/TLS 常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `SYSPRECAPATH` | `/etc/security/certificates/user/` | 系统 CA 证书路径 |
| `USERINSTALLEDCAPATH` | `/data/certificates/user_cacerts/` | 用户 CA 证书路径 |
| `UIDTRANSFORMDIVISOR` | `200000` | UID 转换除数 |

**文件**: `frameworks/native/net_ssl/net_ssl_verify_cert.cpp`

```cpp
constexpr char SYSPRECAPATH[] = "/etc/security/certificates/user/";
constexpr char USERINSTALLEDCAPATH[] = "/data/certificates/user_cacerts/";
constexpr int UIDTRANSFORMDIVISOR = 200000;
```

### 3.2 权限组 ID

| 常量 | 值 | 说明 |
|------|-----|------|
| `inetGroup` | `40002003` | INTERNET 权限组 ID |

**文件**: `utils/common_utils/src/netstack_common_utils.cpp`

```cpp
constexpr int inetGroup = 40002003;  // 3003 in gateway shell
```

### 3.3 TLS 配置常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `ALPN_PROTOCOLS_HTTP_1_1` | `"http1.1"` | HTTP/1.1 ALPN |
| `ALPN_PROTOCOLS_HTTP_2` | `"h2"` | HTTP/2 ALPN |
| `MAX_ERR_LEN` | `1024` | 错误消息最大长度 |
| `DEFAULT_TIMEOUT_TLS` | `0` | 默认 TLS 超时 |
| `NO_TIMEOUT` | `1` | 无超时 |
| `TLS_TIMEOUT` | `2` | TLS 超时 |

---

## 4. 错误码常量

### 4.1 权限错误

| 常量 | 值 | 说明 |
|------|-----|------|
| `PERMISSION_DENIED_CODE` | `201` | 权限拒绝错误码 |
| `PERMISSION_DENIED_MSG` | `"Permission denied"` | 权限拒绝消息 |

**文件**: `utils/napi_utils/include/base_context.h:34-35`

### 4.2 WebSocket 错误码

| 常量 | 值 | 说明 |
|------|-----|------|
| `WEBSOCKET_ERROR_PERMISSION_DENIED` | 无权限 | WebSocket 权限错误 |
| `WEBSOCKET_ERROR_DISALLOW_HOST` | 原子服务主机名不允许 | 主机名限制 |

**文件**: `frameworks/native/websocket_native/include/websocket_client_error.h`

---

## 5. 功能标志

### 5.1 条件编译

```cpp
#if HAS_NETMANAGER_BASE
    // 集成 netmanager_base 的代码
    external_deps += [
        "netmanager_base:net_conn_manager_if",
        "netmanager_base:net_security_config_if",
    ]
#else
    // 独立模式代码
    defines += [ "HAS_NETMANAGER_BASE=0" ]
#endif
```

### 5.2 HTTP 缓存配置

```cpp
if (netstack_http_boringssl) {
  defines += [ "HTTP_CACHE_FILE_PATH_USE_BASE" ]
}
```

---

## 6. 产品相关配置

### 6.1 SDK 构建

```gn
if (product_name == "ohos-sdk") {
  # SDK 构建禁用某些功能
  defines -= [ "HTTP_PROXY_ENABLE" ]
}
```

### 6.2 非 SDK 构建

```gn
if (product_name != "ohos-sdk") {
  defines += [ "HTTP_PROXY_ENABLE" ]
  cflags_cc += [
    "-fstack-protector-strong",
    "-D_FORTIFY_SOURCE=2",
    "-O2",
  ]
}
```

---

## 7. 外部依赖配置

### 7.1 必需依赖

| 依赖 | 用途 | 关键库 |
|------|------|--------|
| curl | HTTP 客户端 | libcurl.so |
| openssl | TLS/SSL | libcrypto.so, libssl.so |
| napi | Node-API | ace_napi |
| ipc | 进程间通信 | ipc_single |

### 7.2 可选依赖

| 依赖 | 条件 | 用途 |
|------|------|------|
| netmanager_base | HAS_NETMANAGER_BASE=1 | 网络连接管理 |
| bundle_framework | - | 应用框架 |
| samgr | - | SA 管理器 |
| hilog | - | 日志系统 |

---

## 相关文档
- [Architecture](01_Architecture.md)
- [N-API Reference](02_N-API_Reference.md)
- [Native API](03_Inner_API.md)
- [Build Targets](04_Build_Targets.md)
- [Security Review](05_Security_Review.md)
- [Callgraphs](Callgraphs.md)
