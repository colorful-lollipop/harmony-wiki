# 目录结构与模块职责

## 目的

描述 hdc 项目的目录组织结构、各模块的职责边界和文件分类，帮助开发者理解代码组织方式。

## 适用范围

本文档适用于：
- 理解代码目录结构
- 了解模块职责划分
- 定位代码文件

## 相关跳转

- [项目概览](./00_Overview.md) - 项目定位和核心能力
- [架构说明](./03_Architecture.md) - 组件关系和数据流
- [内部 API](./05_Internal_API.md) - 模块间接口定义

---

## 目录树结构

```
/Volumes/lexar/code/d/work/oh/developtools/hdc/
├── BUILD.gn                      # 主构建文件
├── hdc.gni                       # GN 全局配置
├── bundle.json                    # 组件元数据
├── README.md                      # 英文文档
├── README_zh.md                  # 中文文档
├── LICENSE                        # Apache 2.0 许可证
│
├── src/                           # C++ 源代码（不含测试）
│   ├── common/                  # [67 文件] 公用代码（Host/Daemon 共享）
│   ├── daemon/                  # [32 文件] 设备端守护进程代码
│   ├── host/                    # [33 文件] PC 端 client/server 代码
│   └── register/                # [5 文件] JDWP 注册模块
│
├── hdc_rust/                      # Rust 实现
│   ├── src/
│   │   ├── cffi/               # C Foreign Function Interface 桥接层
│   │   ├── common/              # Rust 公用模块
│   │   ├── daemon/              # Rust daemon 实现
│   │   ├── daemon_lib/          # Rust daemon 库
│   │   ├── host/                # Rust host 实现
│   │   ├── host_transfer/       # Rust host 传输模块
│   │   ├── serializer/          # 数据序列化
│   │   ├── tar/                 # TAR 归档
│   │   └── transfer/            # 传输协议
│   └── BUILD.gn
│
├── credential/                    # 凭证管理（条件编译）
│   ├── main.cpp
│   ├── credential_base.cpp/h
│   ├── hdc_subscriber.cpp/h
│   └── secret_manage.cpp/h
│
├── hdcd_user_permit/             # 用户权限助手（条件编译）
│   ├── BUILD.gn
│   └── src/
│       ├── main.cpp
│       ├── common.h
│       └── connection.cpp/h
│
├── sudo/                         # sudo 工具（条件编译）
│   ├── BUILD.gn
│   └── src/
│       ├── main.cpp
│       └── sudo_iam.cpp/h
│
├── scripts/                      # 构建脚本和工具
│   ├── build_standalone_linux_host.sh
│   ├── file_path.cfg
│   ├── hdc_hash_gen.py
│   └── sha256.py
│
└── wiki/                         # 工程文档
    ├── README.md
    ├── SUMMARY.md
    ├── _work/
    │   ├── NOTES.md
    │   └── PLAN.md
    └── ...
```

---

## 模块职责

### 1. src/common/ - 公用模块

**职责**：Host 端和 Device 端共享的核心基础设施

**关键模块**：

| 模块 | 文件 | 职责 |
|--------|------|------|
| 基础工具 | `base.cpp/h` | 字符串处理、编码/解码、文件 I/O、随机数生成 |
| 认证 | `auth.cpp/h` | RSA 密钥生成、签名验证、host 密钥管理 |
| Session 管理 | `session.cpp/h` | Session 生命周期、握手、命令调度 |
| Channel 管理 | `channel.cpp/h` | Channel 创建/销毁、数据传输 |
| 任务管理 | `task.cpp/h` | 任务基类、任务生命周期 |
| 文件传输 | `file.cpp/h` | 文件传输任务实现 |
| 端口转发 | `forward.cpp/h` | 端口转发（TCP/JDWP/Ark/Abstract Socket） |
| 压缩 | `compress.cpp/h` | TAR 归档创建 |
| 解压 | `decompress.cpp/h` | TAR 归档提取 |
| TCP 通信 | `tcp.cpp/h` | TCP Socket 通信基类 |
| USB 通信 | `usb.cpp/h` | USB Bulk 传输实现 |
| UART 通信 | `uart.cpp/h` | 串口通信支持 |
| SSL/TLS | `hdc_ssl.cpp/h` | TLS 1.3 PSK 加密 |
| HUKS 集成 | `hdc_huks.cpp/h` | 华为通用密钥库集成 |
| TLV 编码 | `tlv.cpp/h` | Type-Length-Value 数据编码 |
| Header 处理 | `header.cpp/h` | TAR 格式头处理 |
| Entry 处理 | `entry.cpp/h` | TAR 条目处理 |
| 内存池 | `memory_pool.cpp/h` | 自定义内存池优化 |
| 心跳 | `heartbeat.cpp/h` | 连接保活机制 |
| 文件描述符 | `file_descriptor.cpp/h` | 异步文件描述符 I/O |
| 连接校验 | `connect_validation.cpp/h` | 连接安全验证 |
| 凭证消息 | `credential_message.cpp/h` | 凭证消息协议 |
| 命令事件上报 | `command_event_report.cpp/h` | 命令审计和统计 |
| 统计上报 | `hdc_statistic_reporter.cpp/h` | 使用统计 |
| 服务器日志 | `server_cmd_log.cpp/h` | 服务器命令日志 |
| UV 状态 | `uv_status.cpp/h` | libuv 循环监控 |

