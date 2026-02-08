# 项目定位与边界

## 目的

描述 hdc 项目的定位、核心能力、运行环境和关键概念，帮助新开发者理解项目的整体范围和边界。

## 适用范围

本文档适用于：
- hdc 项目的架构理解
- 功能特性概览
- 运行环境要求
- 项目边界和限制

## 相关跳转

- [目录结构](./02_Directory_Structure.md) - 详细的模块组织
- [架构说明](./03_Architecture.md) - Client/Server/Daemon 三部分架构
- [对外 API](./04_External_API.md) - JDWP 注册接口
- [内部 API](./05_Internal_API.md) - 模块间接口
- [安全评审](./08_Security_Review.md) - 安全机制和风险

---

## 项目定位

### 核心定位

hdc（OpenHarmony Device Connector）是 OpenHarmony 生态系统的设备连接和调试工具，为开发者提供命令行界面，用于：
- 设备连接管理
- 文件传输
- 端口转发
- Shell 命令执行
- 应用安装/卸载
- JDWP（Java Debug Wire Protocol）调试支持

**证据**：`README_zh.md:14-16`

### 解决的核心问题

1. **跨平台设备连接**
   - 支持 Windows/Linux/Mac 开发机
   - 支持 USB/TCP/UART 多种传输方式
   - 支持模拟器连接

2. **统一调试接口**
   - 提供 `hdc` 命令行工具
   - 集成到 IDE（DevEco Studio 等）
   - 支持远程 Shell 和日志查看

3. **高效数据传输**
   - 支持大文件传输
   - 支持断点续传
   - 支持压缩（LZ4）

## 项目边界

### 包含范围

✅ **包含**：
- hdc 客户端（PC 工具）
- hdc 服务端（PC 后台进程）
- hdcd 守护进程（设备端）
- JDWP 注册库（libhdc_register.so）
- 凭证管理工具（hdc_credential）
- sudo 工具（sudo）
- 用户权限助手（hdcd_user_permit）
- 公共代码库（src/common/）

### 不包含范围

❌ **不包含**：
- OpenHarmony Ability 框架
- SystemAbility 系统
- 应用层调试功能（hilog 抓取等）
- 测试框架和测试用例

---

## 核心能力

### 1. 设备连接管理

**功能**：
- USB 设备发现和连接
- TCP 网络连接
- UART 串口连接
- 多设备管理
- 连接状态监控

**证据**：`src/common/define_enum.h:25`

```cpp
enum ConnType { CONN_USB = 0, CONN_TCP, CONN_SERIAL, CONN_BT, CONN_UNKNOWN };
```

### 2. 文件传输

**功能**：
- PC ↔ 设备双向文件传输
- 支持目录和单个文件
- 保持文件权限和属性
- TAR 归档格式

**命令范围**：`src/common/define_enum.h:3000-3099`

### 3. 端口转发

**功能**：
- TCP 端口转发（forward/reverse）
- JDWP 端口映射
- 抽象套接字转发
- 文件系统访问转发

**命令范围**：`src/common/define_enum.h:2500-2599`

### 4. Shell 命令执行

**功能**：
- 远程 Shell 执行
- 交互式 Shell 模式
- Shell 数据流重定向
- TTY 支持

**命令范围**：`src/common/define_enum.h:2000-2099`

### 5. 应用管理

**功能**：
- HAP 应用安装（支持增量安装）
- 应用卸载
- 应用启动
- Bundle 名称验证

**命令范围**：`src/common/define_enum.h:3500-3599`

### 6. JDWP 调试支持

**功能**：
- Java 调试协议集成
- JDWP 端口映射
- App 注册到 JDWP 服务

**证据**：`src/register/hdc_connect.cpp`

### 7. 认证与加密

**功能**：
- RSA 公钥认证
- RSA 签名验证
- PSK（Pre-Shared Key）TLS 1.3 加密
- HUKS（华为通用密钥库）集成

**证据**：`src/common/auth.cpp`, `src/daemon/daemon.cpp`

---

## 运行环境

### PC 端（开发机）

**支持平台**：
- **Linux**: Ubuntu 18.04+ 64 位（推荐）
- **Windows**: Windows 10+ 64 位
- **macOS**: macOS 10.15+ 64 位

**依赖库**：
- libusb（USB 通信）
- libuv（异步 I/O）
- OpenSSL（SSL/TLS 加密）
- lz4（压缩）
- bounds_checking_function（安全函数）

**证据**：`BUILD.gn:380-429`

### 设备端（OpenHarmony）

**最小系统版本**：OpenHarmony 3.0+

**组件依赖**（`bundle.json:26-53`）：
- c_utils
- hilog
- ipc
- ability_runtime
- hitrace
- selinux
- openssl
- huks
- libusb
- ylong_runtime

---

## 关键概念

### Client/Server/Daemon 三部分

**架构**：

```
┌─────────────┐     UDS/TCP     ┌──────────────┐
│  HDC Client │◄──────────────►│  HDC Server  │◄──────────────►│ HDC Daemon  │
│  (命令行)  │                  │  (后台进程)   │                  │  (设备)     │
└─────────────┘                  └──────────────┘                  └─────────────┘
```

**说明**：
- **Client**: 用户在命令行执行的 `hdc` 命令
- **Server**: PC 上运行的后台进程，管理 client-daemon 通信
- **Daemon**: 设备上运行的守护进程，处理 client 请求

**证据**：`README_zh.md:18-27`

### Session 与 Channel

**Session**：一次完整的 PC ↔ Device 连接会话
- 全局唯一的 sessionId
- 包含多个 Channel（逻辑通道）

**Channel**：Session 内的逻辑通信管道
- 用于特定功能（Shell、文件传输、端口转发等）
- 拥有独立的 channelId

**证据**：`src/common/define_plus.h:208-358`

### 认证流程

**认证类型**：`src/common/session.h:27`

1. **AUTH_NONE** - 初始握手
2. **AUTH_TOKEN** - Token 认证
3. **AUTH_SIGNATURE** - RSA 签名认证
4. **AUTH_PUBLICKEY** - 公钥认证
5. **AUTH_OK** - 认证成功
6. **AUTH_FAIL** - 认证失败
7. **AUTH_SSL_TLS_PSK** - TLS PSK 加密

---

## 限制与约束

1. **单实例约束**
   - 同一时间，一个设备只能有一个 hdcd 实例
   - Server 管理所有 client 连接

2. **权限约束**
   - 默认需要用户授权（通过 UI 对话框）
   - Root 操作需要 sudo 工具
   - 系统分区操作需要先挂载为读写

3. **网络约束**
   - USB 模式下不支持多设备并发
   - TCP 模式需要手动配置（persist.hdc.mode=tcp）

4. **版本约束**
   - Client 和 Daemon 版本必须一致
   - 版本不匹配时拒绝连接

---

## 关键结论

1. **hdc 是纯 C++ 工具**，不提供 N-API 绑定
2. **使用自定义 Socket 协议**，不依赖 OpenHarmony SystemAbility
3. **三部分架构清晰**：Client（PC）、Server（PC）、Daemon（设备）
4. **支持多种传输方式**：USB、TCP、UART
5. **内置 RSA 认证和 TLS 1.3 加密**，保障连接安全
6. **完整的应用生命周期管理**：安装、启动、调试、卸载

---

## 待确认事项

**TODO(需确认)**：
1. 各平台的具体性能指标（传输速率、延迟等）
2. 企业部署环境的特殊配置
3. 跨版本兼容性策略详情
