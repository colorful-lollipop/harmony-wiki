# 关键配置项

## GN 构建配置

### config.gni

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `sharing_framework_support_wfd` | `true` | 支持 WFD 框架 |
| `wifi_display_support_sink` | `true` | 支持 Sink 端 |
| `wifi_display_support_source` | `true` | 支持 Source 端 |
| `SHARING_ROOT_DIR` | `//foundation/CastEngine/castengine_wifi_display` | 根目录路径 |

### BUILD.gn 编译器配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `-Wall` | 启用 | 警告当错误 |
| `-Wextra` | 启用 | 额外警告 |
| `-Werror` | 启用 | 警告当错误 |
| `-D_FORTIFY_SOURCE=2` | 启用 | 运行时缓冲区检查 |
| `-fvisibility=hidden` | 启用 | 隐藏符号可见性 |
| `-O2` | 启用 | 优化级别 |

### Sanitizer 配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `cfi` | `true` | 控制流完整性 |
| `cfi_cross_dso` | `true` | 跨 DSO CFI |
| `ubsan` | `true` | 未定义行为检测 |
| `boundary_sanitize` | `true` | 边界检查 |
| `integer_overflow` | `true` | 整数溢出检测 |

## N-API 配置

### 模块注册

| 配置项 | 值 | 文件位置 |
|--------|-----|----------|
| 模块名 | `multimedia.SharingWfd` | `native_module_ohos_wfd.cpp:38` |
| 注册函数 | `Export()` | `native_module_ohos_wfd.cpp:23` |
| C++ 命名空间 | `OHOS::Sharing` | `native_module_ohos_wfd.cpp:18-19` |

### WfdSink API

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 类名 | `WfdSinkImpl` | JS 类名 |
| 构造函数引用 | `WfdSinkNapi::constructor_` | `wfd_napi_sink.cpp:25` |
| 默认 Bundle | `com.example.player` | `wfd_napi_sink.cpp:26` |
| 默认 Ability | `MainAbility` | `wfd_napi_sink.cpp:27` |
| 字符串最大长度 | 255 | `wfd_napi_sink.cpp:33` |

### WfdSource API

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 类名 | `WfdSourceImpl` | JS 类名 |
| 构造函数引用 | `WfdSourceNapi::constructor_` | `wfd_napi_source.cpp:26` |
| 默认 Bundle | `com.example.player` | `wfd_napi_source.cpp:27` |
| 默认 Ability | `MainAbility` | `wfd_napi_source.cpp:28` |

## 服务配置

### sharing_service.cfg

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 服务名 | `sharing_service` | 服务名称 |
| 用户 | `audio` | 运行用户 |
| 权限级别 | `system_basic` | APL 级别 |
| SELinux | `u:r:sharing_service:s0` | 安全上下文 |

### SA 配置

| SA ID | 进程 | 库路径 | 按需启动 |
|-------|------|--------|----------|
| 5527 | sharing_service | libsharing_service.z.so | true |
| 5528 | sharing_service | libsharing_service.z.so | true |

## 日志配置

### 日志宏

| 宏 | 级别 | 用途 |
|----|------|------|
| `SHARING_LOGI` | INFO | 普通信息 |
| `SHARING_LOGD` | DEBUG | 调试信息 |
| `SHARING_LOGW` | WARN | 警告信息 |
| `SHARING_LOGE` | ERROR | 错误信息 |

### 日志格式

```cpp
// 公共属性输出
SHARING_LOGI("message %{public}p.", pointer);
SHARING_LOGI("message %{public}s.", string);

// 私有属性输出 (日志级别 >= DEBUG)
SHARING_LOGD("message %{private}p.", pointer);
```

## 事件配置 (hisysevent)

### Domain

| Domain | 说明 |
|--------|------|
| `SHARING` | 投屏框架事件 |

### 事件定义

| 事件名 | 类型 | 级别 | 说明 |
|--------|------|------|------|
| `SHARING_INIT_FAIL` | FAULT | CRITICAL | 初始化失败 |
| `SHARING_OPT_FAIL` | FAULT | CRITICAL | 操作失败 |
| `SHARING_INIT` | BEHAVIOR | MINOR | 初始化事件 |
| `SHARING_FORWARD_EVENT` | BEHAVIOR | MINOR | 转发事件 |

## 依赖配置

### 内部依赖

| 组件 | 用途 |
|------|------|
| `interfaces/innerkits/native/wfd` | Inner API |
| `services/interaction/` | 进程交互 |
| `services/context/` | 业务容器 |
| `services/agent/` | 业务代理 |
| `services/mediachannel/` | 媒体通道 |
| `services/codec/` | 编解码 |
| `services/protocol/` | 协议栈 |
| `services/network/` | 网络 |

### 外部系统依赖

| 系统组件 | 用途 |
|----------|------|
| `ability_base` | Ability 基础 |
| `ability_runtime` | Ability 运行时 |
| `bundle_framework` | 包管理 |
| `ipc` | IPC 通信 |
| `safwk` | SA 框架 |
| `hilog` | 日志 |
| `graphic_surface` | 图形 Surface |
| `audio_framework` | 音频框架 |
| `access_token` | 访问控制 |
| `samgr` | 服务管理 |

### 第三方依赖

| 依赖 | 用途 |
|------|------|
| `cJSON` | JSON 解析 |
| `jsoncpp` | JSON 解析 |
| `openssl` | 加密 |
| `ffmpeg` | 媒体处理 |
