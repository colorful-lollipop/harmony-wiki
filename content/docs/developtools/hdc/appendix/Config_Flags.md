# 配置标志

## 目的

列出 hdc 项目中的关键编译宏、Feature Flags 和平台相关配置。

## 适用范围

本文档适用于：
- 理解编译配置选项
- 定制构建配置
- 平台特性控制
- 功能开关管理

## 相关跳转

- [GN 目标梳理](./06_GN_Targets.md) - 构建系统详细说明

---

## 编译配置宏

### 平台相关宏

**定义位置**：`BUILD.gn`

| 宏名 | 使用位置 | 说明 | 值 |
|--------|----------|------|------|
| `HDC_HOST` | Host 端构建 | PC 端 | 1 |
| `HDC_DAEMON` | Daemon 端构建 | 设备端 | 1 |
| `HARMONY_PROJECT` | 通用 | OpenHarmony 项目 | 1 |
| `USE_CONFIG_UV_THREADS` | 通用 | 使用配置的线程池 | 1 |

**条件编译宏**：

| 宏名 | 触发条件 | 说明 |
|--------|----------|------|
| `HOST_MAC` | `is_mac` | macOS 平台 |
| `HOST_LINUX` | `is_linux` | Linux 平台 |
| `HOST_MINGW` | `is_mingw` | Windows/MinGW 平台 |
| `WIN32_LEAN_AND_MEAN` | `is_mingw` | Windows API 精简 |
| `HOST_OHOS` | `is_ohos` | OpenHarmony 构建 |
| `IS_RELEASE_VERSION` | `is_ohos && build_variant == "user"` | 用户版本构建 |

**证据**：`BUILD.gn:385-449`

### 功能特性宏

| 宏名 | 默认值 | 说明 | 证据 |
|--------|---------|------|--------|
| `HDC_DEBUG` | false | 调试模式 | `hdc.gni:5` |
| `HDC_HILOG` | true | 启用 hilog 日志 | `BUILD.gn:106` |
| `OPENSSL_SUPPRESS_DEPRECATED` | true | 抑制 OpenSSL 废弃警告 | `BUILD.gn:107` |
| `HDC_SUPPORT_ENCRYPT_TCP` | true | 支持 TCP 加密 | `BUILD.gn:391` |
| `MEMORY_POOL_ENABLE` | true | 启用内存池 | `BUILD.gn:110` |
| `JS_JDWP_CONNECT` | true | 启用 JDWP 连接 | `BUILD.gn:543` |
| `HDC_VERSION_CHECK` | false | 启用版本检查 | `BUILD.gn:459` |
| `SURPPORT_SELINUX` | 条件 | SELinux 支持（需要 `build_selinux`）| `BUILD.gn:157` |
| `UPDATER_MODE` | 条件 | Updater 模式（image_name == "updater"）| `BUILD.gn:408` |
| `HDC_TRACE` | 条件 | 跟踪支持（需要 system 镜像）| `BUILD.gn:164` |
| `HDC_STATISTIC_REPORT_ENABLE` | 条件 | 统计上报（需要 system 镜像）| `BUILD.gn:165` |
| `HDC_HICOLLIE_ENABLE` | 条件 | hicollie 支持（需要 system 镜像）| `BUILD.gn:166` |
| `HDC_SUPPORT_ENCRYPT_PRIVATE_KEY` | 条件 | 私钥加密（is_ohos）| `BUILD.gn:441` |
| `HDC_SUPPORT_REPORT_COMMAND_EVENT` | 条件 | 命令事件上报（is_ohos）| `BUILD.gn:443` |
| `HDC_EMULATOR` | 条件 | 模拟器模式（is_emulator）| `hdc.gni` |

---

## hdc.gni 配置参数

### 全局构建参数

**文件**：`/Volumes/lexar/code/d/work/oh/developtools/hdc/hdc.gni`