**稳定接口**（公共 API）：
- `HdcSessionBase` 类 - Session 管理
- `HdcChannelBase` 类 - Channel 管理
- `HdcTaskBase` 类 - 任务基类
- `HdcTransferBase` 类 - 文件传输基类
- `Base::*` 命名空间 - 工具函数

### 2. src/daemon/ - 设备端守护进程

**职责**：运行在 OpenHarmony 设备上的守护进程，处理来自 client 的请求

**关键模块**：

| 模块 | 文件 | 职责 |
|--------|------|------|
| Daemon 核心 | `daemon.cpp/h` | 主守护进程逻辑、认证处理、命令分发 |
| 应用管理 | `daemon_app.cpp/h` | 应用安装/卸载/启动、bundle 验证 |
| Bridge 功能 | `daemon_bridge.cpp/h` | 桥接功能实现 |
| 端口转发 | `daemon_forward.cpp/h` | 设备端端口转发 |
| SSL 处理 | `daemon_ssl.cpp/h` | Daemon 端 SSL 处理 |
| TCP 传输 | `daemon_tcp.cpp/h` | Daemon 端 TCP 传输 |
| UART 传输 | `daemon_uart.cpp/h` | Daemon 端 UART 传输 |
| Unity 集成 | `daemon_unity.cpp/h` | Unity 框架集成 |
| USB 传输 | `daemon_usb.cpp/h` | Daemon 端 USB 传输 |
| Shell 执行 | `shell.cpp/h` | Shell 命令执行 |
| JDWP 集成 | `jdwp.cpp/h` | Java 调试协议集成 |
| 系统依赖 | `system_depend.cpp/h` | 系统相关操作（挂载、权限等） |
| 配置文件 | `etc/BUILD.gn` | 配置文件安装 |

**配置文件**（`src/daemon/etc/`）：
- `hdc_credential.cfg` - 凭证配置
- `hdcd.cfg` / `hdcd.root.cfg` - Daemon 配置
- `hdc.para` / `hdc.root.para` - 系统参数
- `hdc.para.dac` - DAC 权限参数

**关键类**：
- `HdcDaemon` : public `HdcSessionBase` - Daemon 主类
- `HdcDaemonAuthInfo` - 认证信息结构
- `UserPermit` 枚举 - 用户权限类型

### 3. src/host/ - PC 端 Client/Server

**职责**：运行在开发机 PC 上的 client 和 server 进程

**关键模块**：

| 模块 | 文件 | 职责 |
|--------|------|------|
| Client | `client.cpp/h` | 命令行客户端实现 |
| Server | `server.cpp/h` | 后台服务器实现 |
| Server for Client | `server_for_client.cpp/h` | Server 到 Client 的接口 |
| 扩展 Client | `ext_client.cpp/h` | 扩展客户端功能 |
| 应用管理 | `host_app.cpp/h` | Host 端应用管理 |
| 端口转发 | `host_forward.cpp/h` | Host 端端口转发 |
| Shell 选项 | `host_shell_option.cpp/h` | Shell 命令选项解析 |
| SSL 处理 | `host_ssl.cpp/h` | Host 端 SSL 处理 |
| TCP 传输 | `host_tcp.cpp/h` | Host 端 TCP 传输 |
| UART 传输 | `host_uart.cpp/h` | Host 端 UART 传输 |
| Unity 集成 | `host_unity.cpp/h` | Host 端 Unity 集成 |
| USB 传输 | `host_usb.cpp/h` | Host 端 USB 传输 |
| 升级功能 | `host_updater.cpp/h` | 固件升级支持 |
| 命令翻译 | `translate.cpp/h` | 命令翻译/本地化 |
| 系统依赖 | `system_depend.cpp/h` | Host 系统相关操作 |