| 参数 | 默认值 | 说明 |
|--------|---------|------|
| `hdcd_uv_thread_size` | 4 | Daemon UV 线程池大小 |
| `hdc_uv_thread_size` | 128 | Host UV 线程池大小 |
| `hdc_ospm_auth_disable` | false | 禁用 OSPM 认证 |
| `hdc_debug` | false | 启用调试模式 |
| `hdc_host_hide_debug_win` | true | Windows 下隐藏调试窗口 |
| `hdc_support_uart` | true | 支持 UART 传输 |
| `hdc_test_coverage` | false | 测试覆盖率 |
| `hdc_jdwp_test` | false | JDWP 测试 |
| `js_jdwp_connect` | true | JS JDWP 连接 |
| `hdc_version_check` | false | 版本检查 |
| `support_hdcd_user_permit` | false | 支持用户权限助手 |
| `hdc_support_account_constraint` | true | 支持账户约束 |

---

## Bundle.json Feature Flags

### 组件特性

**文件**：`/Volumes/lexar/code/d/work/oh/developtools/hdc/bundle.json:15-20`

| Feature | 默认值 | 说明 | 证据 |
|--------|---------|------|--------|
| `hdc_feature_support_sudo` | false | sudo 功能支持 | bundle.json:16 |
| `hdc_feature_support_credential` | false | 凭证管理支持 | bundle.json:17 |
| `hdc_feature_support_report_command_event` | false | 命令事件上报支持 | bundle.json:18 |
| `hdc_feature_support_usr_symlink` | false | 用户符号链接支持 | bundle.json:19 |

**启用方式**：
- 这些 features 通过 `bundle.json` 定义
- 构建系统根据这些 flags 决定是否编译相应模块

**证据**：`BUILD.gn:59-64` - 这些 flags 用于条件编译

---

## 系统参数配置

### HDC 系统参数（Device 端）

| 参数 | 路径 | 说明 | 相关代码 |
|--------|------|------|----------|
| `persist.hdc.root` | /system/param/hdc.root.para | Root 模式开关 | src/daemon/main.cpp:274 |
| `persist.hdc.mode` | /system/param/hdc.mode | 连接模式（usb/tcp）| TODO(需确认) |
| `const.debuggable` | /system/param/const.debuggable | 开发者模式开关 | src/register/hdc_connect.cpp:145 |
| `const.hdc.secure` | /system/param/const.hdc.secure | 安全模式开关 | src/daemon/daemon.cpp:137 |
| `const.boot.oemmode` | /system/param/const.boot.oemmode | OEM 锁机模式 | src/daemon/daemon.cpp:138 |
| `persist.hdc.daemon.auth_result` | /system/param/hdc.daemon.auth_result | 认证结果 | src/daemon/daemon.cpp:104 |
| `persist.hdc.daemon.auth_msg` | /system/param/hdc.daemon.auth_msg | 认证消息 | src/daemon/daemon.cpp:103 |

**参数读取**：
- `SystemDepend::GetDevItem()` - 使用 OpenHarmony 参数框架
- 证据：`src/daemon/daemon.cpp:120-166`

---

## 常量定义

### 协议常量

**文件**：`src/common/define.h`

| 常量名 | 值 | 说明 |
|--------|--------|------|
| `VER_PROTOCOL` | 0x01 | 协议版本 |
| `HANDSHAKE_MESSAGE` | "OHOS HDC" | 握手消息 |
| `PACKET_FLAG` | "HW" | 包标记（2 字节）|
| `UDS_PATH` | "/data/hdc/hdc_debug/hdc_server" | Unix Domain Socket 路径 |
| `DEFAULT_BACKLOG` | 10 | 默认监听队列大小 |
| `HDC_SOCKETPAIR_SIZE` | 128 | Socketpair 缓冲区大小 |
| `MAX_CONNECTKEY_SIZE` | 128 | 最大连接密钥大小 |
| `BUF_SIZE_MICRO` | 8 | 微缓冲区大小 |
| `BUF_SIZE_TINY` | 32 | 小缓冲区大小 |
| `BUF_SIZE_MIDDLE` | 4096 | 中等缓冲区大小 |