**关键类**：
- `HdcServer` : public `HdcSessionBase` - Server 主类
- `HdcClient` - Client 主类

### 4. src/register/ - JDWP 注册模块

**职责**：提供 JDWP（Java Debug Wire Protocol）注册库

| 文件 | 职责 |
|------|------|
| `define_register.h` | 注册常量和类型定义 |
| `hdc_connect.cpp/h` | JDWP 连接管理实现 |
| `hdc_connect.h` | JDWP 连接管理接口（extern "C"） |
| `hdc_jdwp.cpp/h` | JDWP 模拟器实现 |
| `hdc_jdwp.h` | JDWP 模拟器定义 |

**导出接口**（C 接口，非 N-API）：
- `StartConnect(processName, pkgName, isDebug, callback)` - 启动 JDWP 连接
- `StopConnect()` - 停止 JDWP 连接

**生成产物**：`libhdc_register.so` (ohos_shared_library)

### 5. hdc_rust/ - Rust 实现

**职责**：现代化的 Rust 实现（迁移中）

**主要目录**：

| 目录 | 职责 |
|------|------|
| `src/cffi/` | C Foreign Function Interface 桥接层 |
| `src/common/` | Rust 公用模块 |
| `src/daemon/` | Rust daemon 实现 |
| `src/daemon_lib/` | Rust daemon 库 |
| `src/host/` | Rust host 实现 |
| `src/host_transfer/` | Rust host 传输模块 |
| `src/serializer/` | 数据序列化（protobuf-like） |
| `src/tar/` | TAR 归档实现 |
| `src/transfer/` | 传输协议实现 |

**关键模块**：
- `serialize_structs` (ohos_static_library) - C++ 桥接库
- `lib` (ohos_rust_static_library) - Rust 静态库
- `hdcd` (ohos_rust_executable) - Rust daemon 可执行文件
- `hdc_rust` (ohos_rust_executable) - Rust host 可执行文件

### 6. credential/ - 凭证管理

**职责**：设备端凭证管理进程

| 文件 | 职责 |
|------|------|
| `main.cpp` | 凭证服务入口 |
| `credential_base.cpp/h` | 基础凭证操作 |
| `hdc_subscriber.cpp/h` | HDC 事件订阅 |
| `secret_manage.cpp/h` | 密钥/秘密管理 |

**条件编译**：`hdc_feature_support_credential`

### 7. hdcd_user_permit/ - 用户权限助手

**职责**：用户授权对话框支持

| 文件 | 职责 |
|------|------|
| `src/main.cpp` | 权限助手入口 |
| `src/common.h` | 公共定义 |
| `src/connection.cpp/h` | ExtensionAbility 连接 |

**条件编译**：`support_hdcd_user_permit`

### 8. sudo/ - Sudo 工具

**职责**：权限提升工具

| 文件 | 职责 |
|------|------|
| `src/main.cpp` | sudo 命令入口 |
| `src/sudo_iam.cpp/h` | 身份认证管理 |

**条件编译**：`hdc_feature_support_sudo`

---

## 文件分类原则

### 排除测试文件

本文档及分析**不包含**测试相关内容：
- ❌ `test/` 目录
- ❌ `*_test.*` 文件
- ❌ `*_fuzzer.*` 文件
- ❌ `unittest/` 目录

### 头文件 vs 实现文件

- **头文件（*.h）**：接口定义、结构体、常量
- **实现文件（*.cpp）**：具体实现逻辑
- **头文件位置**：
  - `src/common/` - 公用接口定义
  - `src/daemon/` - Daemon 特定接口
  - `src/host/` - Host 特定接口

---

## 关键结论

1. **三部分架构清晰**：common/（共享）、daemon/（设备）、host/（PC）
2. **67 个公用模块文件**：涵盖通信、认证、文件传输、压缩等核心功能
3. **Rust 迁移进行中**：`hdc_rust/` 包含完整的 Rust 实现
4. **条件编译模块**：credential/、hdcd_user_permit/、sudo/ 根据 feature flags 编译
5. **配置文件分离**：daemon/etc/ 包含系统配置和参数文件

---

## 待确认事项

**TODO(需确认)**：
1. Rust 实现的迁移进度（哪些模块已完成迁移）
2. 各模块的单元测试覆盖情况
3. 代码复杂度分析（如需要）