### 枚举值

**文件**：`src/common/define_enum.h`

| 枚举 | 范围 | 说明 |
|--------|--------|------|
| `ConnType` | 0-4 | 连接类型（USB/TCP/SERIAL/BT/UNKNOWN）|
| `ConnStatus` | 0-3 | 连接状态（UNKNOW/READY/CONNECTED/OFFLINE/UNAUTH）|
| `AuthVerifyType` | 0-2 | 认证类型（RSA_ENCRYPT/RSA_3072_SHA512/PSK_TLS）|
| `InnerCtrlCommand` | 0-19 | 内部控制命令 |
| `HdcCommand` | 0-5000 | 完整命令范围 |
| `TaskType` | 0-5 | 任务类型（UNITY/SHELL/FILE/FORWARD/APP/FLASHD）|
| `UserPermit` | 0-2 | 用户权限（REFUSE/ALLOWONCE/ALLOWFORVER）|

---

## 配置建议

### 推荐配置（开发环境）

| 场景 | 建议配置 | 原因 |
|--------|----------|------|
| 调试开发 | 启用 `HDC_DEBUG`, `hdc_debug` | 详细日志输出 |
| 性能测试 | 增加 `hdc_uv_thread_size` | 更多线程处理并发 |
| 安全测试 | 启用 `HDC_VERSION_CHECK` | 防止版本不匹配 |
| 生产构建 | 禁用调试，启用 `IS_RELEASE_VERSION` | 最小化日志和调试信息 |

### 推荐配置（生产环境）

| 场景 | 建议配置 | 原因 |
|--------|----------|------|
| 安全构建 | 启用 `HDC_SUPPORT_ENCRYPT_TCP` | TLS 加密保护 |
| 安全构建 | 设置 `const.hdc.secure=1` | 强制认证 |
| 安全构建 | 设置 `const.boot.oemmode=rd` | OEM 锁机模式 |
| 稳定性 | 设置 `hdc_ospm_auth_disable=false` | 启用 OSPM 认证 |
| 审计 | 启用 `HDC_SUPPORT_REPORT_COMMAND_EVENT` | 记录所有命令 |

---

## 配置依赖关系

### Feature Flags 依赖图

```
hdc_feature_support_sudo
  └─> 编译 sudo 模块（sudo/BUILD.gn）

hdc_feature_support_credential
  └─> 编译 credential 模块（credential/BUILD.gn）
      └─> 依赖 HUKS 集成（src/common/hdc_huks.cpp）

hdc_feature_support_report_command_event
  └─> 编译时添加命令事件上报（src/common/command_event_report.cpp）
      └─> 依赖 common_event_service

support_hdcd_user_permit
  └─> 编译 hdcd_user_permit 模块（hdcd_user_permit/BUILD.gn）
      └─> 依赖 ability_runtime, window_manager
```

---

## 关键结论

1. **三层配置**：
   - 编译时配置（hdc.gni + bundle.json features）
   - 运行时配置（系统参数）
   - 运行时环境（环境变量、文件权限）

2. **Feature Flags 策略**：
   - 主要功能通过 bundle.json 的 features 控制
   - 平台相关 features 通过 is_* 条件变量控制
   - 镜像相关 features 通过 image_name 控制

3. **安全相关配置**：
   - 认证相关：`const.hdc.secure`, `const.boot.oemmode`, `persist.hdc.daemon.*`
   - 加密支持：`HDC_SUPPORT_ENCRYPT_TCP`
   - 版本检查：`HDC_VERSION_CHECK`

4. **调试支持**：
   - 宏控制：`HDC_DEBUG`（调试日志）
   - 运行时控制：`hdc -l <0-5>`（日志级别）

---

## 待确认事项

**TODO(需确认)**：
1. 所有系统参数的完整列表和默认值
2. 各 Feature Flag 的具体行为差异
3. 配置参数的实时生效机制
4. 跨平台配置的最佳实践
